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
### I installed Nginx, then what should we do?

1. `sudo systemctl enable --now nginx`

If `--now` is not supported
1. `sudo systemctl start nginx`
2. `sudo systemctl enable nginx`

If you only run `sudo systemctl start nginx`:

* **Right now:** Nginx will turn on immediately. Your web server will be active, and any websites you are hosting will work perfectly.
* **The catch:** If your server ever restarts (due to a power outage, system updates, or a crash), Nginx will stay off. Your websites will remain offline until you manually log back in and run that exact same command again.

Running only `start` is perfectly fine if you are just playing around or temporarily testing something. However, for a real website that needs to stay online 24/7, you want it to recover automatically after a reboot, which is why `enable` is necessary.


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

**`/dev/null`** is a special virtual device that discards all data written to it.

**How it Works**

* **Writing:** Any data sent to `/dev/null` is instantly destroyed. It takes up no disk space and saves no logs.
* **Reading:** If a process attempts to read from it, it immediately returns an End-Of-File (EOF) signal, meaning it provides absolutely zero data.

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

# 📖 Module 4: Linux Filesystem Operations, Inodes & Storage Management

---

## 💽 1. Disk & Directory Space Analysis

Understanding how storage is allocated and consumed is vital for preventing production outages.

| Command | Purpose | Key Flags & Practical Example |
| --- | --- | --- |
| **`df`** 📊 | **Disk Free:** Reports filesystem disk space usage | `df -h` *(human-readable: GB/MB)*<br>

<br>`df -T` *(print filesystem type: ext4, xfs)* |
| **`du`** 📁 | **Disk Usage:** Estimates space used by files/directories | `du -sh /var/log` *(summary in human-readable format)*<br>

<br>`du -h --max-depth=1 /var` |
| **`lsblk`** 🗂️ | Lists all block storage devices and partitions | `lsblk -f` *(displays UUIDs and filesystem formats)* |

### 🔹 Practical Storage Inspection

```bash
# Check available space across all mounted filesystems
df -h

# Find the top 5 largest directories under /var
sudo du -h --max-depth=1 /var 2>/dev/null | sort -hr | head -n 5

```

---

## 🧠 2. Inodes, Hard Links & Soft Links

Every file in a Linux filesystem is represented internally by an **Inode** (Index Node), which stores file metadata (size, permissions, owner, timestamps, and data block pointers), but **not** the file name or actual file content.

```
┌─────────────────────────────────────────────────────────────┐
│                        Inode Data                           │
├───────────────────┬─────────────────────────────────────────┤
│ Inode Number      │ Unique ID on the filesystem (e.g., 104) │
│ Metadata          │ Permissions (755), Owner (1000:1000)    │
│ Pointers          │ Points to data blocks on physical disk  │
└───────────────────┴─────────────────────────────────────────┘

```

### 🔹 Hard Links vs. Soft (Symbolic) Links

| Characteristic | 🔗 Hard Link | 🔀 Soft Link (Symlink) |
| --- | --- | --- |
| **Target** | Points directly to the **Inode number** | Points to the **File path/name** |
| **Creation** | `ln file.txt hardlink.txt` | `ln -s file.txt symlink.txt` |
| **Across Filesystems?** | ❌ No (restricted to same filesystem) | ✅ Yes |
| **Original Deleted?** | ✅ Content stays accessible via hard link | ❌ Link breaks (dangling symlink) |
| **Directories?** | ❌ Not permitted for users | ✅ Supported |

### 🔹 Inode Exhaustion (`df -i`)

A filesystem can run out of space even if it has gigabytes of free disk capacity if all available **inodes** are consumed by millions of tiny files.

```bash
# Check inode utilization percentage
df -i

# Find directories containing huge numbers of files
find / -xdev -printf '%h\n' | sort | uniq -c | sort -k 1 -n | tail -n 10

```

---

## 🧰 3. Advanced Searching with `find`

The `find` utility traverses directory trees to locate files matching specific criteria.

