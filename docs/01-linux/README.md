# Linux & System Administration

[← Back to Index](../../README.md)

This section covers essential Linux knowledge for DevOps engineers including file systems, permissions, process management, systemd, and more.

---

<details>
<summary><strong>Q1: What is Linux and why is it preferred for servers?</strong></summary>

Linux is an open-source, Unix-like operating system kernel. Combined with GNU tools, it forms a complete OS (GNU/Linux).

**Why Linux for servers:**

| Advantage | Description |
|-----------|-------------|
| **Stability** | Can run for years without rebooting |
| **Security** | Strong permission model, frequent patches |
| **Performance** | Lightweight, efficient resource usage |
| **Cost** | Free and open-source |
| **Flexibility** | Highly customizable, many distributions |
| **CLI-first** | Designed for scripting and automation |

**Common Server Distributions:**

| Distro | Use Case |
|--------|----------|
| Ubuntu Server | General purpose, beginner-friendly |
| RHEL/CentOS/Rocky | Enterprise, long-term support |
| Debian | Stability-focused, minimal |
| Alpine | Containers (tiny footprint ~5MB) |
| Amazon Linux | AWS-optimized |

</details>

<details>
<summary><strong>Q2: Explain the Linux file system hierarchy</strong></summary>

Everything in Linux is organized under the root directory `/`.

```
/
├── bin/      → Essential user binaries (ls, cp, cat)
├── sbin/     → System binaries (systemctl, fdisk)
├── etc/      → Configuration files
├── home/     → User home directories
├── root/     → Root user's home directory
├── var/      → Variable data (logs, databases, mail)
│   ├── log/  → System and application logs
│   └── www/  → Web server files
├── tmp/      → Temporary files (cleared on reboot)
├── usr/      → User programs and data
│   ├── bin/  → User binaries
│   ├── lib/  → Libraries
│   └── local/→ Locally installed software
├── opt/      → Optional/third-party software
├── dev/      → Device files (disks, terminals)
├── proc/     → Virtual filesystem for process info
├── sys/      → Virtual filesystem for kernel info
└── mnt/      → Mount point for temporary mounts
```

**Key Points:**
- `/etc` = Configuration (editable text config)
- `/var/log` = Where you look for logs
- `/proc` = Runtime system info (not real files)
- `/dev` = Hardware represented as files

</details>

<details>
<summary><strong>Q3: Explain Linux file permissions and how to modify them</strong></summary>

Every file has three permission sets for: **Owner**, **Group**, and **Others**.

**Permission Types:**

| Symbol | Numeric | Meaning |
|--------|---------|---------|
| `r` | 4 | Read |
| `w` | 2 | Write |
| `x` | 1 | Execute |
| `-` | 0 | No permission |

**Reading permissions:**
```bash
ls -l myfile.txt
# -rw-r--r-- 1 user group 1024 Jan 15 10:00 myfile.txt
#  │││ │││ │││
#  │││ │││ └── Others: r-- (read only)
#  │││ └───── Group: r-- (read only)
#  └──────── Owner: rw- (read + write)
```

**Changing permissions:**
```bash
# Symbolic mode
chmod u+x script.sh      # Add execute to owner
chmod g-w file.txt       # Remove write from group
chmod o=r file.txt       # Set others to read-only
chmod a+r file.txt       # Add read to all

# Numeric mode
chmod 755 script.sh      # rwxr-xr-x
chmod 644 file.txt       # rw-r--r--
chmod 600 secret.key     # rw-------
```

**Common Permission Patterns:**

| Numeric | Symbolic | Use Case |
|---------|----------|----------|
| 755 | rwxr-xr-x | Executable scripts, directories |
| 644 | rw-r--r-- | Regular files |
| 600 | rw------- | Private keys, secrets |
| 777 | rwxrwxrwx | AVOID - security risk |

**Changing ownership:**
```bash
chown user:group file.txt
chown -R user:group directory/
```

</details>

<details>
<summary><strong>Q4: What are hard links and soft links (symbolic links)?</strong></summary>

Both create references to files, but work differently.

