# Linux Permissions


### User Account Management

#### 1.Creating a User Account — `useradd`

**What it does:** Creates a new user account on the system.

bash

```bash
useradd username
```

**Common flags:**

bash

```bash
useradd -m username        # Create home directory
useradd -m -s /bin/bash username   # Set default shell
useradd -m -G sudo username        # Add to sudo group on creation
useradd -m -c "Full Name" username # Add comment/full name
```

**Real behaviour:** Without `-m`, no home directory is created — the user exists but can't log in properly.

----------

#### 2.Setting a Password — `passwd`

**What it does:** Sets or changes a user's password.

bash

```bash
passwd username         # Set password for a user (as root)
passwd                  # Change your own password
passwd -l username      # Lock a user account
passwd -u username      # Unlock a user account
passwd -e username      # Force password change on next login
```

----------

#### 3.Modifying a User — `usermod`

**What it does:** Modifies an existing user account.

bash

```bash
usermod -aG sudo username        # Add user to sudo group
usermod -aG docker,nginx username # Add to multiple groups
usermod -s /bin/zsh username     # Change shell
usermod -l newname oldname       # Rename user
usermod -L username              # Lock account
usermod -U username              # Unlock account
```

⚠️ Always use `-aG` (append + group), never just `-G` — without `-a` it **replaces** all existing groups.

----------

#### 4.Deleting a User — `userdel`

**What it does:** Removes a user account from the system.

bash

```bash
userdel username          # Delete user only
userdel -r username       # Delete user + home directory + mail spool
```

----------

#### 5.Viewing Users & Groups

bash

```bash
cat /etc/passwd           # All users on the system
cat /etc/group            # All groups
id username               # User's UID, GID, and groups
groups username           # Groups a user belongs to
who                       # Currently logged-in users
last                      # Login history
```

----------

### 🗂️ Understanding File Permissions

Every file and directory has **three permission sets** and **three permission types:**

```
-rwxr-xr--  1  alice  developers  4096  May 8  file.sh
 ─┬─ ─┬─ ─┬─      │       │
  │   │   │       │       └── Group owner
  │   │   │       └────────── User owner
  │   │   └────────────────── Others permissions (r--)
  │   └────────────────────── Group permissions (r-x)
  └────────────────────────── Owner permissions (rwx)
```

Symbol	Meaning			File					Directory

`r`			read		View contents		List files

`w`			write		Edit file					Create/delete files

`x`			execute	Run as program		Enter directory (`cd`)

`-`			no permission						——

----------

#### 1.Numeric (Octal) Permission Values

Permission		Value

`r`					4

`w`					2

`x`					1

none				0

```
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
--- = 0+0+0 = 0
```

So `chmod 754` means:

-   Owner: `rwx` (7)
-   Group: `r-x` (5)
-   Others: `r--` (4)

----------

### 🔧 chmod — Change Permissions

**What it does:** Changes the read/write/execute permissions of a file or directory.

#### 1.Numeric mode

bash

```bash
chmod 755 script.sh      # rwxr-xr-x
chmod 644 file.txt       # rw-r--r--
chmod 700 secret.sh      # rwx------
chmod 777 file.sh        # rwxrwxrwx (dangerous — avoid)
chmod 000 file.txt       # no permissions for anyone
```

#### 2.Symbolic mode

bash

```bash
chmod u+x script.sh      # Add execute for owner
chmod g-w file.txt       # Remove write from group
chmod o+r file.txt       # Add read for others
chmod a+x script.sh      # Add execute for all (u+g+o)
chmod u+x,g-w file.txt   # Multiple changes at once
chmod go= file.txt       # Remove all permissions from group & others
```

#### 3.Recursive

bash

```bash
chmod -R 755 /var/www/   # Apply to directory and all contents
```

----------

### 🔧 chown — Change Ownership

**What it does:** Changes the user and/or group owner of a file or directory.

bash

```bash
chown alice file.txt              # Change owner to alice
chown alice:developers file.txt   # Change owner and group
chown :developers file.txt        # Change group only
chown -R alice:www-data /var/www  # Recursive ownership change
```

----------

### 🔧 chgrp — Change Group

**What it does:** Changes only the group ownership of a file.

bash

```bash
chgrp developers file.txt         # Change group to developers
chgrp -R www-data /var/www/html   # Recursive group change
```

----------

### 🔧 umask — Default Permission Mask

**What it does:** Sets the default permissions subtracted from new files and directories.

bash

```bash
umask           # View current umask (e.g. 0022)
umask 027       # Set new umask for session
```

**How it works:**

```
Files start at:       666 (rw-rw-rw-)
Directories start at: 777 (rwxrwxrwx)

umask 022 means subtract 022:
  Files:       666 - 022 = 644 (rw-r--r--)
  Directories: 777 - 022 = 755 (rwxr-xr-x)

umask 027:
  Files:       666 - 027 = 640 (rw-r-----)
  Directories: 777 - 027 = 750 (rwxr-x---)
```

To make persistent, add to `~/.bashrc` or `/etc/profile`.

----------

### 🔧 Special Permissions

#### 1.SUID (Set User ID) — `4xxx`

Runs the file as the **file's owner**, not the person executing it.

bash

