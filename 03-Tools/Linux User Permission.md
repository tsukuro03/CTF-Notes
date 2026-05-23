## User Management

### View Users

|Command|Description|
|---|---|
|`whoami`|Show current username|
|`id`|Show uid, gid, and all groups|
|`id username`|Show info of a specific user|
|`cat /etc/passwd`|List all users on the system|
|`getent passwd username`|Detailed info of a user|
|`last`|Login history|
|`lastlog`|Last login of every user|
|`w`|Who is logged in right now|

### Create & Delete Users

|Command|Description|
|---|---|
|`useradd username`|Create a new user (no home dir)|
|`useradd -m username`|Create user with home directory|
|`useradd -m -s /bin/bash username`|Create user with bash shell|
|`adduser username`|Interactive user creation (Debian)|
|`userdel username`|Delete user (keep home dir)|
|`userdel -r username`|Delete user and home directory|

### Modify Users

|Command|Description|
|---|---|
|`usermod -aG sudo username`|Add user to sudo group|
|`usermod -aG groupname username`|Add user to a group|
|`usermod -s /bin/bash username`|Change user shell|
|`usermod -l newname oldname`|Rename a user|
|`usermod -L username`|Lock user account|
|`usermod -U username`|Unlock user account|
|`passwd username`|Change user password|
|`passwd -l username`|Lock password (disable login)|
|`passwd -u username`|Unlock password|

---

## Group Management

### View Groups

|Command|Description|
|---|---|
|`groups`|Groups of current user|
|`groups username`|Groups of a specific user|
|`cat /etc/group`|List all groups|
|`getent group groupname`|Info of a specific group|

### Create & Delete Groups

|Command|Description|
|---|---|
|`groupadd groupname`|Create a new group|
|`groupdel groupname`|Delete a group|
|`groupmod -n newname oldname`|Rename a group|

---

## File Permissions

### Read Permissions

```bash
ls -l filename
# -rwxr-xr-- 1 alice users 1234 May 23 file.sh
#  ^^^           = owner permissions (rwx)
#     ^^^        = group permissions (r-x)
#        ^^^     = others permissions (r--)
```

### Permission Symbols

|Symbol|Meaning|
|---|---|
|`r`|Read|
|`w`|Write|
|`x`|Execute|
|`-`|No permission|
|`s`|Setuid / Setgid|
|`t`|Sticky bit|

### chmod — Change Permissions

#### Symbolic mode

|Command|Description|
|---|---|
|`chmod u+x file`|Add execute for owner|
|`chmod g-w file`|Remove write for group|
|`chmod o+r file`|Add read for others|
|`chmod a+x file`|Add execute for all|
|`chmod u+x,g-w file`|Multiple changes at once|
|`chmod -R 755 folder/`|Recursive permission change|

#### Octal mode

|Octal|Permissions|Meaning|
|---|---|---|
|`777`|rwxrwxrwx|Everyone full access|
|`755`|rwxr-xr-x|Owner full, others read+execute|
|`700`|rwx------|Owner only|
|`644`|rw-r--r--|Owner read+write, others read only|
|`600`|rw-------|Owner read+write only|
|`400`|r--------|Owner read only|

```bash
chmod 755 file.sh      # rwxr-xr-x
chmod 644 file.txt     # rw-r--r--
chmod 600 id_rsa       # rw------- (SSH key)
```

---

## Ownership

|Command|Description|
|---|---|
|`chown user file`|Change file owner|
|`chown user:group file`|Change owner and group|
|`chown -R user:group folder/`|Recursive ownership change|
|`chgrp groupname file`|Change group only|

---

## Sudo Management

### Check Sudo Rights

```bash
sudo -l                    # what can current user sudo
sudo -l -U username        # check another user (requires root)
cat /etc/sudoers           # full sudo config
```

### Grant Sudo Access

```bash
# Method 1 — add to sudo group
usermod -aG sudo username

# Method 2 — edit sudoers safely
visudo
# Add one of these lines:
username ALL=(ALL:ALL) ALL              # full sudo with password
username ALL=(ALL) NOPASSWD: ALL       # full sudo without password
username ALL=(ALL) NOPASSWD: /bin/bash # specific command only
```

### Sudoers Syntax

```
username  HOST=(RUNAS:GROUP)  COMMANDS
alice     ALL=(ALL:ALL)       ALL
bob       ALL=(ALL)           NOPASSWD: /bin/systemctl restart nginx
%sudo     ALL=(ALL:ALL)       ALL        # % means group
```

---

## Special Permissions

|Permission|Octal|Command|Effect|
|---|---|---|---|
|Setuid|`4xxx`|`chmod u+s file`|File runs as owner (not caller)|
|Setgid|`2xxx`|`chmod g+s file`|File runs as group|
|Sticky bit|`1xxx`|`chmod +t dir/`|Only owner can delete their files|

```bash
# Find setuid binaries (common privesc check)
find / -perm -4000 -type f 2>/dev/null

# Find setgid binaries
find / -perm -2000 -type f 2>/dev/null

# Find world-writable files
find / -perm -o+w -type f 2>/dev/null
```

---

## Key Files

|File|Description|
|---|---|
|`/etc/passwd`|User list (username, uid, gid, home, shell)|
|`/etc/shadow`|Password hashes (root only)|
|`/etc/group`|Group list and members|
|`/etc/sudoers`|Sudo privilege rules|
|`/etc/sudoers.d/`|Additional sudo config files|

### /etc/passwd format

```
username:x:uid:gid:comment:home:shell
ctf-player:x:1000:1000::/home/ctf-player:/bin/bash
```

### /etc/shadow format

```
username:hash:lastchange:min:max:warn:inactive:expire
```

---

## Privilege Escalation Checks (CTF)

```bash
# 1. Who am I?
whoami && id

# 2. What can I sudo?
sudo -l

# 3. Find SUID binaries
find / -perm -4000 -type f 2>/dev/null

# 4. Check writable files in sensitive locations
find /etc -writable -type f 2>/dev/null

# 5. Check cron jobs
cat /etc/crontab
ls -la /etc/cron.*

# 6. Check running processes
ps aux

# 7. Command history (may contain passwords)
cat ~/.bash_history
```

---

> 💡 **CTF Tip:** Always run `id` and `sudo -l` immediately after getting a shell — these two commands reveal your privileges and potential escalation paths.