**Soft Link (Symbolic Link):**
- A pointer/shortcut to the original file path
- Can link to directories
- Can cross filesystems
- Breaks if original is deleted/moved

```bash
ln -s /path/to/original /path/to/link
```

**Hard Link:**
- Another name for the same file data (same inode)
- Cannot link to directories
- Must be on same filesystem
- Data persists until ALL hard links are deleted

```bash
ln /path/to/original /path/to/hardlink
```

**Comparison:**

| Feature | Soft Link | Hard Link |
|---------|-----------|-----------|
| Points to | File path | Inode (data) |
| Cross filesystem | Yes | No |
| Link to directories | Yes | No |
| Original deleted | Link breaks | Data persists |
| File size | Small (just path) | Same as original |

**Identify links:**
```bash
ls -l
# lrwxrwxrwx ... link -> /original  (soft link, 'l' at start)
# -rw-r--r-- 2 ... file             (hard link count = 2)
```

</details>

<details>
<summary><strong>Q5: How do you find files and search content in Linux?</strong></summary>

**Finding files by name (find):**
```bash
# Find by name
find /path -name "*.log"
find / -name "nginx.conf"

# Find by type
find /var -type f -name "*.log"    # Files only
find /home -type d -name "config"   # Directories only

# Find by size
find / -size +100M                  # Larger than 100MB
find / -size -1k                    # Smaller than 1KB

# Find by time
find /var/log -mtime -7             # Modified in last 7 days
find /tmp -atime +30                # Accessed more than 30 days ago

# Find and execute
find /tmp -name "*.tmp" -delete
find . -name "*.sh" -exec chmod +x {} \;
```

**Searching file content (grep):**
```bash
# Basic search
grep "error" /var/log/syslog
grep -i "error" file.txt            # Case insensitive

# Recursive search
grep -r "TODO" /project/            # Search in all files
grep -rn "password" /etc/           # With line numbers

# Extended regex
grep -E "error|warning|critical" log.txt

# Inverse match
grep -v "debug" application.log     # Exclude debug lines

# Context
grep -A 3 -B 2 "error" log.txt      # 3 after, 2 before
```

**Fast file search (locate):**
```bash
# Uses pre-built database (faster than find)
locate nginx.conf

# Update database
sudo updatedb
```

</details>

<details>
<summary><strong>Q6: Explain process management in Linux</strong></summary>

A process is a running instance of a program.

**Process States:**
- **R** - Running
- **S** - Sleeping (waiting for event)
- **D** - Uninterruptible sleep (usually I/O)
- **Z** - Zombie (terminated but not reaped)
- **T** - Stopped

**Viewing processes:**
```bash
# All processes
ps aux
ps -ef

# Process tree
pstree

# Real-time monitoring
top
htop                    # Better UI

# Specific process
ps aux | grep nginx
pgrep nginx             # Returns PIDs
```

**Signals:**

| Signal | Number | Action |
|--------|--------|--------|
| SIGHUP | 1 | Reload configuration |
| SIGINT | 2 | Interrupt (Ctrl+C) |
| SIGKILL | 9 | Force kill (cannot be caught) |
| SIGTERM | 15 | Graceful termination |
| SIGSTOP | 19 | Pause process |
| SIGCONT | 18 | Resume process |

**Managing processes:**
```bash
# Kill process
kill PID                # Sends SIGTERM (graceful)
kill -9 PID             # Sends SIGKILL (force)
kill -HUP PID           # Reload config

# Kill by name
killall nginx
pkill -f "python script.py"

# Background/Foreground
command &               # Run in background
jobs                    # List background jobs
fg %1                   # Bring job 1 to foreground
bg %1                   # Resume job 1 in background
Ctrl+Z                  # Suspend current process
```

**Priority (nice):**
```bash
nice -n 10 command      # Start with lower priority
renice -n -5 -p PID     # Change priority of running process
# Range: -20 (highest) to 19 (lowest)
```

</details>

<details>
<summary><strong>Q7: What is systemd and how do you manage services?</strong></summary>

systemd is the init system and service manager for modern Linux distributions.