```bash
chmod u+s /usr/bin/program
chmod 4755 /usr/bin/program
ls -l   # Shows: -rwsr-xr-x  (s in owner execute position)
```

Example: `/usr/bin/passwd` runs as root so it can edit `/etc/shadow`.

#### 2.SGID (Set Group ID) — `2xxx`

On files: runs as the file's group. On directories: new files inherit the directory's group.

bash

```bash
chmod g+s /shared/project/
chmod 2755 /shared/project/
ls -l   # Shows: drwxr-sr-x
```

#### 3.Sticky Bit — `1xxx`

On directories: only the file owner can delete their own files (used on `/tmp`).

bash

```bash
chmod +t /shared/tmp/
chmod 1777 /shared/tmp/
ls -l   # Shows: drwxrwxrwt  (t at the end)
```

----------

### 🧪 Scenario Tasks

Work through these in order — each builds on the last.

----------

####  Scenario 1 — Onboard a New Developer

> Your company hired **Alice**. She needs a Linux account, a home directory, bash shell, and must be added to the `developers` group. She must change her password on first login.

bash

```bash
# 1. Create the developers group if it doesn't exist
groupadd developers

# 2. Create Alice's account
useradd -m -s /bin/bash -G developers -c "Alice Smith" alice

# 3. Set a temporary password
passwd alice

# 4. Force password change on first login
passwd -e alice

# 5. Verify
id alice
groups alice
```

**Expected output of `id alice`:**

```
uid=1001(alice) gid=1001(alice) groups=1001(alice),1002(developers)
```

----------

####  Scenario 2 — Shared Project Directory

> The `developers` group needs a shared `/project` directory. All developers must read and write. Others should have no access. New files created inside must automatically belong to the `developers` group.

bash

```bash
# 1. Create the directory
mkdir /project

# 2. Set ownership
chown root:developers /project

# 3. Set permissions — rwx for owner, rwx for group, none for others
chmod 770 /project

# 4. Set SGID so new files inherit the group
chmod g+s /project

# 5. Verify
ls -ld /project
```

**Expected:**

```
drwxrws--- 2 root developers 4096 May 8 /project
```

----------

####  Scenario 3 — Deploy a Script Securely

> You have a deployment script `deploy.sh`. Only the owner (`devops`) should run it. The group should read it but not execute. Others get nothing.

bash

```bash
# 1. Set ownership
chown devops:developers deploy.sh

# 2. Set permissions: owner rwx, group r--, others ---
chmod 740 deploy.sh

# 3. Verify
ls -l deploy.sh
```

**Expected:**

```
-rwxr----- 1 devops developers 512 May 8 deploy.sh
```

----------

####  Scenario 4 — Shared Temp Directory

> Create a `/shared/tmp` directory where all users can create files but **no one can delete another user's files**.

bash

```bash
# 1. Create directory
mkdir -p /shared/tmp

# 2. Full permissions for everyone
chmod 777 /shared/tmp

# 3. Apply sticky bit
chmod +t /shared/tmp

# 4. Verify
ls -ld /shared/tmp
```

**Expected:**

```
drwxrwxrwt 2 root root 4096 May 8 /shared/tmp
```

----------

####  Scenario 5 — Lock a Leaving Employee

> **Bob** is leaving the company. Lock his account immediately without deleting it (HR may need his files).

bash

```bash
# 1. Lock the account
passwd -l bob

# 2. Verify (locked accounts show ! in /etc/shadow)
grep bob /etc/shadow

# 3. Optional: set expiry date to today
usermod --expiredate 1 bob

# 4. When ready to fully remove
userdel -r bob
```

----------

#### Scenario 6 — Harden a Config File

> A web app config file `config.env` contains database credentials. It should be readable and writable **only by the app user** (`appuser`). Nobody else should see it.

bash

```bash
# 1. Set ownership
chown appuser:appuser config.env

# 2. Owner read/write only — no permissions for group or others
chmod 600 config.env

# 3. Verify
ls -l config.env
```

**Expected:**

```
-rw------- 1 appuser appuser 256 May 8 config.env
```

----------

####  Scenario 7 — Audit Permissions on a Web Root

> Check `/var/www/html` — web files should be owned by `www-data`, readable by all, but only writable by the owner.

bash

```bash
# 1. Fix ownership recursively
chown -R www-data:www-data /var/www/html

# 2. Files: rw-r--r-- (644)
find /var/www/html -type f -exec chmod 644 {} \;

# 3. Directories: rwxr-xr-x (755)
find /var/www/html -type d -exec chmod 755 {} \;

# 4. Verify a sample
ls -la /var/www/html
```

----------

### 📋 Quick Reference Cheat Sheet

Task                                    Command

Create user with home                `useradd -m -s /bin/bash username`

Set password                         `passwd username`

Add to group                         `usermod -aG groupname username`

Delete user + home                   `userdel -r username`

Lock account                         `passwd -l username`

Change permissions (numeric)         `chmod 755 file`

Change permissions (symbolic)        `chmod u+x,g-w file`

Change owner                         `chown user:group file`

Recursive ownership                  `chown -R user:group /dir`

Set SGID on directory                `chmod g+s /dir`

Set sticky bit                       `chmod +t /dir`

View permissions                     `ls -la`

View user info                       `id username`
