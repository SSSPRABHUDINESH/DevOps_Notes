# 📖 CHAPTER: Bash Scripting (Part 1 of 3)

### `bash_script.md` — Modules 1 through 10 (Language Core & Control Flow)

---

## 🐚 1. Shell Fundamentals

A shell script is an executable text file containing a sequence of commands executed by an interpreter.

### 🔹 The Shebang (`#!`)

The first line of any script specifies the absolute path to the interpreter binary:

```bash
#!/usr/bin/env bash

```

* 📌 **Why `#!/usr/bin/env bash`?** It finds `bash` in the system's `$PATH`, making the script portable across different Linux distributions and operating systems compared to hardcoding `#!/bin/bash`.

### 🔹 Execution Permissions & Running Scripts

To execute a script directly, give it execute permissions:

```bash
# 1. Grant execute permission
chmod +x deploy.sh

# 2. Execute directly via relative path
./deploy.sh

# 3. Alternative: Run via interpreter explicitly (does not require +x permission)
bash deploy.sh

```

---

## 📦 2. Variables & Scope

Bash variables are untyped strings by default.

### 🔹 Declaration Rules

* ⚠️ **No spaces around the equals sign (`=`):** `NAME="John"` is valid; `NAME = "John"` will result in an error.
* Reference variables with `$VAR` or `${VAR}`.

```bash
#!/usr/bin/env bash

# Variable declaration
SERVICE_NAME="payment-api"
PORT=8080

# Variable referencing
echo "Deploying ${SERVICE_NAME} on port ${PORT}..."

# Read-only (constant) variable
readonly ENVIRONMENT="production"

# Unsetting a variable
TEMP_KEY="secret-123"
unset TEMP_KEY

# Local variables (must be inside a function)
simulate_deployment() {
    # This variable only exists inside this function
    local temp_process_id=9945
    local status="in-progress"
    
    echo "[Inside function] Process ID: ${temp_process_id} | Status: ${status}"
}

# Call the function
simulate_deployment

# Attempting to reference the local variable outside the function returns nothing
echo "[Outside function] Process ID is empty: ${temp_process_id}"

```

### 🔹 Variable Scoping

* **Global by default:** Variables defined anywhere in a script are accessible globally unless explicitly scoped.
* **`local` keyword:** Restricts variable access to the enclosing function.

---

## 📥 3. Input & Output (`echo`, `printf`, `read`)

### 🔹 Output Commands

| Command | Features | Practical Example |
| --- | --- | --- |
| **`echo`** 📢 | Simple output; supports `-e` for escape sequences, `-n` to omit newline | `echo -e "Status:\tOK\nCode:\t200"` |
| **`printf`** 🖨️ | Formatted, type-safe output (similar to C/Python) | `printf "User: %-10s ID: %04d\n" "alice" 42` |

### 🔹 Input with `read`

```bash
#!/usr/bin/env bash

# Prompt user for input (-p)
read -p "Enter environment name: " TARGET_ENV

# Silent input for sensitive data (-s)
read -sp "Enter database password: " DB_PASS
echo ""

# Print the prompt without a trailing newline
echo -n "Enter your service name: "

# Read the input into the variable
read SERVICE_NAME

echo "You entered: $SERVICE_NAME"

# Timeout after 10 seconds (-t)
read -t 10 -p "Confirm deployment (y/n)? " CONFIRM

```

---

## 🚦 4. Exit Codes & Status Verification

Every Linux command returns an exit status between `0` and `255` upon termination.

* ✅ **`0`**: Success / No errors.
* ❌ **`1 - 255`**: Failure / Error codes.
* 🔍 **`$?`**: Captures the exit code of the most recently executed foreground command.

```bash
#!/usr/bin/env bash

# Check if a directory exists
mkdir -p /var/log/my-app

# Inspect exit status
if [ $? -eq 0 ]; then
    echo "Directory created successfully."
else
    echo "Failed to create directory." >&2
    exit 1
fi

```

---

## 🔀 5. Conditional Statements (`if`, `test`, `case`)

### 🔹 Comparison Operators

