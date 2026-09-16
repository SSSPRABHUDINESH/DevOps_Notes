# Keywords in Python:

Keywords are reserved words in Python that have predefined meanings and cannot be used as variable names or identifiers. These words are used to define the structure and logic of the program. They are an integral part of the Python language and are case-sensitive, which means you must use them exactly as specified.

Here are some important Python keywords:

# Python Keywords Reference

| Keyword | Type | Description | Example |
|---------|------|-------------|---------|
| **and** | Logical Operator | Returns True if both operands are true | `if x > 5 and y < 10:` |
| **or** | Logical Operator | Returns True if at least one operand is true | `if x > 5 or y < 10:` |
| **not** | Logical Operator | Returns the opposite of the operand's truth value | `if not x:` |
| **if** | Conditional | Starts a conditional statement | `if condition:` |
| **else** | Conditional | Alternative code block when if is False | `else:` |
| **elif** | Conditional | Checks additional conditions (else if) | `elif condition:` |
| **while** | Loop | Repeatedly executes code while condition is true | `while x < 10:` |
| **for** | Loop | Iterates over a sequence | `for item in list:` |
| **in** | Loop/Operator | Checks if value exists in sequence | `if x in list:` |
| **try** | Exception Handling | Begins a block subject to exception handling | `try:` |
| **except** | Exception Handling | Catches and handles exceptions | `except ValueError:` |
| **finally** | Exception Handling | Always executes, exception or not | `finally:` |
| **def** | Function | Defines a function | `def my_function():` |
| **return** | Function | Specifies the return value of a function | `return result` |
| **class** | OOP | Defines a class (blueprint for objects) | `class MyClass:` |
| **import** | Module | Imports modules or libraries | `import numpy` |
| **from** | Module | Imports specific components from a module | `from os import path` |
| **as** | Module | Creates an alias for a module | `import numpy as np` |
| **True** | Boolean | Represents boolean value "true" | `x = True` |
| **False** | Boolean | Represents boolean value "false" | `x = False` |
| **None** | Special | Represents null or absence of value | `x = None` |
| **is** | Comparison | Identity comparison (same object in memory) | `if x is None:` |
| **lambda** | Function | Creates small anonymous functions | `lambda x: x * 2` |
| **with** | Context Manager | Ensures operations before and after code block | `with open(file) as f:` |
| **global** | Scope | Declares a global variable in function scope | `global x` |
| **nonlocal** | Scope | Declares a nonlocal variable in enclosing scope | `nonlocal x` |

23. **lambda**: It is used to create small, anonymous functions (lambda functions).

24. **with**: It is used for context management, ensuring that certain operations are performed before and after a block of code.

25. **global**: It is used to declare a global variable within a function's scope.

26. **nonlocal**: It is used to declare a variable as nonlocal, which allows modifying a variable in an enclosing (but non-global) scope.