```bash
# 1. Find files by name (case-insensitive)
find /etc -iname "*.conf"

# 2. Find files larger than 100MB
find /var/log -type f -size +100M

# 3. Find files modified in the last 7 days
find /opt/app -type f -mtime -7

# 4. Find and delete files older than 30 days
find /tmp -type f -mtime +30 -exec rm -f {} \;

# 5. Find files by permission mode (e.g., world-writable)
find /var/www -type f -perm 0777

```

---

## 🔌 4. Mounting & Managing Filesystems

Before a partition or disk can be accessed, it must be **mounted** onto a directory (mount point).

```
[ New Disk: /dev/sdb1 ] ──► ( Mount ) ──► [ Directory: /mnt/data ]

```

```bash
# Mount a partition to a directory
sudo mount /dev/sdb1 /mnt/data

# Unmount safely
sudo umount /mnt/data

# View all currently mounted filesystems
mount | column -t

```

### 🔹 Persistent Mounts: `/etc/fstab`

To automatically mount disks at boot time, add an entry to `/etc/fstab`:

```text
# <file system>                           <mount point>   <type>  <options>       <dump>  <pass>
UUID=3a1b2c3d-4e5f-6a7b-8c9d-0e1f2a3b4c5d  /mnt/data       ext4    defaults,noatime  0       2

```

```bash
# Test fstab entries without rebooting (mounts everything listed in fstab)
sudo mount -a

```

---

## 📜 5. Log Files & Log Rotation (`logrotate`)

System and application logs live in `/var/log/`. To prevent these logs from consuming all disk space, the system uses **`logrotate`**.

* 📁 Configuration directory: `/etc/logrotate.d/`
* ⚙️ Main config: `/etc/logrotate.conf`

### 🔹 Sample `logrotate` Configuration (`/etc/logrotate.d/nginx`)

```text
/var/log/nginx/*.log {
    daily                   # Rotate logs every day
    missingok               # Don't error if file is missing
    rotate 14               # Keep 14 historical log files
    compress                # Compress rotated files (.gz)
    delaycompress           # Defer compression to next rotation cycle
    notifempty              # Do not rotate empty files
    create 0640 www-data adm# Create new empty log with these permissions
    sharedscripts
    postrotate
        systemctl reload nginx > /dev/null 2>/dev/null || true
    endscript
}

```

```bash
# Force a dry-run test of logrotate configuration
sudo logrotate -d /etc/logrotate.d/nginx

# Force immediate execution
sudo logrotate -f /etc/logrotate.d/nginx

```

---

# 📖 Module 5: Linux Networking & Port Management

---

## 🌐 1. IP & Interface Configuration (`ip`)

The **`ip`** command (from the `iproute2` package) replaces legacy tools like `ifconfig` and `route` for managing network interfaces, IP addresses, and routing tables.

| Subcommand | Purpose | Practical Example |
| --- | --- | --- |
| **`ip addr`** (or `ip a`) 🏷️ | Show IP addresses assigned to all network interfaces | `ip a show eth0` *(view IP details for interface eth0)* |
| **`ip link`** 🔌 | View and modify the state of network interfaces | `sudo ip link set eth0 up` *(bring interface up)* |
| **`ip route`** (or `ip r`) 🗺️ | View and manage the kernel routing table | `ip route show` *(check the default gateway)* |

### 🔹 Practical Examples

```bash
# Display all network interfaces with assigned IPv4 and IPv6 addresses
ip -br a

# View the default gateway and routing paths
ip route show

# Temporarily assign an IP address to an interface
sudo ip addr add 192.168.1.50/24 dev eth0

```

---

## 🔌 2. Sockets & Active Connections (`ss`)

The **`ss`** (Socket Statistics) command inspects network sockets and listening ports, replacing the older `netstat` tool.

```
┌─────────────────────────────────────────────────────────────┐
│                   Common `ss` Flag Summary                  │
├───────┬─────────────────────────────────────────────────────┤
│ `-t`  │ TCP sockets                                         │
│ `-u`  │ UDP sockets                                         │
│ `-l`  │ Listening sockets (ports currently waiting for traffic)│
│ `-n`  │ Numeric display (show port numbers instead of names)│
│ `-p`  │ Process ID and program name using the socket        │
└───────┴─────────────────────────────────────────────────────┘

```

### 🔹 Practical Port Inspection