**Key Concepts:**
- **Unit**: Anything systemd manages (service, socket, timer, etc.)
- **Target**: Group of units (similar to runlevels)
- **Unit files**: Configuration files in `/etc/systemd/system/` or `/lib/systemd/system/`

**Managing services:**
```bash
# Status
systemctl status nginx
systemctl is-active nginx
systemctl is-enabled nginx

# Start/Stop
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx      # Reload config without restart

# Enable/Disable (start on boot)
systemctl enable nginx
systemctl disable nginx
systemctl enable --now nginx  # Enable and start immediately

# List services
systemctl list-units --type=service
systemctl list-units --state=failed
```

**Viewing logs (journalctl):**
```bash
# All logs
journalctl

# For specific service
journalctl -u nginx

# Follow logs (like tail -f)
journalctl -u nginx -f

# Since boot
journalctl -b

# Time-based
journalctl --since "1 hour ago"
journalctl --since "2024-01-15" --until "2024-01-16"

# Priority
journalctl -p err           # Errors and above
```

**Example unit file (`/etc/systemd/system/myapp.service`):**
```ini
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=appuser
ExecStart=/usr/bin/python3 /opt/app/main.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
# After creating/modifying unit file
systemctl daemon-reload
systemctl enable myapp
systemctl start myapp
```

</details>

<details>
<summary><strong>Q8: How do you check disk usage and manage storage?</strong></summary>

**Disk usage commands:**
```bash
# Filesystem disk space
df -h                       # Human-readable
df -hT                      # With filesystem type
df -h /var                  # Specific mount point

# Directory/file sizes
du -sh /var/log             # Summary of directory
du -h --max-depth=1 /var    # One level deep
du -ah /var | sort -rh | head -20  # Largest files

# Find large files
find / -type f -size +100M -exec ls -lh {} \;
```

**Disk management:**
```bash
# List block devices
lsblk
fdisk -l

# Mount/unmount filesystems
mount /dev/sdb1 /mnt/data
umount /mnt/data

# Persistent mounts (/etc/fstab)
# <device>  <mount>  <type>  <options>  <dump>  <pass>
/dev/sdb1   /data    ext4    defaults    0       2
```

**Check and repair:**
```bash
# Check filesystem (unmounted only)
fsck /dev/sdb1

# Check disk health
smartctl -a /dev/sda
```

**LVM (Logical Volume Manager):**
```bash
# View LVM
pvdisplay               # Physical volumes
vgdisplay               # Volume groups
lvdisplay               # Logical volumes

# Extend logical volume
lvextend -L +10G /dev/vg01/lv01
resize2fs /dev/vg01/lv01    # For ext4
```

</details>

<details>
<summary><strong>Q9: How do you check and manage memory in Linux?</strong></summary>

**View memory:**
```bash
# Quick overview
free -h

# Output explanation:
#               total    used    free   shared  buff/cache   available
# Mem:          16Gi    4Gi     2Gi    100Mi      10Gi        11Gi
# Swap:          2Gi    0Bi     2Gi
```

**Key memory concepts:**
- **Used**: Memory in use by applications
- **Free**: Completely unused memory
- **Buffers**: Cache for block device I/O
- **Cached**: Page cache for files
- **Available**: Memory available for new apps (free + reclaimable cache)

**Memory info:**
```bash
cat /proc/meminfo
vmstat 1                    # Real-time stats every 1 second
```

**Top memory consumers:**
```bash
# Using ps
ps aux --sort=-%mem | head

# Using top
top -o %MEM

# Clear page cache (if needed)
sync; echo 3 > /proc/sys/vm/drop_caches
```

**Swap management:**
```bash
# View swap
swapon --show

# Create swap file
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

# Add to /etc/fstab for persistence
/swapfile none swap sw 0 0
```

**OOM (Out of Memory) Killer:**
- When memory is exhausted, kernel kills processes
- Check OOM kills: `dmesg | grep -i "killed process"`
- Adjust OOM score: `/proc/PID/oom_score_adj`

</details>

<details>
<summary><strong>Q10: How do you monitor CPU performance?</strong></summary>

**View CPU info:**
```bash
# CPU details
lscpu
cat /proc/cpuinfo

# CPU count
nproc                   # Number of available processors
```