```
┌──────────────────────────────┬──────────────────────────────┬──────────────────────────────┐
│ 🔢 Numeric Comparisons       │ 🔤 String Comparisons        │ 📁 File Test Operators       │
├──────────────────────────────┼──────────────────────────────┼──────────────────────────────┤
│ `-eq` : Equal to             │ `==` or `=` : Strings match  │ `-f` : Regular file exists   │
│ `-ne` : Not equal to         │ `!=`        : Do not match   │ `-d` : Directory exists      │
│ `-gt` : Greater than         │ `-z`        : String is empty│ `-e` : Path exists           │
│ `-lt` : Less than            │ `-n`        : String not null│ `-r` / `-w` / `-x` : Perms   │
│ `-ge` / `-le` : GTE / LTE    │                              │ `-s` : File is not empty     │
└──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘

```

### 🔹 Single vs. Double Brackets (`[` vs. `[[`)

* `[ ... ]`: Standard POSIX test command.
* `[[ ... ]]`: Modern Bash extension. Supports pattern matching, regular expressions (`=~`), and logical operators (`&&`, `||`) without escaping.

```bash
#!/usr/bin/env bash

FILE_PATH="/etc/hosts"

# Double brackets example with pattern matching
if [[ -f "$FILE_PATH" && -r "$FILE_PATH" ]]; then
    echo "Hosts file is readable."
elif [[ ! -e "$FILE_PATH" ]]; then
    echo "File does not exist."
else
    echo "File exists but is unreadable."
fi

```

### 🔹 Pattern Matching with `case`

```bash
#!/usr/bin/env bash

ACTION="$1"

case "$ACTION" in
    (start|START)
        echo "Starting service..."
        ;;
    (stop)
        echo "Stopping service..."
        ;;
    (restart|reload)
        echo "Restarting service..."
        ;;
    (*)
        echo "Usage: $0 {start|stop|restart}" >&2
        exit 1
        ;;
esac

```

---

## 🔁 6. Loops (`for`, `while`, `until`)

### 🔹 `for` Loops

```bash
#!/usr/bin/env bash

# Iterating over a list
for REGION in "us-central1" "europe-west1" "asia-east1"; do
    echo "Configuring network for: ${REGION}"
done

# Iterating over a numeric range
for i in {1..5}; do
    echo "Attempt ${i}..."
done

# C-style for loop
for (( i=0; i<3; i++ )); do
    echo "Counter: ${i}"
done

```

### 🔹 Reading Files Line-by-Line with `while`

```bash
#!/usr/bin/env bash

CONFIG_FILE="servers.txt"

# Safe line-by-line reading preserving whitespace (IFS=) and backslashes (-r)
while IFS= read -r SERVER_IP || [[ -n "$SERVER_IP" ]]; do
    echo "Connecting to ${SERVER_IP}..."
done < "$CONFIG_FILE"

```

---

## 🧩 7. Functions

Functions encapsulate reusable logic and can accept positional arguments.

```bash
#!/usr/bin/env bash

# Function definition
log_message() {
    local LEVEL="$1"
    local MESSAGE="$2"
    local TIMESTAMP
    TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")

    echo "[${TIMESTAMP}] [${LEVEL}] ${MESSAGE}"
}

# Invoking the function
log_message "INFO" "Application started."
log_message "ERROR" "Database connection timeout."

```

* 📌 **Capturing Function Output:** Use standard output rather than `return` for strings:
```bash
get_cluster_ip() {
    echo "10.0.0.45"
}
IP=$(get_cluster_ip)

```



---

## 🎯 8. Arguments & Positional Parameters

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      Special Parameter Reference                            │
├───────────────────┬─────────────────────────────────────────────────────────┤
│ `$0`              │ Name of the script being executed                       │
│ `$1` ... `$9`     │ Positional arguments 1 through 9                        │
│ `${10}`           │ Positional arguments 10 and above (braces required)     │
│ `$#`              │ Total count of arguments passed to script               │
│ `$@`              │ All arguments as separate quoted strings (`"$1" "$2"`)  │
│ `$*`              │ All arguments joined into a single string (`"$1 $2"`)   │
│ `$$`              │ Process ID (PID) of the current running script          │
└───────────────────┴─────────────────────────────────────────────────────────┘

