
## Processes & Services 

----------

### 🔍 What is a Process?

Every program running on Linux is a **process**. Each process gets a unique **PID (Process ID)**. When a process spawns another, the original is the **parent** and the new one is the **child**.

```
systemd (PID 1)
  └── sshd (PID 412)
        └── bash (PID 1023)
              └── ps (PID 1024)
```

Attribute&emsp;&emsp; Description

**PID**&emsp;&emsp;&emsp;&emsp;Unique process ID

**PPID**&emsp;&emsp;&emsp;Parent process ID

**UID**&emsp;&emsp;&emsp;&emsp;User who owns the process

**State**&emsp;&emsp;&emsp;Running, sleeping, stopped, zombie

**Nice**&emsp;&emsp;&emsp;Priority level (-20 to +19)

----------

### 📊 Viewing Processes

#### `ps` — Process Snapshot

**What it does:** Shows a snapshot of currently running processes.

bash

```bash
ps                    # Your processes in current shell
ps aux                # All processes, all users (most used)
ps -ef                # Full format listing
ps aux | grep nginx   # Find a specific process
ps -u alice           # Processes owned by alice
ps --forest           # Show parent-child tree
```

**Understanding `ps aux` output:**

```
USER    PID  %CPU %MEM    VSZ   RSS TTY   STAT  START   TIME COMMAND
root      1   0.0  0.1  22560  1234 ?     Ss    09:00   0:01 /sbin/init
alice  1023   2.1  0.5  15320  4321 pts/0 S     09:05   0:00 bash
```

Column&emsp;Meaning

`USER`&emsp;Owner of the process

`PID`&emsp; Process ID

`%CPU`&emsp;CPU usage percentage

`%MEM`&emsp;Memory usage percentage

`STAT`&emsp;Process state (S=sleeping, R=running, Z=zombie, D=disk wait)

`COMMAND`&emsp;The command that started it

----------

#### `top` — Live Process Monitor

**What it does:** Real-time, auto-refreshing view of processes and system resource usage.

bash

```bash
top               # Launch top
top -u alice      # Show only alice's processes
top -p 1234       # Watch a specific PID
```

**Inside top — keyboard shortcuts:**

Key&emsp;Action

`k`&emsp;	Kill a process (enter PID)

`r`&emsp;	Renice (change priority)

`M`&emsp;	Sort by memory usage

`P`&emsp;	Sort by CPU usage

`u`&emsp;	Filter by user

`q`&emsp;	Quit

`1`&emsp;	Toggle per-CPU view

----------

#### `htop` — Enhanced Live Monitor

**What it does:** A friendlier, colour-coded version of `top` with mouse support.

bash

```bash
htop              # Launch htop
htop -u alice     # Filter by user
```

**Install if not present:**

bash

```bash
apt install htop   # Debian/Ubuntu
yum install htop   # RHEL/CentOS
```

**Inside htop — useful shortcuts:**

Key&emsp;	Action

`F3`&emsp;	Search for a process

`F5`&emsp;	Tree view (parent-child)

`F6`&emsp;	Sort by column

`F9`&emsp;	Kill a process

`F10`&emsp;Quit

----------

#### `pgrep` & `pidof` — Find a PID

**What it does:** Finds the PID of a process by name.

bash

```bash
pgrep nginx           # Returns PID(s) of nginx
pgrep -u alice bash   # Find bash processes owned by alice
pgrep -l nginx        # Show name alongside PID
pidof nginx           # Alternative — same purpose
```

----------

### 📡 Signals & Killing Processes

#### Understanding Signals

Signals are messages sent to processes to trigger specific behaviour.

Signal	&emsp;Number	&emsp;Meaning

`SIGTERM`&emsp;15&emsp;	&emsp;Politely ask process to stop (default)

`SIGKILL`&emsp;9&emsp;&emsp;&emsp;Force kill — cannot be ignored

`SIGHUP`&emsp;	1&emsp;&emsp;&emsp;	Reload config (hangup)

`SIGSTOP`&emsp;19&emsp;&emsp;Pause a process