**Monitor CPU usage:**
```bash
# Real-time
top                     # Press '1' to see per-core
htop                    # Better visualization

# Snapshot
mpstat 1                # Per-second stats
vmstat 1                # System-wide stats

# Per-process
pidstat 1               # Per-process CPU usage
```

**Understanding load average:**
```bash
uptime
# 14:30:01 up 10 days, load average: 1.50, 2.00, 1.75
#                                    1min  5min  15min
```

**Load Average Interpretation:**
- On a 4-core system:
  - Load < 4: System is handling requests fine
  - Load = 4: All cores fully utilized
  - Load > 4: Processes waiting for CPU time

**Find high CPU processes:**
```bash
ps aux --sort=-%cpu | head -10
top -bn1 | head -15
```

**CPU stress testing:**
```bash
# Install stress
stress --cpu 4 --timeout 30s
```

</details>

<details>
<summary><strong>Q11: How do you manage users and groups?</strong></summary>

**User management:**
```bash
# Create user
useradd username
useradd -m -s /bin/bash username    # With home dir and shell

# Set password
passwd username

# Modify user
usermod -aG docker username         # Add to group
usermod -s /bin/zsh username        # Change shell
usermod -L username                 # Lock account
usermod -U username                 # Unlock account

# Delete user
userdel username
userdel -r username                 # With home directory
```

**Group management:**
```bash
# Create group
groupadd developers

# Add user to group
usermod -aG developers username
gpasswd -a username developers      # Alternative

# Remove from group
gpasswd -d username developers

# View groups
groups username
id username
```

**Important files:**
```bash
/etc/passwd     # User accounts (username:x:UID:GID:info:home:shell)
/etc/shadow     # Encrypted passwords
/etc/group      # Group definitions
/etc/sudoers    # Sudo permissions (edit with visudo)
```

**Sudo configuration:**
```bash
# Edit sudoers safely
visudo

# Common entries:
username ALL=(ALL) ALL              # Full sudo access
username ALL=(ALL) NOPASSWD: ALL    # No password required
%developers ALL=(ALL) /usr/bin/docker  # Group can run docker
```

</details>

<details>
<summary><strong>Q12: What is SSH and how do you configure it securely?</strong></summary>

SSH (Secure Shell) provides encrypted remote access to systems.

**Basic SSH usage:**
```bash
# Connect
ssh user@hostname
ssh -p 2222 user@hostname           # Custom port

# Execute remote command
ssh user@host "df -h"

# SCP (copy files)
scp file.txt user@host:/path/
scp user@host:/path/file.txt ./
scp -r directory/ user@host:/path/  # Recursive

# Rsync (better for large/incremental transfers)
rsync -avz directory/ user@host:/path/
```

**SSH key authentication:**
```bash
# Generate key pair
ssh-keygen -t ed25519 -C "your_email@example.com"
# Or RSA
ssh-keygen -t rsa -b 4096

# Copy public key to server
ssh-copy-id user@host
# Or manually: cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys

# SSH with specific key
ssh -i ~/.ssh/mykey user@host
```

**SSH config (`~/.ssh/config`):**
```
Host myserver
    HostName 192.168.1.100
    User admin
    Port 2222
    IdentityFile ~/.ssh/mykey

Host *
    ServerAliveInterval 60
```

**Secure SSH server (`/etc/ssh/sshd_config`):**
```bash
# Recommended settings:
Port 2222                           # Non-default port
PermitRootLogin no                  # Disable root login
PasswordAuthentication no           # Keys only
PubkeyAuthentication yes
MaxAuthTries 3
AllowUsers admin deploy             # Whitelist users

# Apply changes
systemctl restart sshd
```

</details>

<details>
<summary><strong>Q13: How do you check network configuration and troubleshoot connectivity?</strong></summary>

**View network configuration:**
```bash
# IP addresses
ip addr
ip a                                # Short form
hostname -I                         # Just IPs

# Routing table
ip route
route -n                            # Legacy

# DNS configuration
cat /etc/resolv.conf

# Network interfaces
ip link
```

