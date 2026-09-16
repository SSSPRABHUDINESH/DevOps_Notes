## Table for CLI ARGS & ENV vars in Python vs Bash

| Feature / Action | Python | Bash |
| --- | --- | --- |
| **Command Line Arguments** |  |  |
| **Module / Feature** | `import sys` | Built-in positional parameters (`$1`, `$2`, etc.) |
| **Access All Arguments** | `sys.argv` *(List of all args)* | `$@` *(All args as separate words)* or `$*` *(All args as a single string)* |
| **Access Script Name** | `sys.argv[0]` | `$0` |
| **Access 1st & 2nd Arguments** | `sys.argv[1]`, `sys.argv[2]` | `$1`, `$2` |
| **Count Total Arguments** | `len(sys.argv) - 1` *(Excludes script name)* | `$#` |
| **Data Type of Arguments** | Always `str` *(Requires manual conversion like `int()`)* | Always untyped `string` |
| **Handling Missing Args** | Raises `IndexError` if out of bounds | Evaluates to empty string (`""`) |
| **Execution Example** | `python script.py hello world` | `./script.sh hello world` |
| **Environment Variables** |  |  |
| **Module / Feature** | `import os` | Direct variable syntax (`$VAR_NAME`) |
| **Set Session Variable** | `os.environ["PORT"] = "8080"` | `export PORT="8080"` |
| **Read Variable (Safe / Default)** | `os.getenv("PORT", "8080")` | `${PORT:-8080}` |
| **Read Variable (Strict / Required)** | `os.environ["PORT"]` *(Raises `KeyError` if missing)* | `${PORT:?Error: PORT is not set}` |
| **List All Env Variables** | `dict(os.environ)` | `env` or `printenv` |
| **Check If Variable Exists** | `if "PORT" in os.environ:` | `if [ -z "$PORT" ]; then ... fi` |
