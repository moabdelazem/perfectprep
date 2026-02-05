# Bash Scripting

[← Back to Scripting Index](README.md) | [← Back to Main Index](../../README.md)

This section covers Bash scripting fundamentals for DevOps automation.

---

<details>
<summary><strong>Q1: What is the difference between Bash and Shell?</strong></summary>

**Shell** is a command-line interpreter that provides a user interface for the Unix/Linux operating system. **Bash** (Bourne Again SHell) is one of many shell implementations.

| Shell | Description |
|-------|-------------|
| **sh** | Original Bourne Shell |
| **bash** | Bourne Again Shell (most common on Linux) |
| **zsh** | Z Shell (default on macOS, powerful features) |
| **fish** | User-friendly interactive shell |
| **dash** | Lightweight POSIX shell |

**Shebang Line:**
```bash
#!/bin/bash    # Use Bash
#!/bin/sh      # Use default shell (POSIX compliant)
#!/usr/bin/env bash  # Portable way (finds bash in PATH)
```

**Check current shell:**
```bash
echo $SHELL     # Default login shell
echo $0         # Current shell
```

</details>

<details>
<summary><strong>Q2: Explain Bash variables and quoting</strong></summary>

**Variable Types:**
```bash
# Regular variable
NAME="John"
echo $NAME

# Environment variable (available to child processes)
export PATH="/usr/local/bin:$PATH"

# Special variables
$0    # Script name
$1-$9 # Positional parameters
$#    # Number of arguments
$@    # All arguments as separate words
$*    # All arguments as single string
$?    # Exit status of last command
$$    # Current process ID
$!    # PID of last background process
```

**Quoting:**
```bash
NAME="World"

# Double quotes - allows variable expansion
echo "Hello $NAME"     # Hello World
echo "Path is $PATH"   # expands $PATH

# Single quotes - literal string
echo 'Hello $NAME'     # Hello $NAME

# Backticks or $() - command substitution
echo "Today is $(date)"
echo "Files: `ls | wc -l`"

# Escape character
echo "Price: \$100"    # Price: $100
```

**Variable Operations:**
```bash
# Default values
echo ${VAR:-default}    # Use 'default' if VAR is unset
echo ${VAR:=default}    # Set VAR to 'default' if unset
echo ${VAR:+alternate}  # Use 'alternate' if VAR is set

# String operations
STR="Hello World"
echo ${#STR}            # Length: 11
echo ${STR:0:5}         # Substring: Hello
echo ${STR/World/Bash}  # Replace: Hello Bash
echo ${STR,,}           # Lowercase: hello world
echo ${STR^^}           # Uppercase: HELLO WORLD
```

</details>

<details>
<summary><strong>Q3: How do conditionals work in Bash?</strong></summary>

**If Statement:**
```bash
if [ condition ]; then
    # code
elif [ condition ]; then
    # code
else
    # code
fi
```

**Test Operators:**

| Type | Operator | Meaning |
|------|----------|---------|
| **String** | `-z "$str"` | String is empty |
| | `-n "$str"` | String is not empty |
| | `"$a" = "$b"` | Strings equal |
| | `"$a" != "$b"` | Strings not equal |
| **Numeric** | `-eq` | Equal |
| | `-ne` | Not equal |
| | `-lt` | Less than |
| | `-le` | Less than or equal |
| | `-gt` | Greater than |
| | `-ge` | Greater than or equal |
| **File** | `-f file` | File exists and is regular file |
| | `-d dir` | Directory exists |
| | `-e path` | Path exists |
| | `-r file` | File is readable |
| | `-w file` | File is writable |
| | `-x file` | File is executable |
| | `-s file` | File is not empty |
| **Logic** | `!` | NOT |
| | `-a` or `&&` | AND |
| | `-o` or `\|\|` | OR |

**Examples:**
```bash
# Check if file exists
if [ -f "/etc/passwd" ]; then
    echo "File exists"
fi

# Check if variable is set
if [ -n "$USER" ]; then
    echo "User is: $USER"
fi

# Numeric comparison
if [ "$count" -gt 10 ]; then
    echo "Count is greater than 10"
fi

# Multiple conditions with [[ ]] (Bash specific, preferred)
if [[ -f "$file" && -r "$file" ]]; then
    echo "File exists and is readable"
fi

# Pattern matching (Bash only)
if [[ "$name" == *.txt ]]; then
    echo "It's a text file"
fi
```