```

### 🔹 CLI Argument Parsing with `getopts`

```bash
#!/usr/bin/env bash

while getopts "e:p:h" OPT; do
    case "$OPT" in
        e) ENV_NAME="$OPTARG" ;;
        p) PORT_NUM="$OPTARG" ;;
        h) echo "Usage: $0 -e <env> -p <port>"; exit 0 ;;
        *) echo "Invalid option" >&2; exit 1 ;;
    esac
done

echo "Target: ${ENV_NAME:-dev}, Port: ${PORT_NUM:-8080}"

```

---

## 🗃️ 9. Arrays (Indexed & Associative)

### 🔹 Indexed Arrays

```bash
#!/usr/bin/env bash

# Declaration and assignment
SERVICES=("auth" "billing" "orders")

# Accessing elements
echo "First service: ${SERVICES[0]}"

# Accessing all elements
echo "All services: ${SERVICES[@]}"

# Array length
echo "Total count: ${#SERVICES[@]}"

# Append element
SERVICES+=("notifications")

# Slicing (${ARRAY[@]:offset:length})
echo "Subset: ${SERVICES[@]:1:2}"

```

### 🔹 Associative Arrays (Key-Value Maps)

```bash
#!/usr/bin/env bash

# Must be explicitly declared with -A
declare -A HTTP_STATUS

HTTP_STATUS["200"]="OK"
HTTP_STATUS["404"]="Not Found"
HTTP_STATUS["500"]="Internal Server Error"

echo "Code 200 means: ${HTTP_STATUS["200"]}"

```

---

## 🔤 10. String Operations & Parameter Expansion

Bash provides built-in string manipulation that avoids invoking subshells or external processes like `sed` and `awk`.

```bash
#!/usr/bin/env bash

IMAGE_TAG="us-central1-docker.pkg.dev/my-project/apps/order-api:v1.2.0"

# 1. String Length (${#STRING})
echo "Length: ${#IMAGE_TAG}"

# 2. Substring Extraction (${STRING:offset:length})
echo "Prefix: ${IMAGE_TAG:0:11}" # us-central1

# 3. Substring Replacement
# Replace first match: ${STRING/pattern/replacement}
echo "${IMAGE_TAG/order-api/payment-api}"
# Replace all matches: ${STRING//pattern/replacement}

# 4. Prefix Stripping
# Shortest match from start: ${STRING#pattern}
echo "${IMAGE_TAG#*/}" # my-project/apps/order-api:v1.2.0
# Longest match from start: ${STRING##pattern}
echo "${IMAGE_TAG##*:}" # v1.2.0 (extracts just the tag)

# 5. Suffix Stripping
# Shortest match from end: ${STRING%pattern}
echo "${IMAGE_TAG%:*}" # us-central1-docker.pkg.dev/my-project/apps/order-api
# Longest match from end: ${STRING%%pattern}

# 6. Case Conversion
TEXT="Production"
echo "${TEXT^^}" # PRODUCTION (Uppercase)
echo "${TEXT,,}" # production (Lowercase)

```

---

# 📖 CHAPTER: Bash Scripting (Part 2 of 3)

### `bash_script.md` — Modules 11 through 21 (Text Processing, Process Control & Strict Mode)

---

## ⚙️ 11–12. Command Substitution & I/O Pipelines

* 🔄 **Command Substitution:** Run a subshell and capture its standard output into a variable using `$(command)` (modern standard) instead of legacy backticks ``command``.
```bash
CURRENT_DATE=$(date +%Y-%m-%d)

```


* ⛓️ **Pipelines (`|`):** Connect the `stdout` of one command directly to the `stdin` of the next:
```bash
ps aux | grep "python" | wc -l

```



---

## 🛠️ 13–15. Text Wrangling & File Operations

* 🔍 **`grep`:** Filter lines matching patterns (`grep -E "500|502" app.log`).
* ✂️ **`sed`:** Stream editor for text substitution (`sed -i 's/dev/prod/g' config.env`).
* 📊 **`awk`:** Column-based pattern processing (`awk -F':' '{print $1, $3}' /etc/passwd`).
* 📦 **`find` & `xargs`:** Locate files and pass them as batch arguments:
```bash
find /var/log -name "*.log" -mtime +30 | xargs rm -f