**Connectivity troubleshooting:**
```bash
# Test connectivity
ping -c 4 google.com
ping -c 4 8.8.8.8                   # Bypass DNS

# Trace route
traceroute google.com
mtr google.com                      # Better tool

# DNS lookup
nslookup google.com
dig google.com
host google.com
dig +trace google.com               # Full DNS resolution path

# Check if port is open
nc -zv hostname 80                  # Netcat
telnet hostname 80
```

**Check listening ports:**
```bash
# All listening ports
ss -tuln
netstat -tuln                       # Legacy

# Which process is using a port
ss -tulnp
lsof -i :80
fuser 80/tcp
```

**Firewall (iptables/nftables):**
```bash
# View rules
iptables -L -n -v
iptables -L -t nat                  # NAT table

# Allow port
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Block IP
iptables -A INPUT -s 192.168.1.100 -j DROP

# Save rules
iptables-save > /etc/iptables/rules.v4
```

**Firewall (firewalld - RHEL/CentOS):**
```bash
firewall-cmd --list-all
firewall-cmd --add-port=80/tcp --permanent
firewall-cmd --reload
```

**Firewall (ufw - Ubuntu):**
```bash
ufw status
ufw allow 80/tcp
ufw enable
```

</details>

<details>
<summary><strong>Q14: How do you analyze logs in Linux?</strong></summary>

**Log locations:**
```bash
/var/log/syslog         # System logs (Debian/Ubuntu)
/var/log/messages       # System logs (RHEL/CentOS)
/var/log/auth.log       # Authentication logs
/var/log/kern.log       # Kernel logs
/var/log/dmesg          # Boot messages
/var/log/nginx/         # Application-specific logs
```

**Viewing logs:**
```bash
# Full file
cat /var/log/syslog
less /var/log/syslog

# Last N lines
tail -n 100 /var/log/syslog

# Follow in real-time
tail -f /var/log/syslog
tail -f /var/log/*.log              # Multiple files

# Multiple files with multitail
multitail /var/log/syslog /var/log/auth.log
```

**Filtering and searching:**
```bash
# Search for errors
grep -i "error" /var/log/syslog
grep -i "failed\|error\|critical" /var/log/syslog

# Extract time range
awk '/Feb 03 10:00/,/Feb 03 11:00/' /var/log/syslog

# Count occurrences
grep -c "error" /var/log/syslog

# Unique IPs from access log
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn
```

**journalctl (systemd):**
```bash
# All logs
journalctl

# Follow mode
journalctl -f

# Specific service
journalctl -u nginx -f

# Errors only
journalctl -p err

# Since time
journalctl --since "2 hours ago"
journalctl --since "2024-01-15 10:00" --until "2024-01-15 12:00"

# Kernel messages
journalctl -k

# Disk usage
journalctl --disk-usage
journalctl --vacuum-size=500M       # Limit log size
```

</details>

<details>
<summary><strong>Q15: What is /proc and /sys filesystem?</strong></summary>

Both are **virtual filesystems** - they don't exist on disk but are generated by the kernel.

**/proc - Process Information:**

| Path | Information |
|------|-------------|
| `/proc/cpuinfo` | CPU details |
| `/proc/meminfo` | Memory statistics |
| `/proc/loadavg` | System load averages |
| `/proc/uptime` | System uptime |
| `/proc/PID/` | Per-process information |
| `/proc/PID/cmdline` | Command that started process |
| `/proc/PID/fd/` | Open file descriptors |
| `/proc/PID/status` | Process status/memory |

```bash
# View process info
cat /proc/1234/cmdline
ls -l /proc/1234/fd             # Open files by process
cat /proc/1234/status | grep VmRSS   # Memory usage
```

**/sys - Kernel/Hardware Information:**

| Path | Purpose |
|------|---------|
| `/sys/class/net/` | Network interfaces |
| `/sys/block/` | Block devices |
| `/sys/devices/` | Hardware device tree |
| `/sys/fs/` | Filesystem parameters |

```bash
# Network interface info
cat /sys/class/net/eth0/address     # MAC address
cat /sys/class/net/eth0/speed       # Link speed

# Disk info
cat /sys/block/sda/size             # Disk size in sectors
```

