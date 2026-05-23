## User & Authentication

|File|Description|
|---|---|
|`/etc/passwd`|User accounts list (username, uid, gid, home, shell)|
|`/etc/shadow`|Password hashes (readable by root only)|
|`/etc/group`|Group definitions and members|
|`/etc/gshadow`|Group password hashes|
|`/etc/sudoers`|Sudo privileges configuration|
|`/etc/sudoers.d/`|Additional sudo config files|
|`/etc/login.defs`|Default settings for user creation|
|`/etc/securetty`|Terminals allowed for root login|
|`/etc/shells`|List of valid login shells|

---

## System Configuration

|File|Description|
|---|---|
|`/etc/hostname`|System hostname|
|`/etc/hosts`|Static hostname-to-IP mappings|
|`/etc/resolv.conf`|DNS resolver configuration|
|`/etc/fstab`|Filesystem mount table|
|`/etc/os-release`|OS version and distribution info|
|`/etc/timezone`|System timezone|
|`/etc/localtime`|Timezone binary data|
|`/etc/environment`|System-wide environment variables|
|`/etc/profile`|System-wide shell environment (login)|
|`/etc/bashrc`|System-wide bash config (non-login)|

---

## Network Configuration

|File|Description|
|---|---|
|`/etc/hosts`|Static DNS entries|
|`/etc/resolv.conf`|DNS servers|
|`/etc/network/interfaces`|Network interface config (Debian)|
|`/etc/NetworkManager/`|NetworkManager config directory|
|`/etc/ssh/sshd_config`|SSH server configuration|
|`/etc/ssh/ssh_config`|SSH client configuration|
|`/etc/hosts.allow`|TCP Wrappers — allowed hosts|
|`/etc/hosts.deny`|TCP Wrappers — denied hosts|
|`/etc/iptables/rules.v4`|Persistent iptables rules (IPv4)|
|`/etc/iptables/rules.v6`|Persistent iptables rules (IPv6)|

---

## Services & Boot

|File|Description|
|---|---|
|`/etc/systemd/`|Systemd configuration directory|
|`/etc/init.d/`|SysVinit service scripts|
|`/etc/crontab`|System-wide cron jobs|
|`/etc/cron.d/`|Additional cron job files|
|`/etc/cron.daily/`|Daily cron scripts|
|`/var/spool/cron/crontabs/`|Per-user cron jobs|
|`/etc/rc.local`|Commands run at boot (legacy)|
|`/etc/modules`|Kernel modules to load at boot|

---

## Logs

|File|Description|
|---|---|
|`/var/log/syslog`|General system log (Debian/Ubuntu)|
|`/var/log/messages`|General system log (RHEL/CentOS)|
|`/var/log/auth.log`|Authentication attempts and sudo usage|
|`/var/log/secure`|Auth log (RHEL/CentOS)|
|`/var/log/kern.log`|Kernel messages|
|`/var/log/dmesg`|Boot and kernel ring buffer|
|`/var/log/cron.log`|Cron job execution log|
|`/var/log/dpkg.log`|Package installation log (Debian)|
|`/var/log/apache2/`|Apache web server logs|
|`/var/log/nginx/`|Nginx web server logs|
|`/var/log/wtmp`|Login history (read with `last`)|
|`/var/log/btmp`|Failed login attempts (read with `lastb`)|
|`/var/log/lastlog`|Last login per user (read with `lastlog`)|

---

## User Home Directory Files

|File|Description|
|---|---|
|`~/.bashrc`|User bash config (non-login shell)|
|`~/.bash_profile`|User bash config (login shell)|
|`~/.bash_history`|Command history|
|`~/.ssh/authorized_keys`|SSH public keys allowed to login|
|`~/.ssh/id_rsa`|SSH private key|
|`~/.ssh/known_hosts`|Known SSH host fingerprints|
|`~/.profile`|User environment variables|
|`~/.netrc`|FTP/HTTP credentials|
|`~/.gnupg/`|GPG keys directory|

---

## Sensitive Files (Penetration Testing)

|File|Why It Matters|
|---|---|
|`/etc/passwd`|User enumeration|
|`/etc/shadow`|Password hashes (crack offline)|
|`/etc/sudoers`|Privilege escalation paths|
|`~/.bash_history`|Commands run — may contain passwords|
|`~/.ssh/id_rsa`|Private SSH key — direct access|
|`~/.netrc`|Plaintext credentials|
|`/var/log/auth.log`|Login attempts, sudo usage|
|`/proc/version`|Kernel version — find exploits|
|`/proc/net/tcp`|Active network connections|
|`/etc/crontab`|Scheduled tasks — possible persistence|
|`/etc/hosts`|Internal network mapping|

---

## /proc — Process & Kernel Info

|File|Description|
|---|---|
|`/proc/version`|Kernel version|
|`/proc/cpuinfo`|CPU information|
|`/proc/meminfo`|Memory usage|
|`/proc/net/tcp`|Active TCP connections|
|`/proc/net/arp`|ARP table — local network hosts|
|`/proc/<pid>/cmdline`|Command line of a process|
|`/proc/<pid>/environ`|Environment variables of a process|
|`/proc/<pid>/maps`|Memory map of a process|

---

## Quick Read Commands

```bash
# User info
cat /etc/passwd
cat /etc/shadow          # root only
cat /etc/group
sudo -l                  # current user sudo rights

# Network
cat /etc/hosts
cat /etc/resolv.conf
cat /etc/ssh/sshd_config

# Logs
cat /var/log/auth.log
last                     # login history
lastb                    # failed logins

# System info
cat /proc/version
cat /etc/os-release
uname -a

# Sensitive
cat ~/.bash_history
cat ~/.ssh/authorized_keys
cat /etc/crontab
```

---

> 💡 **CTF Tip:** When you get a shell, quickly check `/etc/passwd`, `sudo -l`, `~/.bash_history`, and `/etc/crontab` — these four files often reveal the path to privilege escalation.