```



---

## 📡 16–17. Process Management & Signal Traps

* ⏱️ **Job Control:** Use `&` to run background jobs, `$!` for the last background PID, and `wait $!` to pause execution until completion.
* 🪤 **Signals & `trap`:** Intercept termination signals (`SIGINT`, `SIGTERM`, `EXIT`) to execute cleanup routines:
```bash
cleanup() {
    rm -f /tmp/lock.pid
}
trap cleanup EXIT

```



---

## 🛡️ 18–20. Defensive Scripting & Debugging

The **Unofficial Bash Strict Mode** prevents silent failures:

```bash
set -euo pipefail

```

* 🛑 **`-e` (errexit):** Exit immediately if any command returns a non-zero status.
* 🔍 **`-u` (nounset):** Treat unset variables as errors and exit immediately.
* ⛓️ **`-o pipefail`:** A pipeline fails if *any* command in the chain fails, not just the last one.
* 🐞 **`-x` (xtrace):** Print each command and its arguments as it executes (great for debugging).

---

## 🌐 21. Environment Variables

* 📤 **`export`:** Pass variables to child processes:
```bash
export ENVIRONMENT="production"

```



---

# 📖 CHAPTER: Bash Scripting (Part 3 of 3)

### `bash_script.md` — Modules 22 through 29 (DevOps Integrations & Production Scenarios)

---

## ⏰ 22. Cron & Scheduled Scripts

Cron runs scheduled background jobs at fixed intervals via the daemon `cron`.

### 🔹 Crontab Syntax

```text
┌───────────── Minute (0 - 59)
│ ┌───────────── Hour (0 - 23)
│ │ ┌───────────── Day of Month (1 - 31)
│ │ │ ┌───────────── Month (1 - 12)
│ │ │ │ ┌───────────── Day of Week (0 - 6, 0=Sunday)
│ │ │ │ │
* * * * * /path/to/script.sh

```

### 🔹 Managing Crontab

```bash
# Edit current user's crontab
crontab -e

# List current crontab entries
crontab -l

```

### 🔹 Production Cron Best Practices

* 🌍 **Minimal Environment:** Cron runs with a bare-bones environment (often only `PATH=/usr/bin:/bin`). Always define full paths inside your script or set `$PATH` explicitly at the top of the crontab.
* 📜 **Explicit Logging:** Redirect both `stdout` and `stderr` to capture errors:
```text
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

```



---

## 🔑 23. SSH Automation

Automate remote server configuration without interactive prompts.

### 🔹 Key-Based Execution & Heredocs

```bash
#!/usr/bin/env bash
set -euo pipefail

TARGET_HOST="app-server-01"
TARGET_USER="deploy"

# Execute a single command remotely
ssh -o BatchMode=yes -o ConnectTimeout=5 "${TARGET_USER}@${TARGET_HOST}" "uptime"

# Execute a multi-line block via heredoc
ssh "${TARGET_USER}@${TARGET_HOST}" 'bash -s' << 'EOF'
    echo "Running updates on remote host..."
    sudo apt-get update -y
    sudo systemctl restart nginx
EOF

```

* 🛡️ **Flag Highlights:**
* `-o BatchMode=yes`: Disables password prompts; fails immediately if key-based authentication fails (crucial for automation).
* `-o ConnectTimeout=5`: Prevents the script from hanging indefinitely if the host is down.



---

## 🐙 24. Bash + Git Automation

Automate version control checks and release pipelines.

```bash
#!/usr/bin/env bash
set -euo pipefail

# Check for uncommitted changes
if ! git diff-index --quiet HEAD --; then
    echo "⚠️ Warning: Working directory has uncommitted changes." >&2
    exit 1
fi

# Fetch current branch name and commit hash
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
COMMIT_HASH=$(git rev-parse --short HEAD)

echo "🚀 Deploying commit ${COMMIT_HASH} from branch ${CURRENT_BRANCH}..."