**Runtime kernel tuning:**
```bash
# View setting
cat /proc/sys/net/ipv4/ip_forward

# Modify (temporary)
echo 1 > /proc/sys/net/ipv4/ip_forward

# Permanent (/etc/sysctl.conf)
net.ipv4.ip_forward = 1

# Apply
sysctl -p
```

</details>

<details>
<summary><strong>Q16: How do you troubleshoot a slow Linux server?</strong></summary>

**Systematic troubleshooting approach:**

**1. Check load and uptime:**
```bash
uptime
# High load? Check if CPU, memory, or I/O bound
```

**2. CPU check:**
```bash
top                         # Overall CPU, find high CPU processes
mpstat 1                    # Per-second, identify spikes
ps aux --sort=-%cpu | head  # Top CPU consumers
```

**3. Memory check:**
```bash
free -h                     # Total/available memory
vmstat 1                    # Watch for swapping (si/so columns)
ps aux --sort=-%mem | head  # Top memory consumers
```

**4. Disk I/O check:**
```bash
iostat -x 1                 # Disk I/O statistics
iotop                       # Per-process I/O (like top)
# High %util = disk is bottleneck
```

**5. Disk space check:**
```bash
df -h                       # Filesystem full?
du -sh /* 2>/dev/null | sort -h  # What's using space
```

**6. Network check:**
```bash
sar -n DEV 1                # Network throughput
ss -s                       # Socket summary
netstat -an | wc -l         # Connection count
```

**7. Check for runaway processes:**
```bash
ps aux --sort=-%cpu | head
dmesg | tail                # Kernel messages
journalctl -p err --since "1 hour ago"
```

**Quick diagnostic script:**
```bash
#!/bin/bash
echo "=== UPTIME ===" && uptime
echo "=== MEMORY ===" && free -h
echo "=== DISK ===" && df -h
echo "=== TOP PROCESSES ===" && ps aux --sort=-%cpu | head -5
echo "=== NETWORK ===" && ss -s
echo "=== ERRORS ===" && journalctl -p err --since "1 hour ago" | tail -10
```

</details>

<details>
<summary><strong>Q17: What is the difference between environment variables and shell variables?</strong></summary>

**Shell Variables:**
- Exist only in the current shell
- Not inherited by child processes
- Set with: `VAR="value"`

**Environment Variables:**
- Inherited by child processes
- Set with: `export VAR="value"`

```bash
# Shell variable (local)
MY_VAR="hello"
echo $MY_VAR        # Works
bash -c 'echo $MY_VAR'   # Empty - not exported

# Environment variable (global)
export MY_VAR="hello"
bash -c 'echo $MY_VAR'   # Works - inherited
```

**Common environment variables:**

| Variable | Purpose |
|----------|---------|
| `PATH` | Directories to search for commands |
| `HOME` | User's home directory |
| `USER` | Current username |
| `SHELL` | Default shell |
| `PWD` | Current directory |
| `EDITOR` | Default text editor |
| `LANG` | System language/locale |

**Setting permanently:**
```bash
# User-specific (~/.bashrc or ~/.bash_profile)
export PATH="$PATH:/opt/myapp/bin"
export EDITOR="vim"

# System-wide (/etc/environment or /etc/profile.d/custom.sh)
export JAVA_HOME="/usr/lib/jvm/java-11"
```

**View all:**
```bash
env                 # Environment variables
set                 # All variables (including shell)
printenv PATH       # Specific variable
```

</details>

<details>
<summary><strong>Q18: How do you schedule tasks in Linux?</strong></summary>

**Cron (time-based scheduling):**

```bash
# Edit user crontab
crontab -e

# List crontab
crontab -l

# Cron format:
# ┌───────── minute (0 - 59)
# │ ┌─────── hour (0 - 23)
# │ │ ┌───── day of month (1 - 31)
# │ │ │ ┌─── month (1 - 12)
# │ │ │ │ ┌─ day of week (0 - 6, Sunday = 0)
# │ │ │ │ │
# * * * * * command
```

