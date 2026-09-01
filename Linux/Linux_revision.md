# 📖 Module 1: Linux Filesystem Hierarchy & Advanced Permissions

---

## 📁 1. Filesystem Hierarchy Standard (FHS)

Linux organizes files in a single unified tree starting at the root directory (`/`).

| Directory | Role & Purpose | Practical Examples |
| --- | --- | --- |
| **`/etc`** ⚙️ | Host-specific system configuration files | `/etc/passwd` (user info), `/etc/fstab` (mounts), `/etc/hosts` |
| **`/var`** 📈 | Variable data that grows over runtime | `/var/log/` (system logs), `/var/spool/mail`, `/var/lib/docker` |
| **`/tmp`** ⏳ | Temporary files cleared on system reboot | Scratch space for running applications |
| **`/opt`** 📦 | Optional third-party application software | Custom enterprise software (e.g., `/opt/google/chrome`) |
| **`/proc`** 🧠 | Virtual filesystem exposing kernel & process state | `/proc/cpuinfo`, `/proc/meminfo`, `/proc/<PID>/` |
| **`/dev`** 💽 | Device nodes representing hardware/virtual devices | `/dev/sda1` (disk partition), `/dev/null`, `/dev/urandom` |
| **`/sys`** 🔌 | Virtual filesystem exposing device driver subsystems | `/sys/class/net/` (network interface device states) |

---

## 🔒 2. Standard File Permissions

Permissions are defined for three user classes: **Owner (u)**, **Group (g)**, and **Others (o)**.

* 📖 **Read (`r` = 4):** View file contents or list directory contents.
* ✍️ **Write (`w` = 2):** Modify file contents or create/delete files in a directory.
* ⚡ **Execute (`x` = 1):** Run file as a program or enter/traverse a directory (`cd`).

### 🔹 Managing Ownership & Modes

```bash
# Set owner to rwx, group to r-x, others to r-x (755)
chmod 755 /opt/app/start.sh

# Change user and group ownership recursively
chown -R appuser:appgroup /opt/app/

```

---

## 🧮 3. `umask` (User Mask)

The **`umask`** determines the default permissions assigned to newly created files and directories by masking (subtracting) bits from the system base defaults.

* Base file permissions: `666` (`rw-rw-rw-`)
* Base directory permissions: `777` (`rwxrwxrwx`)

$$\text{Default Permissions} = \text{Base Mode} - \text{Umask}$$

```bash
# View current umask (e.g., 0022)
umask

# With umask 022:
# New file: 666 - 022 = 644 (rw-r--r--)
# New directory: 777 - 022 = 755 (rwxr-xr-x)

```

---

## 🛡️ 4. Special Permission Bits

Special permissions provide elevated execution privileges or access constraints beyond standard read/write/execute.

```
┌───────────────┬──────────────┬───────────────────────────────┬───────────────────────────────┐
│ Special Bit   │ Octal Value  │ File Behavior                 │ Directory Behavior            │
├───────────────┼──────────────┼───────────────────────────────┼───────────────────────────────┤
│ **SUID** 👑   │ `4000`       │ Runs with privileges of file  │ No standard directory effect  │
│ (SetUID)      │              │ owner (e.g., `/usr/bin/passwd`)│                               │
├───────────────┼──────────────┼───────────────────────────────┼───────────────────────────────┤
│ **SGID** 👥   │ `2000`       │ Runs with privileges of file  │ New files created inside      │
│ (SetGID)      │              │ group                         │ inherit parent directory group│
├───────────────┼──────────────┼───────────────────────────────┼───────────────────────────────┤
│ **Sticky** 📌 │ `1000`       │ No standard file effect       │ Only file owner or root can   │
│ Bit           │              │                               │ delete/rename files (`/tmp`)  │
└───────────────┴──────────────┴───────────────────────────────┴───────────────────────────────┘
```
---

# 📖 Module 2: Process Management, Signals & Systemd Services

---

## ⚙️ 1. Process Monitoring & Inspection

Every running program in Linux is represented by a unique **Process ID (PID)**.

| Command | Description | Common Syntax & Example |
| --- | --- | --- |
| **`ps`** 📋 | Snapshot of active processes | `ps aux` *(view all running processes with user/CPU/RAM details)* |
| **`top`** 📊 | Real-time dynamic resource monitor | `top` *(built-in command-line dashboard)* |
| **`htop`** 🖥️ | Enhanced interactive process viewer | `htop` *(color-coded, supports scrolling and mouse clicks)* |
| **`pgrep`** 🔍 | Find PID by process name | `pgrep -u appuser nginx` *(find Nginx PIDs for user)* |

### 🔹 Practical Process Inspection Examples

```bash
# View all processes in a standard BSD format
ps aux

# Sort top 5 processes by memory consumption
ps aux --sort=-%mem | head -n 6

# Sort top 5 processes by CPU consumption
ps aux --sort=-%cpu | head -n 6

```

---

## ⏳ 2. Job Control: Background & Foreground Execution

