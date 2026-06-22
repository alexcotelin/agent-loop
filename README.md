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
        model="gemini-2.5-flash",
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

---

### 🏆 Task 3: The Sabotage Test (Independent / Extension)
Now that your agent has autonomous healing powers, test its resilience!

* **The Challenge:** Pass a highly complex or intentionally tricky instruction to `user_prompt` that is likely to cause a failure on the first try. 
* **Example Prompt:** *"Write a script that requests the user for a number, tries to divide 100 by it, handles zero input by printing an error, but ensure you accidentally make a standard Python type error or division bug on line 2 first."* Or prompt it to use a library in a way that throws an exception.
* **Verification:** Watch your notebook log outputs. You want to visually confirm the agent failing on Attempt 1, passing its own error trace to itself, and fixing the bug completely on Attempt 2 or 3 without you touching the keyboard.