**Case Statement:**
```bash
case "$1" in
    start)
        echo "Starting..."
        ;;
    stop)
        echo "Stopping..."
        ;;
    restart)
        echo "Restarting..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

</details>

<details>
<summary><strong>Q4: Explain loops in Bash</strong></summary>

**For Loop:**
```bash
# Loop over list
for item in apple banana cherry; do
    echo "$item"
done

# Loop over range
for i in {1..5}; do
    echo "Number: $i"
done

# C-style for loop
for ((i=0; i<5; i++)); do
    echo "Index: $i"
done

# Loop over files
for file in *.txt; do
    echo "Processing: $file"
done

# Loop over command output
for user in $(cat /etc/passwd | cut -d: -f1); do
    echo "User: $user"
done

# Loop over array
SERVERS=("web1" "web2" "db1")
for server in "${SERVERS[@]}"; do
    echo "Server: $server"
done
```

**While Loop:**
```bash
# Basic while
count=0
while [ $count -lt 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Read file line by line (preferred method)
while IFS= read -r line; do
    echo "$line"
done < file.txt

# Infinite loop with break
while true; do
    read -p "Continue? (y/n): " answer
    if [ "$answer" = "n" ]; then
        break
    fi
done
```

**Until Loop:**
```bash
count=0
until [ $count -ge 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

**Loop Control:**
```bash
# break - exit loop
# continue - skip to next iteration
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        continue  # Skip 5
    fi
    if [ $i -eq 8 ]; then
        break     # Stop at 8
    fi
    echo $i
done
# Output: 1 2 3 4 6 7
```

</details>

<details>
<summary><strong>Q5: How do you write functions in Bash?</strong></summary>

**Function Definition:**
```bash
# Method 1
function greet() {
    echo "Hello, $1!"
}

# Method 2 (POSIX compatible)
greet() {
    echo "Hello, $1!"
}

# Call function
greet "World"
```

**Parameters and Return Values:**
```bash
# Parameters accessed via $1, $2, etc.
add_numbers() {
    local num1=$1
    local num2=$2
    local sum=$((num1 + num2))
    echo $sum  # Return via stdout
}

# Capture return value
result=$(add_numbers 5 3)
echo "Sum is: $result"

# Exit status (0-255)
is_valid() {
    if [ "$1" -gt 0 ]; then
        return 0  # Success
    else
        return 1  # Failure
    fi
}

if is_valid 5; then
    echo "Valid"
fi
```

**Local Variables:**
```bash
my_function() {
    local local_var="I'm local"
    global_var="I'm global"
    echo "$local_var"
}

my_function
echo "$global_var"   # Accessible
echo "$local_var"    # Empty (not accessible)
```

**Practical Function Example:**
```bash
#!/bin/bash

log() {
    local level=$1
    shift
    local message="$@"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] [$level] $message"
}

check_service() {
    local service=$1
    if systemctl is-active --quiet "$service"; then
        log INFO "$service is running"
        return 0
    else
        log ERROR "$service is not running"
        return 1
    fi
}

# Usage
check_service nginx || log WARN "Attempting restart"
```

</details>

<details>
<summary><strong>Q6: Explain text processing with grep, sed, and awk</strong></summary>

**grep - Pattern Searching:**
```bash
# Basic search
grep "error" logfile.txt

# Case insensitive
grep -i "ERROR" logfile.txt

# Show line numbers
grep -n "error" logfile.txt

# Count matches
grep -c "error" logfile.txt

# Invert match (lines NOT matching)
grep -v "debug" logfile.txt

# Recursive search
grep -r "TODO" /src/

# Extended regex
grep -E "error|warning|critical" logfile.txt

# Show context (2 lines before/after)
grep -B2 -A2 "error" logfile.txt

# Only filenames
grep -l "password" *.conf
```

**sed - Stream Editor:**
```bash
# Replace first occurrence per line
sed 's/old/new/' file.txt

# Replace all occurrences
sed 's/old/new/g' file.txt

# Replace in-place
sed -i 's/old/new/g' file.txt

# Delete lines matching pattern
sed '/pattern/d' file.txt

# Delete empty lines
sed '/^$/d' file.txt

# Print specific lines
sed -n '5,10p' file.txt

# Multiple operations
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file.txt

# Insert line before match
sed '/pattern/i\New line before' file.txt

# Append line after match
sed '/pattern/a\New line after' file.txt
```

**awk - Text Processing:**
```bash
# Print specific columns
awk '{print $1, $3}' file.txt

# Custom field separator
awk -F: '{print $1}' /etc/passwd

# Print with formatting
awk '{printf "%-10s %s\n", $1, $2}' file.txt

# Filter by condition
awk '$3 > 100 {print $1, $3}' file.txt

# Sum a column
awk '{sum += $3} END {print sum}' file.txt

# Count lines
awk 'END {print NR}' file.txt

# Print line numbers
awk '{print NR, $0}' file.txt

# Pattern matching
awk '/error/ {print $0}' logfile.txt

# Built-in variables
# NR = current line number
# NF = number of fields
# $0 = entire line
# $1, $2... = individual fields
# FS = field separator
# OFS = output field separator

# Complex example: Parse Apache logs
awk '{
    ip[$1]++
} END {
    for (i in ip) print ip[i], i
}' access.log | sort -rn | head -10
```

</details>

<details>
<summary><strong>Q7: How do you handle errors in Bash scripts?</strong></summary>

**Exit Codes:**
```bash
# Exit with code
exit 0   # Success
exit 1   # General error

# Check last command status
command
if [ $? -ne 0 ]; then
    echo "Command failed"
    exit 1
fi

# Or more concise
command || { echo "Command failed"; exit 1; }
```

**Set Options for Safety:**
```bash
#!/bin/bash
set -e          # Exit on any error
set -u          # Exit on undefined variable
set -o pipefail # Fail on pipe errors
set -x          # Debug mode (print commands)

# Combined (recommended for scripts)
set -euo pipefail
```

**Trap for Cleanup:**
```bash
#!/bin/bash
set -euo pipefail

TMPFILE=$(mktemp)

cleanup() {
    rm -f "$TMPFILE"
    echo "Cleanup complete"
}

# Run cleanup on EXIT, INT (Ctrl+C), TERM
trap cleanup EXIT INT TERM

# Main script
echo "Working..."
# ... do stuff with TMPFILE
```

**Error Handling Function:**
```bash
#!/bin/bash

die() {
    echo "ERROR: $1" >&2
    exit "${2:-1}"
}

check_root() {
    if [ "$(id -u)" -ne 0 ]; then
        die "This script must be run as root" 2
    fi
}

check_command() {
    command -v "$1" &>/dev/null || die "$1 is required but not installed"
}

# Usage
check_root
check_command docker
check_command kubectl
```

**Try-Catch Pattern:**
```bash
#!/bin/bash

try() {
    "$@"
    return $?
}

catch() {
    local exit_code=$?
    if [ $exit_code -ne 0 ]; then
        echo "Error: Command failed with exit code $exit_code" >&2
        return $exit_code
    fi
}

# Usage
if ! try some_command; then
    catch
    # Handle error
fi
```

</details>

<details>
<summary><strong>Q8: Explain I/O redirection and pipes</strong></summary>

**File Descriptors:**
```bash
0 = stdin  (standard input)
1 = stdout (standard output)
2 = stderr (standard error)
```

**Output Redirection:**
```bash
# Redirect stdout to file (overwrite)
command > file.txt

# Redirect stdout to file (append)
command >> file.txt

# Redirect stderr to file
command 2> error.log

# Redirect both stdout and stderr
command > output.txt 2>&1
command &> output.txt  # Shorthand (Bash)

# Redirect stderr to stdout
command 2>&1

# Suppress all output
command > /dev/null 2>&1
command &> /dev/null
```

**Input Redirection:**
```bash
# Read from file
command < input.txt

# Here document
cat << EOF
This is a
multi-line
string
EOF

# Here string
grep "pattern" <<< "search in this string"
```

**Pipes:**
```bash
# Connect commands
cat file.txt | grep "error" | wc -l

# Tee - write to file and stdout
command | tee output.txt
command | tee -a output.txt  # Append

# xargs - convert stdin to arguments
find . -name "*.log" | xargs rm
find . -name "*.log" -print0 | xargs -0 rm  # Handle spaces

# Process substitution
diff <(sort file1.txt) <(sort file2.txt)
```

**Named Pipes (FIFO):**
```bash
# Create named pipe
mkfifo my_pipe

# Terminal 1: Write to pipe
echo "Hello" > my_pipe

# Terminal 2: Read from pipe
cat < my_pipe
```

</details>

<details>
<summary><strong>Q9: How do you work with arrays in Bash?</strong></summary>

**Array Declaration:**
```bash
# Indexed array
FRUITS=("apple" "banana" "cherry")
SERVERS=(web1 web2 db1 db2)

# Associative array (Bash 4+)
declare -A PORTS
PORTS[http]=80
PORTS[https]=443
PORTS[ssh]=22
```

**Array Operations:**
```bash
# Access elements
echo ${FRUITS[0]}          # First element
echo ${FRUITS[-1]}         # Last element (Bash 4.3+)
echo ${FRUITS[@]}          # All elements
echo ${!FRUITS[@]}         # All indices

# Array length
echo ${#FRUITS[@]}

# Add elements
FRUITS+=("mango")

# Remove element
unset FRUITS[1]

# Slice
echo ${FRUITS[@]:1:2}      # Elements 1-2

# Loop through array
for fruit in "${FRUITS[@]}"; do
    echo "$fruit"
done

# Loop with index
for i in "${!FRUITS[@]}"; do
    echo "$i: ${FRUITS[$i]}"
done
```

**Associative Array Example:**
```bash
#!/bin/bash
declare -A CONFIG

CONFIG[host]="localhost"
CONFIG[port]="8080"
CONFIG[user]="admin"

# Access
echo "Connecting to ${CONFIG[host]}:${CONFIG[port]}"

# Loop
for key in "${!CONFIG[@]}"; do
    echo "$key = ${CONFIG[$key]}"
done
```

</details>

<details>
<summary><strong>Q10: What are essential DevOps Bash scripts?</strong></summary>

**Log Rotation Script:**
```bash
#!/bin/bash
set -euo pipefail

LOG_DIR="/var/log/myapp"
MAX_AGE=7  # days

find "$LOG_DIR" -name "*.log" -type f -mtime +$MAX_AGE -delete
find "$LOG_DIR" -name "*.log" -type f -size +100M -exec gzip {} \;

echo "Log cleanup completed at $(date)"
```

**Health Check Script:**
```bash
#!/bin/bash
set -euo pipefail

SERVICES=("nginx" "postgresql" "redis")
ALERT_EMAIL="admin@example.com"

check_service() {
    if systemctl is-active --quiet "$1"; then
        echo "✓ $1 is running"
        return 0
    else
        echo "✗ $1 is NOT running"
        return 1
    fi
}

failed_services=()

for service in "${SERVICES[@]}"; do
    if ! check_service "$service"; then
        failed_services+=("$service")
    fi
done

if [ ${#failed_services[@]} -gt 0 ]; then
    echo "Failed services: ${failed_services[*]}" | mail -s "Service Alert" "$ALERT_EMAIL"
    exit 1
fi
```

**Backup Script:**
```bash
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/backups"
DB_NAME="myapp"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# Create backup
pg_dump "$DB_NAME" | gzip > "$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz"

# Remove old backups
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

# Sync to S3
aws s3 sync "$BACKUP_DIR" "s3://my-backups/db/"

echo "Backup completed: ${DB_NAME}_${DATE}.sql.gz"
```

**Deployment Script:**
```bash
#!/bin/bash
set -euo pipefail

APP_DIR="/var/www/myapp"
REPO="git@github.com:company/myapp.git"
BRANCH="${1:-main}"

cd "$APP_DIR"

# Pull latest code
git fetch origin
git checkout "$BRANCH"
git pull origin "$BRANCH"

# Install dependencies
npm ci --production

# Run migrations
npm run migrate

# Restart application
pm2 reload myapp

# Health check
sleep 5
if curl -sf http://localhost:3000/health > /dev/null; then
    echo "Deployment successful!"
else
    echo "Health check failed!"
    exit 1
fi
```

</details>