```bash
# View all active listening TCP and UDP ports with process names
sudo ss -tulnp

# Check if a specific port (e.g., port 8080) is currently listening
sudo ss -tulpn | grep ':8080'

# List all established outbound TCP connections
ss -t state established

```

---

## 📡 3. Testing Connectivity & HTTP Requests (`ping`, `curl`)

### 🔹 `ping` (ICMP Echo)

Tests reachability between hosts on an IP network and measures round-trip time.

```bash
# Send 4 ICMP echo requests to verify network connectivity
ping -c 4 8.8.8.8

# Test local network stack health (loopback interface)
ping -c 2 127.0.0.1

```

### 🔹 `curl` (Client URL)

Transfers data to or from a network server using protocols like HTTP, HTTPS, and FTP.

```bash
# Send an HTTP GET request and print response headers only (-I)
curl -I https://example.com

# Send a POST request with a JSON payload and custom headers
curl -X POST https://api.example.com/orders \
  -H "Content-Type: application/json" \
  -d '{"item": "book", "qty": 1}'

# Follow HTTP redirects (-L) and measure total response time
curl -s -w 'Total Time: %{time_total}s\n' -o /dev/null -L https://example.com

```

---

## 🗺️ 4. DNS Resolution Basics

Domain Name System (DNS) translates human-friendly hostnames (e.g., `google.com`) into IP addresses (e.g., `142.250.190.46`).

```
[ Application ] ──► [ /etc/hosts ] ──► [ /etc/resolv.conf ] ──► [ Upstream DNS Server ]

```

### 🔹 Key DNS Files

* **`/etc/hosts`** 📝: Local static mapping of hostnames to IP addresses (evaluated before DNS queries).
* **`/etc/resolv.conf`** ⚙️: Configures nameserver IP addresses used for system DNS lookups.
* **`/etc/nsswitch.conf`** 🔀: Defines the lookup order for name resolution (e.g., `hosts: files dns`).

### 🔹 DNS Query Tools

```bash
# Query DNS records for a domain using dig
dig example.com +short

# Perform a reverse DNS lookup (find hostname from IP)
dig -x 8.8.8.8 +short

# Query a specific DNS server directly (e.g., Cloudflare DNS at 1.1.1.1)
dig @1.1.1.1 example.com

```

---

## 🚪 5. Ports & Protocols

Network ports are 16-bit numbers ($0$ to $65535$) identifying specific communication endpoints on an operating system.

| Port Range | Category | Description |
| --- | --- | --- |
| **$0 - 1023$** 🔒 | **Well-Known Ports** | Reserved for privileged/system services (requires root to bind) |
| **$1024 - 49151$** ⚙️ | **Registered Ports** | User processes and custom application services |
| **$49152 - 65535$** 🔄 | **Dynamic / Ephemeral Ports** | Temporary ports used by clients for outbound requests |

### 🔹 Common Standard Ports

```
┌──────┬──────────┬───────────────────────────────────────────┐
│ Port │ Protocol │ Common Service                            │
├──────┼──────────┼───────────────────────────────────────────┤
│ 22   │ TCP      │ SSH (Secure Shell)                        │
│ 53   │ UDP/TCP  │ DNS (Domain Name System)                  │
│ 80   │ TCP      │ HTTP (Unencrypted Web)                    │
│ 443  │ TCP      │ HTTPS (TLS Encrypted Web)                 │
│ 3306 │ TCP      │ MySQL Database                            │
│ 5432 │ TCP      │ PostgreSQL Database                       │
│ 6379 │ TCP      │ Redis In-Memory Store                     │
└──────┴──────────┴───────────────────────────────────────────┘

```

---

# 📖 Module 6: System Resource Troubleshooting & Limits

---

## ⚡ 1. CPU Bottleneck Troubleshooting

CPU issues generally fall into two categories: **high utilization** (heavy computation) or **high load average** (processes waiting for CPU time or disk I/O).

### 🔹 Key Metrics & Load Average

* 📈 **Load Average:** The average number of processes in a *runnable* or *uninterruptible sleep* state over 1, 5, and 15 minutes.
* 📏 **Rule of Thumb:** A load average equal to your total number of CPU cores means $100\%$ optimal utilization. Anything significantly higher means processes are queuing up and waiting.

