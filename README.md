# 🚀 AI Agent Workshop: Build a Self-Correcting Code Loop

In this workshop, you will build an autonomous AI coding loop. Instead of just asking an AI for code, you will write a Python control script that instructs Gemini to write code, executes it dynamically inside your notebook, catches any runtime errors, and automatically feeds those errors back to the AI until it fixes itself.

---

## 🛠️ Step 0: Pre-Workshop Setup (Do THIS Before the Workshop!)
Please complete these three steps **before arriving**:

1. **Access Google Colab:** Ensure you can sign in to [Google Colab](https://colab.research.google.com/) using a Google account.
2. **Get an API Key:** Go to [Google AI Studio](https://aistudio.google.com/) and create a free API Key.
3. **Save Your Key:** Copy the key and save it securely in a temporary `.txt` file on your laptop so you can copy-paste it instantly during the session.

---

## ⏱️ Workshop Tasks

### 🚀 Task 1: The Code Generator (Done Together)
We will open a blank Google Colab notebook, install the new `google-genai` SDK, and write a script that sends a prompt to Gemini demanding raw Python code. We will then use Python's built-in `exec()` function to execute that AI-written code string live and capture its terminal outputs.

#### Task 1 Boilerplate Code:
```python
!pip install -q -U google-genai

import io
import sys
from google import genai
from google.genai import types

# =====================================================================
# 🛠️ SETUP: Paste your Gemini API key below!
# =====================================================================
GEMINI_API_KEY = "YOUR_API_KEY_HERE"

# Initialize the Gemini Client using the official v1 SDK standard
client = genai.Client(api_key=GEMINI_API_KEY)

def ask_ai_for_code(prompt: str) -> str:
    """Sends a prompt to Gemini and demands strict, raw Python code back."""
    system_instruction = (
        "You are an expert Python code generator. "
        "Output ONLY valid executable Python code. Do NOT wrap your answer in "
        "markdown code blocks (no ```python). Do not write explanations."
    )
    
    response = client.models.generate_content(
        model="gemini-flash-lite-latest",
        contents=prompt,
        config=types.GenerateContentConfig(
            system_instruction=system_instruction,
            temperature=0.1,
        ),
    )
    
    # Strip away accidental markdown block formatting if the model leaks it
    code = response.text.strip()
    if code.startswith("```python"):
        code = code.split("```python")[1].split("```")[0].strip()
    elif code.startswith("```"):
        code = code.split("```")[1].split("```")[0].strip()
        
    return code

def execute_generated_code(code_string: str) -> str:
    """Safely runs the code string and captures whatever it prints out."""
    old_stdout = sys.stdout
    redirected_output = io.StringIO()
    sys.stdout = redirected_output
    
    try:
        # This executes the raw string as actual running Python code
        exec(code_string, {})
        output = redirected_output.getvalue()
    except Exception as e:
        # Returns the error traceback if the AI code crashes
        output = f"EXECUTION_ERROR: {str(e)}"
    finally:
        sys.stdout = old_stdout
        
    return output

# =====================================================================
# 🏃‍♂️ Run Task 1
# =====================================================================
user_prompt = "Write a script that calculates the 10th Fibonacci number and prints 'Result: ' followed by the answer."
print(f"🤖 Step 1: Prompting AI -> '{user_prompt}'\n")

ai_code = ask_ai_for_code(user_prompt)
print("---- GENERATED CODE ----")
print(ai_code)
print("------------------------\n")

print("⚙️ Step 2: Running code inside Python...")
execution_result = execute_generated_code(ai_code)

print("---- CONSOLE OUTPUT ----")
print(execution_result)
print("------------------------")
```

---

### 🔄 Task 2: The Self-Correction Loop (Independent)
Right now, if the AI makes a logic mistake or an execution typo, the script catches the error but stops there. Your task is to turn this into an **agentic loop**.

* **The Goal:** Wrap the evaluation steps in a `while` loop (allow up to 3 or 4 attempts).
* **The Logic:** If `execute_generated_code` returns an output starting with `"EXECUTION_ERROR:"`, do not give up! Instead, programmatically append that exact error string to a new prompt (e.g., *"Your previous code failed with error X. Rewrite the code completely and fix this error."*) and pass it back to `ask_ai_for_code()`.
* **Exit Condition:** The loop should terminate immediately if the code executes successfully without an error string being thrown.

```python
# =====================================================================
# 🔄 TASK 2: THE SELF-CORRECTION AGENT LOOP
# =====================================================================