`SIGCONT`&emsp;18&emsp;&emsp;Resume a paused process

**Rule of thumb:** Always try `SIGTERM` first. Use `SIGKILL` only when the process refuses to stop — it skips cleanup and can cause data corruption.

----------

#### `kill` — Send Signal by PID

bash

```bash
kill 1234             # Send SIGTERM (15) to PID 1234
kill -9 1234          # Force kill PID 1234
kill -15 1234         # Explicit SIGTERM
kill -1 1234          # SIGHUP — reload config
kill -SIGTERM 1234    # Same as kill -15
```

----------

#### `pkill` — Send Signal by Name

bash

```bash
pkill nginx           # Kill all nginx processes by name
pkill -9 nginx        # Force kill by name
pkill -u alice        # Kill all processes owned by alice
pkill -SIGHUP nginx   # Reload nginx config
```

----------

#### `killall` — Kill All by Name

bash

```bash
killall nginx         # Kill all processes named nginx
killall -9 nginx      # Force kill all nginx
killall -u alice      # Kill all of alice's processes
```

----------

### ⚖️ Process Priority — `nice` & `renice`

**What it does:** Controls how much CPU time a process gets. Lower nice = higher priority.

```
Nice value range: -20 (highest priority) to +19 (lowest priority)
Default nice value: 0
Only root can set negative values
```

#### `nice` — Start a process with a priority

bash

```bash
nice -n 10 ./backup.sh        # Start with low priority (+10)
nice -n -5 ./critical.sh      # Start with high priority (-5, root only)
nice -n 19 ./batch-job.sh     # Lowest possible priority
```

#### `renice` — Change priority of running process

bash

```bash
renice -n 10 -p 1234          # Lower priority of PID 1234
renice -n -5 -p 1234          # Increase priority (root only)
renice -n 15 -u alice         # Lower priority of all alice's processes
```

----------

### 🔄 Foreground & Background Jobs

#### Running in Background

bash

```bash
./backup.sh &             # Start process in background
sleep 100 &               # Background sleep
nohup ./script.sh &       # Run in background, survives terminal close
```

#### Managing Jobs

bash

```bash
jobs                      # List background jobs in current shell
jobs -l                   # With PIDs
fg                        # Bring last background job to foreground
fg %1                     # Bring job 1 to foreground
bg                        # Resume stopped job in background
bg %2                     # Resume job 2 in background
```

#### Ctrl shortcuts

Shortcut&emsp;&emsp;&emsp;Action

`Ctrl+C`&emsp;&emsp;&emsp;Terminate foreground process (SIGTERM)

`Ctrl+Z`&emsp;&emsp;&emsp;Pause/suspend foreground process

`Ctrl+\`&emsp;&emsp;&emsp;Force quit (SIGQUIT)

----------

### 🕐 Scheduling — `cron` & `at`

#### `cron` — Recurring Scheduled Jobs

**What it does:** Runs commands on a repeating schedule.

bash

```bash
crontab -e        # Edit your cron jobs
crontab -l        # List your cron jobs
crontab -r        # Remove all your cron jobs
crontab -u alice -l  # View alice's crontab (root only)
```

**Cron syntax:**

```
┌─────────── minute       (0–59)
│ ┌───────── hour         (0–23)
│ │ ┌─────── day of month (1–31)
│ │ │ ┌───── month        (1–12)
│ │ │ │ ┌─── day of week  (0–7, 0&7=Sunday)
│ │ │ │ │
* * * * *  command to run
```

**Examples:**

bash

```bash
# Every minute
* * * * * /scripts/check.sh

# Every day at 2:30 AM
30 2 * * * /scripts/backup.sh

# Every Monday at 9 AM
0 9 * * 1 /scripts/weekly-report.sh

# Every 5 minutes
*/5 * * * * /scripts/monitor.sh

# First day of every month at midnight
0 0 1 * * /scripts/monthly-cleanup.sh