```
Total Cores = 4
Load Average: 4.00  ➔ 100% capacity (ideal utilization)
Load Average: 8.00  ➔ System is overloaded (4 processes waiting in queue on average)

```

### 🔹 Diagnostic Commands

| Command | Purpose | Useful Flags & Syntax |
| --- | --- | --- |
| **`uptime`** ⏱️ | View system uptime and 1, 5, 15 min load averages | `uptime` |
| **`top` / `htop**` 📊 | Identify top CPU-consuming processes | Press `P` inside `top` to sort by `%CPU` |
| **`mpstat`** 🧠 | Per-core CPU utilization breakdown | `mpstat -P ALL 1 3` *(report per-core stats 3 times, every 1 sec)* |
| **`pidstat`** 🔍 | CPU usage per process over time | `pidstat -u 1 5` *(monitor process CPU every 1 sec for 5 intervals)* |

```bash
# Check how many CPU cores are available on the system
nproc
# or
lscpu | grep "CPU(s):"

# View CPU breakdown (user, system, iowait, idle) per core
mpstat -P ALL 1 1

```

---

## 🧠 2. Memory & Swap Troubleshooting

When physical RAM runs out, the kernel moves memory pages to **Swap space** on the disk. Excessive swapping degrades performance drastically because disk access is much slower than RAM.

```
┌─────────────────────────────────────────────────────────────┐
│                    Memory Usage States                      │
├──────────────┬──────────────────────────────────────────────┤
│ **Used**     │ Actively allocated by applications and OS    │
│ **Buffers**  │ In-flight raw disk blocks waiting to write   │
│ **Cached**   │ Page cache (files read from disk kept in RAM)│
│ **Available**│ RAM immediately available without swapping   │
└──────────────┴──────────────────────────────────────────────┘

```

### 🔹 Diagnostic Commands

| Command | Purpose | Useful Flags & Syntax |
| --- | --- | --- |
| **`free`** 📉 | Quick snapshot of total, used, free, and swap memory | `free -m` *(Megabytes)* or `free -h` *(Human-readable)* |
| **`vmstat`** 🔄 | Virtual memory, swap in (`si`), and swap out (`so`) stats | `vmstat 1 5` *(report every 1 second)* |

```bash
# Display memory in human-readable units
free -h

# Check swap activity (if si/so > 0 constantly, memory is starving)
vmstat 1 5

# Check if the kernel Out-Of-Memory (OOM) killer terminated any processes
sudo dmesg -T | grep -i -E "oom|killed process"

```

---

## 💽 3. Disk I/O & Bottlenecks

High CPU `iowait` (`%wa` in `top`) means the CPU is sitting idle waiting for slow storage read/write operations to complete.

### 🔹 Diagnostic Commands

| Command | Purpose | Useful Flags & Syntax |
| --- | --- | --- |
| **`iostat`** 📊 | Storage device read/write throughput and latency | `iostat -xz 1 3` *(extended device stats)* |
| **`iotop`** 🔍 | Interactive view of top I/O-consuming processes | `sudo iotop -o` *(show only processes actively doing I/O)* |
| **`lsof`** 📂 | List open files by process or file path | `sudo lsof +D /var/log` *(find open files in a directory)* |

```bash
# View detailed disk I/O metrics
# Look at: %util (device saturation) and await (average I/O response time in ms)
iostat -xz 1 5

# Find which processes are generating the most disk writes right now
sudo iotop -oPa

```

---

## 🛡️ 4. Resource Limits (`ulimit` & `/etc/security/limits.conf`)

Linux enforces limits on system resources to prevent a single buggy or malicious process from exhausting file descriptors, memory, or process tables.

### 🔹 Soft Limits vs. Hard Limits

* 🟡 **Soft Limit:** The current operational limit enforced by the OS. Non-root users can increase this up to the hard limit.
* 🔴 **Hard Limit:** The maximum ceiling set by the root user/administrator. Non-root users cannot exceed this.