# 1. Give the AI a prompt that has room for syntax error or confusion
initial_prompt = (
    "Write a script that takes a string '123a', tries to convert it to an integer, "
    "and multiplies it by 2. Intentionally write broken code on the first line to "
    "trigger a ValueError, but make sure the final logic works if fixed."
)

current_prompt = initial_prompt
max_attempts = 3
attempt = 1
success = False

print(f"🚀 Starting Agent Loop for problem:\n'{initial_prompt}'\n")

while attempt <= max_attempts and not success:
    print(f"🔄 [ATTEMPT {attempt} of {max_attempts}]")
    print("🤖 Prompting Gemini for code...")
    
    # Get code from AI based on the current prompt
    ai_code = ask_ai_for_code(current_prompt)
    
    print("⚙️ Running code string...")
    result = execute_generated_code(ai_code)
    
    # Check if the output contains our execution error flag
    if "EXECUTION_ERROR:" in result:
        print(f"❌ Failed! Python threw an error:")
        print(f"   {result}\n")
        
        # 🧠 THE AGENT TWIST: Feed the failure back into the prompt history
        current_prompt = (
            f"Your previous code failed. Here is the exact Python error trace:\n"
            f"{result}\n\n"
            f"Please review your code, fix the error, and rewrite the script perfectly."
        )
        attempt += 1
    else:
        print("✅ Success! The code executed flawlessly.")
        print("---- CONSOLE OUTPUT ----")
        print(result.strip())
        print("------------------------")
        success = True

if not success:
    print("🛑 Agent failed to resolve the issue within the attempt limit.")
```
---

### 🏆 Task 3: The Sabotage Test (Independent / Extension)
Now that your agent has autonomous healing powers, test its resilience!

* **The Challenge:** Pass a highly complex or intentionally tricky instruction to `user_prompt` that is likely to cause a failure on the first try. 
* **Example Prompt:** *"Write a script that requests the user for a number, tries to divide 100 by it, handles zero input by printing an error, but ensure you accidentally make a standard Python type error or division bug on line 2 first."* Or prompt it to use a library in a way that throws an exception.
* **Verification:** Watch your notebook log outputs. You want to visually confirm the agent failing on Attempt 1, passing its own error trace to itself, and fixing the bug completely on Attempt 2 or 3 without you touching the keyboard.

```python
  # =====================================================================
# 🏆 TASK 3: THE SABOTAGE TEST (REUSE YOUR LOOP FROM TASK 2)
# =====================================================================

# Change your `initial_prompt` to one of these sabotage prompts to watch the agent heal itself:

# 💣 SABOTAGE OPTION A: The Missing Library/Deprecation Test
initial_prompt = (
    "Write a script that uses the 'requests' library to get the status code of 'https://httpbin.org/status/200'. "
    "However, you MUST intentionally misspell the import on line 1 as 'import reqwests' to force an error on the first run, "
    "and let the self-correction loop fix it."
)

# 💣 SABOTAGE OPTION B: The Type Trap
# initial_prompt = (
#     "Write a script that calculates 10 divided by 2. "
#     "But on line 2, intentionally add a string to an integer (like: x = '5' + 5) to trigger a TypeError, "
#     "and rely on your error loop to fix it."
# )

# 💣 SABOTAGE OPTION C:
initial_prompt = (
    "Write a Python script using 'requests' and 'BeautifulSoup' to fetch the live Wikipedia page "
    "for 'Python (programming language)' (https://en.wikipedia.org/wiki/Python_(programming_language)) "
    "and print out the text inside the main infobox side-panel. "
    "SABOTAGE INSTRUCTION: You must guess the HTML class name of the infobox on your first attempt "
    "without looking. If your script prints an empty result or throws an AttributeError because the "
    "HTML tag doesn't exist, raise a runtime error with a message explaining what tag failed so your "
    "self-correction loop can read the error, inspect a snippet of the page text, and choose a better HTML selector."
)

# 💣 SABOTAGE OPTION D:
initial_prompt = (
    "Write a Python script that reads a local file named 'user_data.csv', calculates the average "
    "age of the users listed in it, and prints the result. "
    "SABOTAGE INSTRUCTION: Do NOT write any code that creates the 'user_data.csv' file on your first attempt. "
    "Let the script crash with a FileNotFoundError. Your self-correction loop must intercept that specific "
    "error, realize the file is missing, generate mock CSV data on the fly, write code to create the file first, "
    "and then calculate the average successfully."
)
```