```

---

## 🐳 25. Bash + Docker Automation

### 🔹 Container Entrypoint Scripts & Signal Handling

When writing an `entrypoint.sh` for a Docker container, use `exec` to replace the shell process with your application binary. This ensures signals like `SIGTERM` from Docker or Kubernetes reach your app directly.

```bash
#!/usr/bin/env bash
set -e

# Run database migrations before app launch
echo "Running pre-flight checks..."
python manage.py migrate

# Replace shell process with the main application (PID 1)
exec gunicorn --bind 0.0.0.0:8080 app:wsgi

```

---

## ☸️ 26. Bash + Kubernetes (`kubectl`)

Automate cluster status verification and wait conditions.

```bash
#!/usr/bin/env bash
set -euo pipefail

NAMESPACE="production"
DEPLOYMENT_NAME="payment-api"

# Wait for a rollout to complete with a timeout
echo "Checking rollout status for ${DEPLOYMENT_NAME}..."
kubectl rollout status deployment/"${DEPLOYMENT_NAME}" \
    --namespace="${NAMESPACE}" \
    --timeout=120s

# Extract Pod names dynamically using jsonpath
READY_PODS=$(kubectl get pods -n "${NAMESPACE}" \
    -l app="${DEPLOYMENT_NAME}" \
    -o jsonpath='{.items[*].metadata.name}')

echo "Active pods: ${READY_PODS}"

```

---

## ☁️ 27. Bash + Google Cloud CLI (`gcloud`)

Combine `gcloud` with formatted outputs and `jq` for cloud automation.

```bash
#!/usr/bin/env bash
set -euo pipefail

PROJECT_ID="my-prod-project"
SERVICE_NAME="orders-service"
REGION="us-central1"

# Fetch Cloud Run service URL using GCP built-in formatting
SERVICE_URL=$(gcloud run services describe "${SERVICE_NAME}" \
    --project="${PROJECT_ID}" \
    --region="${REGION}" \
    --format='value(status.url)')

echo "Service accessible at: ${SERVICE_URL}"

# Health check using curl
HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" "${SERVICE_URL}/healthz")

if [[ "$HTTP_STATUS" -ne 200 ]]; then
    echo "❌ Health check failed with status: ${HTTP_STATUS}" >&2
    exit 1
fi

echo "✅ Service is healthy!"

```

---

## 🚀 28. Bash in CI/CD Pipelines

Bash scripts in CI/CD platforms (like Cloud Build or GitHub Actions) must fail fast, handle secrets safely, and emit structured outputs.

```bash
#!/usr/bin/env bash
# CI/CD test and build runner
set -euo pipefail

echo "=========================================="
echo "Starting CI Pipeline on Build: ${BUILD_ID:-local}"
echo "=========================================="

# 1. Linting
echo "Step 1: Running Linter..."
flake8 src/

# 2. Testing
echo "Step 2: Running Unit Tests..."
pytest tests/ --junitxml=reports/test-results.xml

# 3. Artifact Packaging
echo "Step 3: Packaging release..."
tar -czf "release-${BUILD_ID:-latest}.tar.gz" -C dist/ .

echo "✅ Build completed successfully."

```

---

## 🛡️ 29. Production Enterprise Script Template

A battle-tested template bringing together logging, argument parsing, lock files, traps, and strict execution.

```bash
#!/usr/bin/env bash
# ==============================================================================
# Script Name : prod_backup_sync.sh
# Description : Automated archive and cloud sync with lock file & error traps
# ==============================================================================
set -euo pipefail
IFS=$'\n\t'

# Configuration & Variables
readonly SCRIPT_NAME="$(basename "$0")"
readonly LOCK_FILE="/tmp/${SCRIPT_NAME}.lock"
readonly LOG_FILE="/var/log/${SCRIPT_NAME%.*}.log"

# Logging helper
log() {
    local level="$1"
    shift
    local msg="$*"
    local timestamp
    timestamp="$(date '+%Y-%m-%d %H:%M:%S')"
    echo "[${timestamp}] [${level}] ${msg}" | tee -a "${LOG_FILE}"
}