Linux allows you to manage long-running tasks within your shell session without blocking the terminal.

* 🟢 **`&` (Run in Background):** Starts a command directly in the background.
* ⏸️ **`Ctrl + Z`:** Suspends the currently running foreground process.
* 📋 **`jobs`:** Lists all active background jobs in the current shell.
* 🔄 **`bg` / `fg`:** Resumes a suspended job in the background (`bg %1`) or brings it back to the foreground (`fg %1`).
* 🛡️ **`nohup`:** Runs a process immune to hangups (`SIGHUP`), keeping it alive even if the SSH session closes.

```bash
# Run a backup script in the background, ignoring session logout
nohup /opt/scripts/backup.sh > /var/log/backup.log 2>&1 &

# View active shell jobs
jobs -l

# Bring job #1 back to the foreground
fg %1

```

---

## 📡 3. Signals & Terminating Processes

Signals are asynchronous notifications sent by the kernel or users to control process state.

```
┌──────────────┬────────┬────────────────────────────────────────────────────────┐
│ Signal Name  │ Number │ Action / Behavior                                      │
├──────────────┼────────┼────────────────────────────────────────────────────────┤
│ **SIGHUP**   │ `1`    │ Hangup detected; often triggers a config reload        │
│ **SIGINT**   │ `2`    │ Interrupt from keyboard (`Ctrl + C`)                   │
│ **SIGKILL**  │ `9`    │ Force-kill immediately (cannot be caught or ignored)  │
│ **SIGTERM**  │ `15`   │ Graceful termination request (allows cleanup)          │
│ **SIGSTOP**  │ `19`   │ Pause/suspend process execution (`Ctrl + Z`)           │
└──────────────┴────────┴────────────────────────────────────────────────────────┘

```

### 🔹 Termination Commands

```bash
# Send standard graceful shutdown signal (SIGTERM) to PID 4120
kill -15 4120

# Force-kill a stuck/unresponsive process (SIGKILL)
kill -9 4120

# Kill all processes matching a specific name
killall nginx

# Kill processes matching a pattern with a specific signal
pkill -15 -f "python main.py"

```

---

## 🛠️ 4. Systemd & Service Management (`systemctl`)

**Systemd** is the default init system and service manager (PID 1) in modern Linux distributions.

```
┌────────────────────────────────────────────────────────────────────────┐
│                      Common systemctl Operations                       │
├────────────────────────────────┬───────────────────────────────────────┤
│ `systemctl start <service>`    │ Start a service immediately           │
│ `systemctl stop <service>`     │ Stop a running service                │
│ `systemctl restart <service>`  │ Restart a service                     │
│ `systemctl reload <service>`   │ Reload config without dropping traffic│
│ `systemctl enable <service>`   │ Configure service to start at boot    │
│ `systemctl disable <service>`  │ Prevent service from starting at boot │
│ `systemctl status <service>`   │ Check operational status & last logs  │
└────────────────────────────────┴───────────────────────────────────────┘

```

### 🔹 Anatomy of a Custom Systemd Unit File

Path: `/etc/systemd/system/order-api.service`

```ini
[Unit]
Description=Order API Service
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/order-api
ExecStart=/usr/bin/python3 /opt/order-api/main.py
Restart=always
RestartSec=5
Environment=PORT=8080

[Install]
WantedBy=multi-user.target

```

```bash
# Reload systemd manager after modifying unit files
sudo systemctl daemon-reload

# Enable and start the service in one step
sudo systemctl enable --now order-api.service

```

---

## 📜 5. Inspecting System Logs with `journalctl`

`journalctl` queries logs collected by the `systemd-journald` service.

```bash
# Follow live logs for a specific service in real time
journalctl -u order-api.service -f

# View logs generated during the current boot cycle with priority ERROR and above
journalctl -b -p err

# View logs filtered by time window
journalctl -u order-api.service --since "1 hour ago"

# View system logs with explanatory metadata for debugging failures
journalctl -xe

```

---

# 📖 Module 3: I/O Redirection & Text Processing Tools

---

## 🔀 1. Standard Streams & I/O Redirection

In Linux, every process is initialized with three default data streams, represented by numeric **File Descriptors (FD)**:

* 📥 **Standard Input (`stdin` / FD `0`):** Keyboard/input stream.
* 📤 **Standard Output (`stdout` / FD `1`):** Default stream for standard output messages.
* ⚠️ **Standard Error (`stderr` / FD `2`):** Default stream for error and diagnostic messages.

```
                  ┌──────────────────────┐
   [ stdin (0) ] ─►│                      │─► [ stdout (1) ] ──► Terminal / File
                  │  Running Process     │
                  │                      │─► [ stderr (2) ] ──► Terminal / Error Log
                  └──────────────────────┘

```

### 🔹 Redirection Operators