**Examples:**
```bash
# Every minute
* * * * * /script.sh

# Every day at 2:30 AM
30 2 * * * /backup.sh

# Every Monday at 9 AM
0 9 * * 1 /weekly-report.sh

# Every 15 minutes
*/15 * * * * /check-health.sh

# First day of month at midnight
0 0 1 * * /monthly-cleanup.sh
```

**System cron directories:**
```bash
/etc/cron.d/        # Custom cron files
/etc/cron.daily/    # Daily scripts
/etc/cron.hourly/   # Hourly scripts
/etc/cron.weekly/   # Weekly scripts
/etc/cron.monthly/  # Monthly scripts
```

**Systemd Timers (modern alternative):**
```bash
# Create timer: /etc/systemd/system/backup.timer
[Unit]
Description=Daily backup

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
# Enable timer
systemctl enable backup.timer
systemctl start backup.timer
systemctl list-timers
```

**One-time tasks (at):**
```bash
# Run once at specific time
at 2:30 AM
> /path/to/script.sh
> Ctrl+D

# List scheduled jobs
atq

# Remove job
atrm 1
```

</details>

<details>
<summary><strong>Q19: Explain input/output redirection and pipes</strong></summary>

**Standard streams:**
- **stdin (0)**: Input stream
- **stdout (1)**: Output stream
- **stderr (2)**: Error stream

**Redirection:**
```bash
# Redirect stdout to file
command > file.txt          # Overwrite
command >> file.txt         # Append

# Redirect stderr
command 2> error.txt

# Redirect both
command > out.txt 2> err.txt
command > all.txt 2>&1      # Both to same file
command &> all.txt          # Shorthand (bash)

# Discard output
command > /dev/null
command 2>/dev/null         # Discard errors
command &>/dev/null         # Discard all

# Input from file
command < input.txt
```

**Pipes:**
```bash
# Send output of one command to another
cat file.txt | grep "error"
ps aux | grep nginx
df -h | grep /dev/sda

# Chain multiple commands
cat access.log | grep "404" | wc -l
ps aux | awk '{print $11}' | sort | uniq -c | sort -rn
```

**Special patterns:**
```bash
# Here document (multi-line input)
cat << EOF
Line 1
Line 2
EOF

# Process substitution
diff <(ls dir1) <(ls dir2)

# Tee (write to file AND stdout)
command | tee output.txt
command | tee -a output.txt     # Append

# xargs (convert stdin to arguments)
find . -name "*.log" | xargs rm
cat urls.txt | xargs -I {} curl {}
```

</details>

<details>
<summary><strong>Q20: How do you archive and compress files?</strong></summary>

**tar (Tape Archive):**
```bash
# Create archive
tar -cvf archive.tar directory/     # Create, verbose, file
tar -czvf archive.tar.gz directory/ # With gzip compression
tar -cjvf archive.tar.bz2 directory/ # With bzip2 compression

# Extract archive
tar -xvf archive.tar                # Extract
tar -xzvf archive.tar.gz            # Extract gzipped
tar -xjvf archive.tar.bz2           # Extract bzip2
tar -xvf archive.tar -C /destination/  # Extract to directory

# List contents
tar -tvf archive.tar
```

**Compression tools:**

| Tool | Extension | Speed | Compression |
|------|-----------|-------|-------------|
| gzip | .gz | Fast | Good |
| bzip2 | .bz2 | Slow | Better |
| xz | .xz | Slowest | Best |
| zip | .zip | Fast | Good (Windows compatible) |

```bash
# gzip
gzip file.txt                   # Compresses in place
gunzip file.txt.gz              # Decompresses
gzip -k file.txt                # Keep original

# zip
zip archive.zip file1 file2
zip -r archive.zip directory/
unzip archive.zip
unzip -l archive.zip            # List contents
```

**Common DevOps scenarios:**
```bash
# Backup with timestamp
tar -czvf backup-$(date +%Y%m%d).tar.gz /data/

# Compress logs older than 7 days
find /var/log -name "*.log" -mtime +7 -exec gzip {} \;

# Transfer compressed (over network)
tar czf - /data | ssh user@host "tar xzf - -C /backup"
```

</details>