# Cleanup and error traps
cleanup() {
    local exit_code=$?
    if [[ -f "${LOCK_FILE}" ]]; then
        rm -f "${LOCK_FILE}"
    fi
    if [[ $exit_code -ne 0 ]]; then
        log "ERROR" "Script exited unexpectedly with code: ${exit_code}"
    fi
    exit "${exit_code}"
}
trap cleanup EXIT INT TERM

# Ensure single instance via lock file
acquire_lock() {
    if [[ -e "${LOCK_FILE}" ]]; then
        log "WARN" "Script is already running (PID: $(cat "${LOCK_FILE}")). Exiting."
        exit 0
    fi
    echo "$$" > "${LOCK_FILE}"
}

# Main execution logic
main() {
    acquire_lock
    log "INFO" "Starting backup synchronization process..."

    # Example workload
    local source_dir="/opt/app/data"
    local backup_target="/tmp/backup_$(date +%Y%m%d).tar.gz"

    if [[ -d "${source_dir}" ]]; then
        tar -czf "${backup_target}" -C "${source_dir}" .
        log "INFO" "Backup archive created at ${backup_target}"
    else
        log "WARN" "Source directory ${source_dir} not found; skipping archive."
    fi

    log "INFO" "Process completed successfully."
}

main "$@"

```
---

## TABLE of Symbols used in BASH scripting:

These are the essential Bash scripting symbols and their functions for your reference documentation.

| Symbol | Name | Usage / Description |
| --- | --- | --- |
| `#!` | Shebang | Placed at the absolute beginning of a script to specify the interpreter (e.g., `#!/usr/bin/env bash`). |
| `#` | Hash / Pound | Starts a line comment (ignored by the shell), except when used in the shebang. |
| `$` | Dollar Sign | Expands variables (e.g., `$VAR`) and references special shell parameters. \vert{} \vert{} `${}` |
| ``` | Backticks | Legacy syntax for command substitution (mostly replaced by `$()`). |
| `[]` | Single Brackets | Standard POSIX test command for conditional evaluations (`if [ condition ]`). |
| `[[]]` | Double Brackets | Modern Bash test syntax supporting regex pattern matching (`=~`) and logical operators. |
| `(())` | Double Parentheses | Evaluates arithmetic expressions and C-style `for` loops (`for ((i=0; i<3; i++))`). |
| `&` | Ampersand | Placed at the end of a command to run it in the background. |
| `$!` | Background PID | Stores the Process ID (PID) of the most recent background command. |
| `$?` | Exit Status | Stores the exit code of the last executed command (`0` means success). |
| `$$` | Process ID (PID) | Stores the PID of the currently running script. |
| `$0`, `$1`, `$2` | Positional Parameters | Reference script execution arguments: `$0` is the script name, `$1` is the first argument, etc. |
| `$#` | Argument Count | Stores the total number of arguments passed to the script. |
| `$@`, `$*` | All Arguments | Expands to all positional parameters passed to the script. |
| `|` | Pipe | Connects the standard output (stdout) of one command directly to the standard input (stdin) of the next. |
| `>`, `>>` | Output Redirection | Redirects output into a file. `>` overwrites the file; `>>` appends to it. |
| `<` | Input Redirection | Feeds the contents of a file into a command's standard input. |
| `2>` | Error Redirection | Redirects standard error (stderr) to a file or stream. |
| `&>` | Combined Redirection | Redirects both standard output (stdout) and standard error (stderr) to the same location. |
| `&&` | Logical AND | Executes the right-hand command *only* if the left-hand command succeeds. |
| `||` | Logical OR | Executes the right-hand command *only* if the left-hand command fails. |
| `!` | Logical NOT | Inverts a test condition or the exit status of a pipeline. |
| `;` | Semicolon | Statement separator. Allows multiple independent commands on the same line. |
| `\`, `'...'`, `"..."` | Escapes & Quotes | `\` escapes the next character. `'` makes everything literal. `"` allows variable expansion inside the string. |
| `~` | Tilde | Expands to the current user's home directory path. |
| `*`, `?` | Wildcards (Globbing) | `*` matches zero or more characters in paths/strings. `?` matches a single character. |

---