# Weekdays only at 6 PM
0 18 * * 1-5 /scripts/end-of-day.sh
```

**System-wide cron (root):**

bash

```bash
cat /etc/crontab              # System crontab
ls /etc/cron.d/               # Drop-in cron files
ls /etc/cron.daily/           # Scripts run daily
ls /etc/cron.weekly/          # Scripts run weekly
```

----------

#### `at` — One-time Scheduled Job

**What it does:** Runs a command once at a specific time.

bash

```bash
at 10:30                      # Schedule at 10:30 AM today
at 10:30 tomorrow             # Tomorrow at 10:30
at now + 2 hours              # 2 hours from now
at 9:00 AM May 15             # Specific date

# Inside the at prompt, type your commands then Ctrl+D
at> /scripts/deploy.sh
at> <EOT>

atq                           # List pending at jobs
atrm 3                        # Remove job number 3
```

----------

### ⚙️ systemd & Services

#### What is systemd?

**systemd** is the init system (PID 1) on modern Linux. It manages services, mounts, networking, and the boot process. Services are defined in **unit files**.

bash

```bash
systemctl --version           # Check systemd version
systemctl list-units          # All active units
systemctl list-unit-files     # All installed unit files
```

----------

#### `systemctl` — Control Services

**What it does:** Start, stop, restart, enable, disable and inspect services.

bash

```bash
# Start / Stop / Restart
systemctl start nginx         # Start nginx now
systemctl stop nginx          # Stop nginx now
systemctl restart nginx       # Stop then start
systemctl reload nginx        # Reload config without stopping

# Enable / Disable (survive reboot)
systemctl enable nginx        # Start on boot
systemctl disable nginx       # Don't start on boot
systemctl enable --now nginx  # Enable AND start immediately

# Status & Inspection
systemctl status nginx        # Detailed status + recent logs
systemctl is-active nginx     # Returns "active" or "inactive"
systemctl is-enabled nginx    # Returns "enabled" or "disabled"
systemctl is-failed nginx     # Returns "failed" if crashed

# View all failed services
systemctl --failed
```

----------

#### `journalctl` — View Service Logs

**What it does:** Reads logs from the systemd journal.

bash

```bash
journalctl                        # All logs (oldest first)
journalctl -r                     # Reverse (newest first)
journalctl -u nginx               # Logs for nginx only
journalctl -u nginx -f            # Follow (live tail) nginx logs
journalctl -u nginx --since today # Today's nginx logs
journalctl -u nginx -n 50         # Last 50 lines
journalctl -p err                 # Error-level logs only
journalctl --since "2024-05-01" --until "2024-05-08"
journalctl -b                     # Logs since last boot
journalctl -b -1                  # Logs from previous boot
```

----------

#### Writing a Custom systemd Service

**Unit file location:** `/etc/systemd/system/myapp.service`

ini

```ini
[Unit]
Description=My Python App
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/app.py
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

**After creating the file:**

bash

```bash
systemctl daemon-reload           # Reload systemd to pick up new file
systemctl enable --now myapp      # Enable and start
systemctl status myapp            # Verify it's running
journalctl -u myapp -f            # Watch its logs live
```

----------

### 🧪 Scenario Tasks

----------

#### ✅ Scenario 1 — Find a Resource-Hungry Process

> Your server is running slow. Identify which process is consuming the most CPU and memory, then gracefully stop it.

bash

```bash
# 1. Check top consumers
top
# Press M to sort by memory, P to sort by CPU

# 2. Identify the PID from output
ps aux --sort=-%cpu | head -10   # Top 10 CPU consumers
ps aux --sort=-%mem | head -10   # Top 10 memory consumers

# 3. Gracefully stop it
kill -15 <PID>

# 4. If it doesn't respond, force kill
kill -9 <PID>

# 5. Confirm it's gone
ps aux | grep <PID>
```

----------

#### ✅ Scenario 2 — Run a Heavy Backup Without Affecting Users

> You need to run `backup.sh` during business hours but it shouldn't slow down other processes.

bash