| Operator | Function | Example |
| --- | --- | --- |
| **`>`** 📝 | Overwrite standard output to a file | `echo "Hello" > file.txt` |
| **`>>`** ➕ | Append standard output to a file | `echo "World" >> file.txt` |
| **`<`** 📥 | Read standard input from a file | `mysql db_name < backup.sql` |
| **`2>`** 🚫 | Redirect only standard error | `ls /root 2> error.log` |
| **`&>`** 📦 | Redirect both stdout and stderr together | `command &> all_output.log` |
| **`2>&1`** 🔀 | Send stderr to wherever stdout is currently pointing | `command > output.log 2>&1` |
| **`|`** ⛓️ | Pipe: Feed stdout of command 1 as stdin to command 2 | `cat /etc/passwd | grep bash` |

### 🔹 The Black Hole: `/dev/null`

`/dev/null` is a special virtual device that discards all data written to it.

```bash
# Silence error messages only (send stderr to /dev/null)
find / -name "*.conf" 2> /dev/null

# Silence all output completely (both stdout and stderr)
systemctl restart custom-service > /dev/null 2>&1

```

---

## 🛠️ 2. Essential Text Processing Utilities

Let's look at the core text processing tools used for parsing log files and data pipelines.

---

### 🔍 A. `grep` (Global Regular Expression Print)

Searches files or streams for lines matching a specified pattern.

```bash
# Case-insensitive search (-i) with line numbers (-n)
grep -in "error" /var/log/syslog

# Invert match (-v): Print lines that do NOT contain "DEBUG"
grep -v "DEBUG" app.log

# Recursive search (-r) across all files in a directory
grep -r "DB_PASSWORD" /etc/nginx/

# Extended regex (-E): Search for multiple alternative patterns
grep -E "404|500|502" access.log

```

---

### ✂️ B. `cut`

Extracts sections (fields or characters) from each line of a file.

```bash
# Extract usernames (field 1) from /etc/passwd using ':' as delimiter
cut -d':' -f1 /etc/passwd

# Extract fields 1 and 3 from a comma-separated file
cut -d',' -f1,3 data.csv

```

---

### 🔤 C. `tr` (Translate)

Translates, squeezes, or deletes characters from standard input.

```bash
# Convert lowercase text to uppercase
echo "hello world" | tr 'a-z' 'A-Z'

# Replace multiple consecutive spaces with a single space (-s)
echo "text    with    spaces" | tr -s ' '

# Delete specific characters (-d)
echo "order_id: #10492" | tr -d '#'

```

---

### 🔢 D. `sort` & `uniq`

Sorts lines of text and removes or counts duplicate lines.

> 📌 **Rule:** `uniq` only detects adjacent duplicate lines, so input **must be sorted** first.

```bash
# Sort numerically (-n) in reverse order (-r)
sort -nr scores.txt

# Count occurrences of unique IP addresses from a web access log
cut -d' ' -f1 access.log | sort | uniq -c

# Show only duplicate lines (-d)
sort list.txt | uniq -d

```

---

### 🔄 E. `sed` (Stream Editor)

Performs basic text transformations, substitutions, and stream filtering.

```bash
# Replace first occurrence of 'http' with 'https' on each line
sed 's/http/https/' config.txt

# Replace all occurrences globally ('g')
sed 's/old_endpoint/new_endpoint/g' app.conf

# In-place file edit (-i): Directly modifies the file on disk
sed -i 's/PORT=8080/PORT=443/g' .env

# Delete blank lines or lines matching a pattern
sed -i '/^$/d' clean_file.txt

```

---

### 📊 F. `awk` (Pattern Scanning & Processing)

A powerful language for column-oriented text processing.

* `$0`: The entire line.
* `$1`, `$2`, `$N`: The first, second, or $N$-th column.
* `NF`: Number of fields in the current line.
* `NR`: Current row/record number.

```bash
# Print only the 1st and 5th columns of a space-delimited output
df -h | awk '{print $1, $5}'

# Filter rows where memory usage (column 4) is greater than 50%
ps aux | awk '$4 > 50.0 {print $2, $11}'

# Custom field separator (-F): Parse /etc/passwd and format output
awk -F':' '{print "User: " $1 " | Home: " $6}' /etc/passwd

```

---

### 📦 G. `xargs` (Argument Builder)

Reads items from standard input and converts them into arguments for another command.

```bash
# Find all .log files and pass them to rm as batch arguments
find /var/log/app -name "*.log" | xargs rm -f

# Read a list of packages from a text file and install them
cat packages.txt | xargs sudo apt-get install -y

```

---

## 🔗 3. Production Pipeline Example

Here is how these tools combine to solve a real-world task—finding the **top 5 most frequent client IP addresses returning 500 errors** from an Nginx access log:

```bash
cat /var/log/nginx/access.log \
  | grep " 500 " \
  | awk '{print $1}' \
  | sort \
  | uniq -c \
  | sort -nr \
  | head -n 5

```

---

Let's test this with a quick scenario before we move to **Module 2: Process Lifecycle & Services**:

Suppose a shared team directory has permissions set to `drwxrwxrwx` without the Sticky bit. What risk does this create when multiple users collaborate in that directory?
