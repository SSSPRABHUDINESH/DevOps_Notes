# Command Line Arguments & Environment Variables
---
## Python Command Line Arguments: Comprehensive Guide

Command line arguments allow you to pass dynamic inputs to a Python script at execution time directly from the terminal. By eliminating hard-coded values, your scripts become modular, reusable, and aligned with standard DevOps practices (such as the Google Cloud CLI `gcloud` or AWS CLI `aws`).

---

### Why Use Command Line Arguments?

* **Eliminates Hard-coding:** Avoid editing source code files whenever inputs change.
* **Enhances Script Reusability:** A single script can execute different logic paths across environments (e.g., dev vs. prod).
* **Aligns with CLI Standards:** Follows industrial automation patterns where infrastructure parameters are passed dynamically.

---

### Understanding the `sys` Module & `sys.argv`

Python’s built-in `sys` module provides access to variables maintained by the interpreter. Command line arguments are stored in a list named `sys.argv`.

- `sys` module will be installed as part of python installation.

#### `sys.argv` Index Breakdown

When executing a command like `python calculator.py 10 add 5`:

```text
               ┌── sys.argv[0] -> "calculator.py" (Script Name)
               │   ┌── sys.argv[1] -> "10" (First Argument)
               │   │    ┌── sys.argv[2] -> "add" (Second Argument)
               │   │    │    ┌── sys.argv[3] -> "5" (Third Argument)
               ▼   ▼    ▼    ▼
python calculator.py  10  add    5

```

| Index | Content | Type | Description |
| --- | --- | --- | --- |
| **`sys.argv[0]`** | Script Name | `str` | Name of the executed Python file. |
| **`sys.argv[1]`** | First Argument | `str` | First user-supplied input. |
| **`sys.argv[2]`** | Second Argument | `str` | Second user-supplied input. |
| **`sys.argv[n]`** | $n^{\text{th}}$ Argument | `str` | Additional inputs. |

---

### Key Technical Considerations

1. **Default Data Type is Always String:** All elements in `sys.argv` are stored as `str`. For numerical calculations, you must explicitly cast them using `float()` or `int()`.
2. **Index Errors (`IndexError`):** If a user runs the script without supplying expected arguments, accessing `sys.argv[1]` raises an `IndexError`. Always check list bounds before accessing elements.
3. **Control Flow:** Combine positional arguments with `if/elif/else` conditional logic to trigger target functions based on user flags.

---

### Complete Worked Example: Dynamic Calculator

Below is a complete implementation of `calculator.py` featuring error handling, type conversion, and dynamic operation selection.

```python
import sys

def add(num1, num2):
    return num1 + num2

def sub(num1, num2):
    return num1 - num2

def mul(num1, num2):
    return num1 * num2

def div(num1, num2):
    if num2 == 0:
        return "Error: Division by zero is not allowed."
    return num1 / num2

def main():
    # 1. Validate argument count to prevent IndexError
    if len(sys.argv) != 4:
        print("Usage: python calculator.py <number1> <operation> <number2>")
        print("Supported operations: add, sub, mul, div")
        sys.exit(1)

    # 2. Extract inputs from sys.argv
    raw_num1 = sys.argv[1]
    operation = sys.argv[2].lower()
    raw_num2 = sys.argv[3]

    # 3. Type Conversion (str -> float)
    try:
        num1 = float(raw_num1)
        num2 = float(raw_num2)
    except ValueError:
        print("Error: First and third arguments must be valid numbers.")
        sys.exit(1)

    # 4. Conditional Logic based on operation argument
    if operation == "add":
        result = add(num1, num2)
    elif operation == "sub":
        result = sub(num1, num2)
    elif operation == "mul":
        result = mul(num1, num2)
    elif operation == "div":
        result = div(num1, num2)
    else:
        print(f"Error: Unknown operation '{operation}'. Use add, sub, mul, or div.")
        sys.exit(1)

    print(f"Result: {result}")

if __name__ == "__main__":
    main()

```

---

### Terminal Execution & Sample Output

**Addition Example:**

```bash
$ python calculator.py 2 add 3
Result: 5.0

```

**Multiplication Example:**

```bash
$ python calculator.py 10.5 mul 2
Result: 21.0

```

**Handling Missing Arguments:**

```bash
$ python calculator.py 2 add
Usage: python calculator.py <number1> <operation> <number2>
Supported operations: add, sub, mul, div

```

---

# Python Environment Variables: Comprehensive Guide

Environment variables (**Env Vars** or **EnVs**) are global key-value pairs managed by your operating system outside your application code. They allow you to pass sensitive data and deployment configurations into a Python script without hardcoding them into source files.

---

### Why Use Environment Variables?

* **Security & Secret Protection:** Passwords, API keys, database credentials, and SSL certificates are kept out of version control systems (like Git). This prevents accidental exposure in public repositories, logs, or build artifacts.
* **Environment Portability:** Code runs dynamically across **Development**, **Staging**, and **Production** without source code modification—you simply change the system's environment variables per server.
* **Automation & CI/CD Readiness:** Standardized pattern for injecting secrets into containerized applications (Docker), Kubernetes pods, and automated runners (GitHub Actions, Cloud Build).