```bash
# 1. Run with lowest priority so it doesn't compete
nice -n 19 ./backup.sh &

# 2. Check its nice value
ps -o pid,ni,comm -p $(pgrep backup)

# 3. If already running, renice it
renice -n 19 -p $(pgrep backup)

# 4. Monitor it without blocking terminal
jobs -l
```

----------

#### ✅ Scenario 3 — Schedule Automated Backups

> Set up a cron job to run `/scripts/backup.sh` every day at 2 AM and log its output.

bash

```bash
# 1. Open crontab
crontab -e

# 2. Add the job
0 2 * * * /scripts/backup.sh >> /var/log/backup.log 2>&1

# 3. Verify it was saved
crontab -l

# 4. Check log after it runs
tail -f /var/log/backup.log
```

**What `2>&1` means:** Redirects stderr (2) to stdout (1) so both are captured in the log.

----------

#### ✅ Scenario 4 — Install and Manage Nginx as a Service

> Install nginx, make sure it starts on boot, verify it's running, and check its logs.

bash

```bash
# 1. Install nginx
apt install nginx -y

# 2. Start and enable on boot
systemctl enable --now nginx

# 3. Check status
systemctl status nginx

# 4. Test it's serving
curl http://localhost

# 5. View logs
journalctl -u nginx -n 30

# 6. Reload config after changes (no downtime)
systemctl reload nginx
```

----------

#### ✅ Scenario 5 — Deploy a Python App as a systemd Service

> Your team built a Python app at `/opt/myapp/app.py`. It should run as `appuser`, restart automatically if it crashes, and start on boot.

bash

```bash
# 1. Create the user
useradd -r -s /bin/false appuser    # -r = system user, no shell needed

# 2. Create the unit file
nano /etc/systemd/system/myapp.service
```

ini

```ini
[Unit]
Description=My Python App
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/app.py
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

bash

```bash
# 3. Reload and start
systemctl daemon-reload
systemctl enable --now myapp

# 4. Verify
systemctl status myapp
journalctl -u myapp -f
```

----------

#### ✅ Scenario 6 — Investigate a Crashed Service

> A colleague says nginx went down overnight. Find out what happened and when.

bash

```bash
# 1. Check current status
systemctl status nginx

# 2. Check logs from last boot
journalctl -u nginx -b

# 3. Check previous boot logs (if it crashed badly enough to reboot)
journalctl -u nginx -b -1

# 4. Filter for errors only
journalctl -u nginx -p err

# 5. Check around a specific time
journalctl -u nginx --since "2024-05-08 02:00" --until "2024-05-08 03:00"

# 6. Restart and watch live
systemctl start nginx
journalctl -u nginx -f
```

----------

#### ✅ Scenario 7 — Kill a Zombie Process

> You spot a zombie process (state `Z` in `ps`). Handle it properly.

bash

```bash
# 1. Find zombie processes
ps aux | grep Z

# 2. Find its parent PID
ps -o ppid= -p <zombie_PID>

# 3. Send SIGCHLD to parent to reap the zombie
kill -SIGCHLD <parent_PID>

# 4. If parent is unresponsive, kill the parent
kill -9 <parent_PID>

# Note: you cannot kill a zombie directly —
# zombies are already dead, waiting for parent to collect exit status
```

----------

### 📋 Quick Reference Cheat Sheet

Task 				&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;			Command

List all processes		`ps aux`

Live process monitor `top` / `htop`

Find PID by name	`pgrep nginx`

Kill by PID (graceful)	`kill -15 <PID>`

Kill by PID (force)	`kill -9 <PID>`

Kill by name	`pkill nginx`

Run in background	`./script.sh &`

List background jobs	`jobs -l`

Bring to foreground	`fg %1`

Start with low priority	`nice -n 19 ./script.sh`

Change running priority	`renice -n 10 -p <PID>`

Edit cron jobs	`crontab -e`

Schedule one-time job	`at now + 1 hour`

Start a service	`systemctl start nginx`

Enable on boot	`systemctl enable nginx`

Check service status	`systemctl status nginx`

View service logs live	`journalctl -u nginx -f`

Reload systemd	`systemctl daemon-reload`

----------

