# Data Types

In programming, a data type is a classification or categorization that specifies which type of value a variable can hold. Data types are essential because they determine how data is stored in memory and what operations can be performed on that data. Python, like many programming languages, supports several built-in data types.

🔥🔥🔥
> Python is **Dynamically typed** programming language.

Here are some of the common data types in Python:

1. **Numeric Data Types:**
   - **int**: Represents integers (whole numbers). Example: `x = 5`
   - **float**: Represents floating-point numbers (numbers with decimal points). Example: `y = 3.14`
   - **complex**: Represents complex numbers. Example: `z = 2 + 3j`

2. **Sequence Types:**
   - **str**: Represents strings (sequences of characters). Example: `text = "Hello, World"`
   - **list**: Represents lists (ordered, mutable sequences). Example: `my_list = [1, 2, 3]`
   - **tuple**: Represents tuples (ordered, immutable sequences). Example: `my_tuple = (1, 2, 3)`

3. **Mapping Type:**
   - **dict**: Represents dictionaries (key-value pairs). Example: `my_dict = {'name': 'John', 'age': 30}`

4. **Set Types:**
   - **set**: Represents sets (unordered collections of unique elements). Example: `my_set = {1, 2, 3}`
   - **frozenset**: Represents immutable sets. Example: `my_frozenset = frozenset([1, 2, 3])`

5. **Boolean Type:**
   - **bool**: Represents Boolean values (`True` or `False`). Example: `is_valid = True`

6. **Binary Types:**
   - **bytes**: Represents immutable sequences of bytes. Example: `data = b'Hello'`
   - **bytearray**: Represents mutable sequences of bytes. Example: `data = bytearray(b'Hello')`

7. **None Type:**
   - **NoneType**: Represents the `None` object, which is used to indicate the absence of a value or a null value.

8. **Custom Data Types:**
   - You can also define your custom data types using classes and objects.

--- 

# Most commonly used built-in Python functions in DevOps. 

- They are grouped logically by how you would use them when writing `automation scripts`, `parsing logs`, or interacting with `files` and `environments`.

## 🐧 System & Environment Management
These functions help your script talk to the operating system, handle environment variables, and manage execution.

| Function | What it does in DevOps | Quick Example |
|---|---|---|
| id() | Returns a unique identifier for an object; occasionally used to verify memory objects during state debugging. | id(config) |
| getattr() | Fetches an attribute from an object dynamically. Essential for dynamic configuration loading. | getattr(os, 'environ') |
| callable() | Checks if an object can be called (like a function). Useful for plug-in verification in automation setups. | callable(my_func) |

## 📁 File, Path & Data Input/Output
Essential functions for reading server configs (YAML/JSON), writing logs, and reading input from Jenkins pipelines or users.

| Function | What it does in DevOps | Quick Example |
|---|---|---|
| open() | Opens files to read logs, update manifests, or parse configurations like .env files. | open('app.log', 'r') |
| print() | Outputs text to the console. Used heavily for CI/CD pipeline logging. | print(f"Deploying to {env}") |
| input() | Pauses execution to accept manual user input (e.g., asking for confirmation before a production destroy). | input("Proceed? (y/n)") |

## 🛠️ Data Manipulation & Conversions
DevOps scripts handle a lot of text, numbers, and lists. These functions transform that data into usable formats.

| Function | What it does in DevOps | Quick Example |
|---|---|---|
| str() | Converts objects (like integers or exceptions) into text strings for logging or shell commands. | str(error_code) |
| int() | Converts text strings (like an HTTP port or response code) into numbers for comparison. | int("8080") |
| len() | Counts items in a list (e.g., number of active servers) or characters in a string. | len(active_pods) |
| type() | Checks the data type of an unknown variable. Great for debugging API responses. | type(api_response) |
| dict() | Creates a dictionary. Crucial for organizing structured data like JSON or metadata tags. | dict(env="prod", id=4) |
| list() | Converts data streams or generators into a standard list for easier processing. | list(active_ips) |
| set() | Removes duplicates automatically. Perfect for finding unique error messages or IP addresses in a log. | set(ip_list) |
| bool() | Evaluates if a variable contains data or is empty/null, helping manage conditional logic. | bool(api_token) |

## 🔍 Filtering, Sorting & Loops
Use these to filter lists of servers, sort deployment order, or loop through metrics efficiently.

| Function | What it does in DevOps | Quick Example |
|---|---|---|
| enumerate() | Loops through a list while keeping track of the loop count (e.g., indexing step numbers in a build). | for i, pod in enumerate(pods): |
| zip() | Combines two parallel lists together (e.g., matching a list of server names with a list of their IP addresses). | zip(hostnames, ips) |
| filter() | Extracts items from a collection that match a condition (e.g., filtering out only the unhealthy nodes). | filter(is_unhealthy, nodes) |
| map() | Applies a function to an entire list (e.g., stripping whitespace from a list of strings extracted from a log). | map(str.strip, lines) |
| sorted() | Sorts items alphabetically or numerically (e.g., arranging deployment targets by priority). | sorted(server_list) |
| range() | Generates a sequence of numbers. Commonly used for loops, like setting up a retry mechanism (e.g., try 3 times). | for attempt in range(3): |
| all() | Returns True if all items in a list meet a condition (e.g., checking if all servers are healthy). | all(status_list) |
| any() | Returns True if at least one item in a list meets a condition (e.g., checking if any service is down). | any(failures) |

## 🧯 Error Handling & Inspection
These functions make troubleshooting automation issues significantly faster.

| Function | What it does in DevOps | Quick Example |
|---|---|---|
| dir() | Lists all available attributes and methods of an object. Essential for debugging third-party cloud SDKs (like boto3). | dir(aws_client) |
| help() | Displays documentation directly in your terminal for any module, function, or library. | help(os.path) |
| repr() | Returns a printable representation of an object, showing hidden characters (like \n or \t) in logs. | repr(raw_data) |