---

### Terminal Management: Setting vs. Viewing

```text
┌────────────────────────────────────────────────────────┐
│               Terminal / Shell (Host OS)              │
│  $ export DB_PASSWORD="secret_password_123"            │
└───────────────────────────┬────────────────────────────┘
                            │ Pass-through
                            ▼
┌────────────────────────────────────────────────────────┐
│               Python Application Memory                │
│  db_pass = os.getenv("DB_PASSWORD")                     │
└────────────────────────────────────────────────────────┘

```

| OS Platform | Set Variable Command | View All Variables Command |
| --- | --- | --- |
| **Linux / macOS (Bash/Zsh)** | `export DB_PASS="my_secret"` | `env` or `printenv` |
| **Windows (Command Prompt)** | `set DB_PASS="my_secret"` | `set` |
| **Windows (PowerShell)** | `$env:DB_PASS="my_secret"` | `Get-ChildItem Env:` |

*Note: Variables exported directly in a terminal session exist only for the lifespan of that active shell instance.*

---

### Working with `os` Module in Python

Python reads environment variables through the built-in `os` module. There are two primary ways to access them:

#### Method 1: `os.getenv("KEY", default_value)` (Safer / Optional Keys)

Returns `None` or a specified fallback default if the key does not exist instead of raising an exception.

```python
import os

# Returns value if present, otherwise returns None
db_host = os.getenv("DB_HOST") 

# Returns explicit fallback if key is missing
db_port = os.getenv("DB_PORT", "5432") 

```

#### Method 2: `os.environ["KEY"]` (Strict / Required Keys)

Accesses environment variables like a standard dictionary. Raises a `KeyError` if the requested variable is not set on the system. Use this for mandatory secrets where the app must crash immediately if configuration is missing.

```python
import os

# Crashes loudly with KeyError if DB_PASSWORD is missing
db_password = os.environ["DB_PASSWORD"] 

```

---

### Command Line Arguments vs. Environment Variables

| Feature | Command Line Arguments (`sys.argv`) | Environment Variables (`os.getenv`) |
| --- | --- | --- |
| **Primary Purpose** | Operational flags & dynamic execution params | Secrets, API keys, infrastructure config |
| **Security Level** | **Low:** Visible in shell history & process trees | **High:** Injected in-memory, kept out of history |
| **Frequency of Change** | Changes per execution run (e.g., `calc 2 add 3`) | Set once per deployment environment |
| **Lifespan** | Exists only for the specific execution command | Persists across process executions in the shell |

---

### Complete Worked Example: Secure Server Connection Script

This script securely connects to a mock remote service by reading sensitive database credentials from environment variables while allowing fallbacks for non-sensitive operational settings.

```python
import os
import sys

def connect_to_database():
    # 1. Fetch REQUIRED sensitive credentials (raises KeyError if absent)
    try:
        db_user = os.environ["DB_USER"]
        db_password = os.environ["DB_PASSWORD"]
    except KeyError as missing_key:
        print(f"CRITICAL ERROR: Mandatory environment variable {missing_key} is missing!")
        print("Please set credentials using: export DB_USER='...' and export DB_PASSWORD='...'")
        sys.exit(1)

    # 2. Fetch OPTIONAL configuration settings with sensible defaults
    db_host = os.getenv("DB_HOST", "127.0.0.1")
    raw_port = os.getenv("DB_PORT", "5432")
    
    # 3. Environment variables are always string data type -> Explicit Casting
    try:
        db_port = int(raw_port)
    except ValueError:
        print(f"ERROR: DB_PORT must be an integer. Got: '{raw_port}'")
        sys.exit(1)

    # 4. Connection Simulation
    print("--- Database Connection Initialization ---")
    print(f"Connecting to Host : {db_host}:{db_port}")
    print(f"Authenticated User : {db_user}")
    print(f"Password Status    : {'*' * len(db_password)} (Loaded safely)")
    print("Connection established successfully!")

if __name__ == "__main__":
    connect_to_database()

```

---

### Terminal Execution & Sample Output

**Failure Case (Missing Required Credentials):**

```bash
$ python app.py
CRITICAL ERROR: Mandatory environment variable 'DB_USER' is missing!
Please set credentials using: export DB_USER='...' and export DB_PASSWORD='...'

```

**Successful Execution (Setting Variables & Running):**

```bash
$ export DB_USER="admin_dev"
$ export DB_PASSWORD="SuperSecretPass123!"
$ export DB_HOST="db.internal.company.com"

$ python app.py
--- Database Connection Initialization ---
Connecting to Host : db.internal.company.com:5432
Authenticated User : admin_dev
Password Status    : *───────────────────* (Loaded safely)
Connection established successfully!

```
