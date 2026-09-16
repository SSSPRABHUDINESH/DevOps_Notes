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
