# Conditional Statements in Python (if, elif, else, switch case)

Conditional statements are a fundamental part of programming that allow you to make decisions and execute different blocks of code based on certain conditions. In Python, you can use `if`, `elif` (short for "else if"), and `else` to create conditional statements.

## `if` Statement

The `if` statement is used to execute a block of code if a specified condition is `True`. If the condition is `False`, the code block is skipped.

```python
if condition:
    # Code to execute if the condition is True
```

- Example:

```python
x = 10
if x > 5:
    print("x is greater than 5")
```

### OR

```python
x = 10
# Parenthesis is completely valid
if(x > 5):
    print("x is greater than 5")
```

## `elif` Statement

The `elif` statement allows you to check additional conditions if the previous `if` or `elif` conditions are `False`. You can have multiple `elif` statements after the initial `if` statement.

```python
if condition1:
    # Code to execute if condition1 is True
elif condition2:
    # Code to execute if condition2 is True
elif condition3:
    # Code to execute if condition3 is True
# ...
else:
    # Code to execute if none of the conditions are True
```

- Example:

```python
x = 10
if x > 15:
    print("x is greater than 15")
elif x > 5:
    print("x is greater than 5 but not greater than 15")
else:
    print("x is not greater than 5")
```

## `else` Statement

The `else` statement is used to specify a block of code to execute when none of the previous conditions (in the `if` and `elif` statements) are `True`.

```python
if condition:
    # Code to execute if the condition is True
else:
    # Code to execute if the condition is False
```

- Example:

```python
x = 3
if x > 5:
    print("x is greater than 5")
else:
    print("x is not greater than 5")
```

---

# Switch case:

In Python 3.10 and newer, switch-case functionality is implemented using the **`match-case`** statement.

### Basic `match-case` Syntax

```python
command = "start"

match command:
    case "start":
        print("Starting the program...")
    case "stop":
        print("Stopping the program...")
    case "pause":
        print("Program paused.")
    case _:
        print("Unknown command")  # Wildcard '_' acts as the default case

```

---

### Key Features

* **Default Case (`_`):** The underscore `_` serves as the wildcard/default option, matching anything that hasn't been caught by previous cases.
* **Combine Multiple Conditions (`|`):** Use the pipe operator `|` to match several patterns in a single case block.
```python
day = "Saturday"

match day:
    case "Saturday" | "Sunday":
        print("It's the weekend!")
    case _:
        print("It's a weekday.")

```


* **Guard Clauses (`if`):** Add extra conditions directly inside a case check.
```python
number = 15

match number:
    case x if x > 0 and x % 2 == 0:
        print(f"{x} is a positive even number")
    case x if x > 0 and x % 2 != 0:
        print(f"{x} is a positive odd number")
    case _:
        print("Number is non-positive")

```



---

### Alternative for Python 3.9 and Older

If you are using an older version of Python that does not support `match-case`, you can achieve the same behavior using a **dictionary of functions or values**:

```python
def get_status(code):
    statuses = {
        200: "OK",
        404: "Not Found",
        500: "Server Error"
    }
    # .get() provides a default value if the key isn't found
    return statuses.get(code, "Unknown Status Code")

print(get_status(200))  # Output: OK
print(get_status(403))  # Output: Unknown Status Code

```