```
┌─────────────────────────────────────────────────────────────┐
│                   Common `ulimit` Flags                     │
├───────┬─────────────────────────────────────────────────────┤
│ `-n`  │ Maximum number of open file descriptors (sockets)   │
│ `-u`  │ Maximum number of processes per user (max user proc)│
│ `-v`  │ Maximum virtual memory available to the process     │
│ `-c`  │ Core dump file size limit                           │
│ `-a`  │ View all current limits for the session             │
└───────┴─────────────────────────────────────────────────────┘

```

### 🔹 Inspecting and Setting Limits

```bash
# View all active limits in the current shell
ulimit -a

# View soft limit for open files
ulimit -Sn

# View hard limit for open files
ulimit -Hn

# Temporarily increase open file limit for the current shell session
ulimit -n 65535

```

### 🔹 Persistent Configuration (`/etc/security/limits.conf`)

To make resource limits permanent across reboots and user logins, add entries to `/etc/security/limits.conf`:

```text
# <domain/user>  <type>   <item>   <value>
*                soft     nofile   65535
*                hard     nofile   65535
appuser          soft     nproc    4096
appuser          hard     nproc    8192

```

---

## 📋 5. Rapid Troubleshooting Checklist

When a server becomes slow or unresponsive, follow this standard triage order:

1. ⏱️ **Check Load & CPU:** Run `uptime` and `top`. Is `%wa` (I/O wait) high or is `%usr`/`%sys` high?
2. 🧠 **Check RAM & Swap:** Run `free -m` and `vmstat 1`. Is swap actively thrashing (`si`/`so`)?
3. 💽 **Check Disk Space & I/O:** Run `df -h`, `df -i`, and `iostat -xz 1`. Is a disk $100\%$ full or saturated?
4. 🌐 **Check Sockets & Limits:** Run `ss -tulnp` and check `ulimit -n` if the service reports "Too many open files".

---

Let's test this with a scenario:

Suppose a high-traffic web server starts returning errors, and the application log displays:

`Error: socket: too many open files`

Which specific resource limit was reached, and which command would you run to see the current limit for that user?

## INTERVIEW QUESTIONS:

1. I have a **nginx webserver**, I want to make some traffic to it. What command I can run?

Ans:
**Basic Continuous Requests (No Installation Required)**
Use a standard `while` loop with `curl` to continuously hit your server. The `-s` flag silences the progress meter, and redirecting the output to `/dev/null` discards the downloaded HTML to prevent terminal spam.

```bash
while true; do curl -s -o /dev/null http://<your-server-ip>/; sleep 1; done

```

- The web server will see this as the exact same user hitting the IP repeatedly.

*(Remove `sleep 1` if you want to hammer the server as fast as your single CPU thread allows).*

In bash, the **`;` (semicolon)** acts as a **command separator**. It allows you to execute multiple commands sequentially on a single line instead of pressing "Enter" after each one.

* **`while true; do ... done`**: This creates an infinite loop. Because "true" is always true, the loop will run endlessly until you manually terminate it (usually by pressing `Ctrl + C`).
* **`curl -s -o /dev/null http://<your-server-ip>/`**: This makes the actual web request to your server.
* `-s` (silent) hides the download progress bar.
* `-o /dev/null` throws away the downloaded webpage HTML so it doesn't flood your terminal with text.


* **`sleep 1`**: Pauses the loop for exactly 1 second before starting the next iteration. This ensures you only send one request per second, rather than hammering the server as fast as your computer can process the loop.

**Concurrent Traffic Generation (Apache Benchmark)**
To simulate multiple users hitting the server at the exact same time, use `ab`.

```bash
# Install on Ubuntu/Debian: sudo apt install apache2-utils
ab -n 10000 -c 100 http://<your-server-ip>/

```

* **`-n 10000`**: Total number of requests to send.
* **`-c 100`**: Number of concurrent connections to keep open at once.

**Sustained Traffic Over Time (Siege)**
If you want to simulate a steady stream of traffic for a specific duration rather than a fixed number of requests, `siege` is ideal.

```bash
# Install on Ubuntu/Debian: sudo apt install siege
siege -c 50 -t 2M http://<your-server-ip>/

```

* **`-c 50`**: Simulates 50 concurrent users.
* **`-t 2M`**: Runs the traffic generator continuously for 2 minutes.
