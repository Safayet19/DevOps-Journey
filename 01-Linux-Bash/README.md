<h1 align="center">Linux Commands & Shell Scripting</h1>

<p align="center">
  A practical Linux reference for everyday commands, system administration, Bash scripting, and troubleshooting.
</p>

This guide covers common tasks with **complete commands**, concise explanations, and reusable scripts. Commands are intended for a Linux terminal (for example, Ubuntu, Debian, or Fedora); Linux-specific tools such as `systemctl` and `apt-get` may not work in Windows Git Bash.

> **Conventions:** `~` means your home directory. Names such as `demo`, `app.log`, and `project/` are examples; replace them with your actual users and paths. Commands beginning with `sudo` require administrative permission. Check paths before running delete, overwrite, or ownership-changing commands.

## Table of Contents

- [Navigation and Directories](#navigation-and-directories)
- [File Management](#file-management)
- [Viewing and Processing Text](#viewing-and-processing-text)
- [Users and Groups](#users-and-groups)
- [Permissions and Ownership](#permissions-and-ownership)
- [Processes and Jobs](#processes-and-jobs)
- [Services and Logs](#services-and-logs)
- [Networking and DNS](#networking-and-dns)
- [SSH and File Transfer](#ssh-and-file-transfer)
- [Disk, Memory and Storage](#disk-memory-and-storage)
- [System Information](#system-information)
- [Package Management](#package-management)
- [Archives and Compression](#archives-and-compression)
- [Shell Environment and Editors](#shell-environment-and-editors)
- [Bash Scripting](#bash-scripting)
- [Scheduling and Automation](#scheduling-and-automation)

---

## Navigation and Directories

| Task | Command | Explanation |
|---|---|---|
| Show current directory | `pwd` | Print the absolute path of your current directory. |
| List files and folders | `ls` | Show visible entries in the current directory. |
| Include hidden files and details | `ls -la` | `-l` shows details; `-a` includes names beginning with `.`. |
| Show readable file sizes | `ls -lh /var/log` | `-h` displays sizes such as KB/MB, used with `-l`. |
| Change directory | `cd /var/log` | Move into `/var/log`. |
| Return to home | `cd ~` | `~` refers to the current user's home directory. |
| Go up one directory | `cd ..` | Move to the parent directory. |
| Go to previous directory | `cd -` | Return to the last working directory. |
| Create nested directories | `mkdir -p ~/project/logs` | `-p` creates missing parent directories and avoids errors if they exist. |
| Remove an empty directory | `rmdir ~/empty-folder` | Only succeeds when the directory is empty. |

## File Management

| Task | Command | Explanation |
|---|---|---|
| Create a file | `touch notes.txt` | Creates an empty file or updates its modification time. |
| Copy one file | `cp notes.txt notes-backup.txt` | Keeps the original and makes another copy. |
| Copy a directory | `cp -r project/ project-backup/` | `-r` recursively copies its contents. |
| Preserve file timestamps | `cp -p notes.txt saved-notes.txt` | `-p` preserves supported timestamps, mode, and ownership where allowed. |
| Move or rename | `mv notes.txt notes-old.txt` | Moves a file or changes its name. |
| Remove one file | `rm notes-old.txt` | Deletes the named file; not moved to the Trash. |
| Remove a directory tree | `rm -r project-backup/` | `-r` recursively deletes; verify the directory first. |
| Find files by name | `find ~ -type f -name '*.log'` | `-type f` selects files; `-name` matches the pattern. |
| Find directories | `find ~ -type d -name 'project'` | `-type d` selects directories. |
| Find recently changed files | `find . -type f -mtime -1` | Finds files modified within roughly the last 24 hours. |
| Search using file index | `locate '*.conf'` | Fast filename search if `locate`/`plocate` is installed and indexed. |
| Identify file type | `file notes.txt` | Detects the file's content type. |
| View file metadata | `stat notes.txt` | Shows size, timestamps, permissions, and ownership. |
| Create a symbolic link | `ln -s notes.txt notes-link.txt` | Creates a link pointing to `notes.txt`. |

> **Deletion safety:** `rm -rf dir/` combines recursive deletion (`-r`) with force (`-f`), removing files without most prompts. Avoid using it until you have checked the exact path.

## Viewing and Processing Text

| Task | Command | Explanation |
|---|---|---|
| Display a file | `cat /etc/os-release` | Print the file contents. |
| Write a file interactively | `cat > notes.txt` | `>` overwrites the file; type text, then press **Ctrl+D**. |
| Append a line | `echo 'Hello, Linux' >> notes.txt` | `>>` adds text without replacing existing content. |
| Read a long file | `less /etc/services` | Scroll with arrows; press `q` to exit. |
| Show first lines | `head -n 10 /etc/passwd` | `-n 10` means the first 10 lines. |
| Show last lines | `tail -n 20 app.log` | `-n 20` means the last 20 lines. |
| Follow a changing log | `tail -f app.log` | `-f` streams newly appended lines; **Ctrl+C** stops it. |
| Search without case sensitivity | `grep -i 'error' app.log` | `-i` matches `error`, `ERROR`, etc. |
| Search through directories | `grep -rn 'TODO' ./project` | `-r` searches recursively; `-n` prints line numbers. |
| Exclude matching lines | `grep -v '^#' /etc/ssh/sshd_config` | `-v` shows lines that do **not** match the pattern. |
| Extract colon-separated field | `cut -d: -f1 /etc/passwd` | `-d:` sets the delimiter; `-f1` selects the first field. |
| Sort lines | `sort names.txt` | Outputs lines in ascending order. |
| Remove repeated lines | `sort names.txt \| uniq` | `uniq` removes adjacent duplicates; `sort` groups them first. |
| Count repeated values | `sort names.txt \| uniq -c \| sort -nr` | `-c` counts; `-nr` sorts largest count first. |
| Count lines | `wc -l app.log` | `-l` counts newline-terminated lines. |
| Extract a structured field | `awk -F: '{print $1}' /etc/passwd` | `-F:` separates fields on `:`; `$1` is the first. |
| Replace text in output | `sed 's/error/ERROR/g' app.log` | Replaces matches in displayed output; the file is unchanged. |
| Print selected lines | `sed -n '1,5p' /etc/passwd` | `-n` suppresses default output; `p` prints lines 1–5. |
| Print text or variables | `echo "Home: $HOME"` | Prints a string and expands `$HOME`. |

**Redirection and pipes:** `>` overwrites, `>>` appends, `2>` redirects error output, and `|` sends one command's output to another. The `\|` characters used inside the table are Markdown-escaped pipes; type a normal `|` in your terminal.

## Users and Groups

| Task | Command | Explanation |
|---|---|---|
| Current username | `whoami` | Show the effective current user. |
| User and group IDs | `id` | Show UID, primary GID, and groups. |
| Group memberships | `groups` | List your groups. |
| Change your password | `passwd` | Prompts for the existing and new passwords as required. |
| View sudo permissions | `sudo -l` | List commands you are allowed to run with `sudo`. |
| Run a command as root | `sudo whoami` | Should print `root` if authorized. |
| Create a user with home | `sudo useradd -m demo` | `-m` creates a home directory for user `demo`. |
| Set a user's password | `sudo passwd demo` | Prompts to set a password for `demo`. |
| Create a group | `sudo groupadd developers` | Adds a group named `developers`. |
| Add user to a group | `sudo usermod -aG developers demo` | `-aG` adds a supplementary group without dropping others. |
| Switch user | `su - demo` | Starts a login shell as `demo`, subject to authentication. |
| Remove a test user | `sudo userdel -r demo` | `-r` also removes the user's home/mail data; use carefully. |

## Permissions and Ownership

| Task | Command | Explanation |
|---|---|---|
| See permissions | `ls -l notes.txt` | Shows owner, group, and mode such as `-rw-r--r--`. |
| Make script executable | `chmod +x script.sh` | Adds execute permission as allowed by the current mode. |
| Set common file permissions | `chmod 644 notes.txt` | Owner read/write; group and others read-only. |
| Limit access to a script | `chmod 750 script.sh` | Owner `rwx`, group `r-x`, others no access. |
| Change directory tree modes | `chmod -R u+rwX ./project` | `-R` recursive; `X` adds execute only to directories and already-executable files. |
| Change owner and group | `sudo chown demo:developers notes.txt` | Assigns both owner and group; they must exist. |
| Change ownership recursively | `sudo chown -R "$USER:$(id -gn)" ./project` | `-R` changes the directory and its contents. |
| Change only the group | `sudo chgrp developers notes.txt` | Keeps the owner unchanged. |
| View the default creation mask | `umask` | Shows permissions masked for newly created files/directories. |

**Reading permissions:** `r` = read (4), `w` = write (2), `x` = execute (1). For example, `755` means `rwxr-xr-x`. For directories, `x` controls traversal.

## Processes and Jobs

| Task | Command | Explanation |
|---|---|---|
| List processes | `ps aux` | Common BSD-style listing of all processes with details. |
| Display process hierarchy | `ps -ef` | Lists processes with parent PIDs in standard format. |
| Watch resource usage | `top` | Interactive process monitor; press `q` to exit. |
| Find a process by name | `pgrep -a sshd` | Finds matching processes and prints their command lines. |
| Send termination signal | `kill 1234` | Sends `SIGTERM` to PID `1234` (replace with a real PID). |
| Force-stop a process | `kill -9 1234` | Sends `SIGKILL`; use only when graceful stop fails. |
| Terminate by process name | `pkill -f 'demo-worker'` | Matches the command line; inspect matches before using. |
| Start a background job | `sleep 60 &` | `&` runs the command as a shell background job. |
| List current shell jobs | `jobs` | Shows jobs controlled by this shell. |
| Bring job to foreground | `fg %1` | Moves job number 1 to the foreground. |
| Resume a stopped job | `bg %1` | Resumes job number 1 in the background. |
| Keep process after logout | `nohup bash script.sh > script.log 2>&1 &` | Ignore hangup, save output, and run in background. |

**Tip:** Try `kill PID` before `kill -9 PID`. Replace example PIDs and process names with real ones discovered via `ps` or `pgrep`.

## Services and Logs

These commands require a Linux system running **systemd**. The SSH service is usually `ssh` on Ubuntu/Debian and `sshd` on Fedora/RHEL.

| Task | Command | Explanation |
|---|---|---|
| Check service status | `systemctl status ssh` | Shows state, recent logs, and process information. |
| Start a service | `sudo systemctl start ssh` | Starts it now, without enabling boot startup. |
| Stop a service | `sudo systemctl stop ssh` | Stops it now; can disconnect remote SSH users. |
| Restart a service | `sudo systemctl restart ssh` | Stops and starts the service. |
| Enable at boot | `sudo systemctl enable ssh` | Enables startup after reboot. |
| Disable boot startup | `sudo systemctl disable ssh` | Does not stop a service currently running. |
| List running services | `systemctl list-units --type=service --state=running` | Shows currently active services. |
| Show recent service logs | `journalctl -u ssh -n 30 --no-pager` | `-u` filters service; `-n 30` shows the last 30 entries. |
| Follow service logs | `journalctl -u ssh -f` | `-f` follows new entries. |
| Show recent system logs | `journalctl -f` | Follows all recent systemd journal entries. |
| Check current-boot logs | `journalctl -b -p warning --no-pager` | `-b` current boot, `-p` warning or more severe. |

## Networking and DNS

| Task | Command | Explanation |
|---|---|---|
| View IP addresses | `ip addr show` | Displays network interfaces and addresses. |
| View routes | `ip route` | Shows the kernel routing table. |
| Check host connectivity | `ping -c 4 1.1.1.1` | `-c 4` sends four ICMP requests. |
| Check listening sockets | `ss -tuln` | TCP (`t`), UDP (`u`), listening (`l`), numeric (`n`). |
| Identify listening processes | `sudo ss -tulpn` | Adds process information (`p`), when permitted. |
| Fetch HTTP response headers | `curl -I https://example.com` | `-I` requests headers only. |
| Specify an HTTP method | `curl -X GET https://example.com` | `-X GET` explicitly chooses the request method. |
| Print HTTP status code | `curl -sS -o /dev/null -w '%{http_code}\n' https://example.com` | `-o` discards body; `-w` prints the code. |
| Download a file | `wget https://example.com/` | Downloads the example website homepage as a local file. |
| Query DNS records | `dig example.com` | Shows DNS query and answer details. |
| Show concise DNS answer | `dig +short example.com` | `+short` displays just answer records. |
| Alternative DNS lookup | `nslookup example.com` | Resolves DNS without the full `dig` output. |
| Show network hops | `traceroute example.com` | Traces a route, if the tool is installed and packets are allowed. |

## SSH and File Transfer

Examples use `192.0.2.10`, a **documentation-only IP address**: substitute the actual server address and username before connecting.

| Task | Command | Explanation |
|---|---|---|
| Connect to server | `ssh admin@192.0.2.10` | Remote shell as `admin`. |
| Use a different port | `ssh -p 2222 admin@192.0.2.10` | `-p` chooses SSH port 2222. |
| Use an SSH private key | `ssh -i ~/.ssh/id_ed25519 admin@192.0.2.10` | `-i` selects an identity file. |
| Create a new SSH key | `ssh-keygen -t ed25519 -C 'laptop-key'` | Creates a modern key pair; prompts for path/passphrase. |
| Install a public key | `ssh-copy-id admin@192.0.2.10` | Adds your public key to a remote account, when supported. |
| Copy a file to server | `scp notes.txt admin@192.0.2.10:~/` | Uploads one file over SSH. |
| Copy a directory | `scp -r project/ admin@192.0.2.10:~/` | `-r` transfers all nested content. |
| Preview a synchronized copy | `rsync -av --dry-run project/ ./project-backup/` | `-a` preserves structure/metadata; `-v` verbose; dry run changes nothing. |
| Synchronize directories | `rsync -av project/ ./project-backup/` | Copies changed content; trailing slash copies contents of `project/`. |

**SSH security:** Never commit private keys (such as `~/.ssh/id_ed25519`) to GitHub. Confirm the host key fingerprint when connecting for the first time.

## Disk, Memory and Storage

| Task | Command | Explanation |
|---|---|---|
| Check disk space | `df -h` | Filesystem size, usage, and free space in readable units. |
| Check inode usage | `df -i` | Detects exhausted file counts even with free disk space. |
| Size of a directory | `du -sh /var/log` | `-s` totals the directory; `-h` uses readable sizes. |
| Show memory and swap | `free -h` | Displays memory and swap in readable units. |
| List storage devices | `lsblk` | Shows block devices, sizes, and mount points. |
| Identify filesystems | `lsblk -f` | Includes filesystem types and UUIDs. |
| Display mounted filesystem | `findmnt /` | Shows the device backing the root filesystem. |
| Check swap devices | `swapon --show` | Lists configured active swap areas. |
| Unmount a mount point | `sudo umount /mnt/backup` | Unmounts an existing mount; requires permissions and no busy processes. |

**Storage warning:** Mounting, unmounting, partitioning, and filesystem formatting affect real disks. Identify devices with `lsblk -f` first; never guess a `/dev/...` path.

## System Information

| Task | Command | Explanation |
|---|---|---|
| Kernel and architecture | `uname -a` | Show kernel release and platform details. |
| Distribution details | `cat /etc/os-release` | Show the Linux distribution and version. |
| Hostname and OS info | `hostnamectl` | Show host information on systemd-based Linux. |
| System uptime/load | `uptime` | Show boot duration and load averages. |
| Current local date | `date` | Show system-local time and date. |
| Current UTC time | `date -u` | `-u` displays UTC. |
| All environment variables | `env` | Print environment variables for the current process. |
| A specific environment value | `printenv HOME` | Print `$HOME`, if defined. |
| CPU details | `lscpu` | Show CPU architecture and capabilities. |

## Package Management

Use the package manager that belongs to your Linux distribution. The two families below are **alternatives**, not commands to use together.

| Task | Command | Explanation |
|---|---|---|
| Refresh package metadata (Debian/Ubuntu) | `sudo apt-get update` | Downloads the latest package indexes. |
| Install a package (Debian/Ubuntu) | `sudo apt-get install -y nginx` | Installs Nginx; `-y` answers yes to prompts. |
| Upgrade installed packages (Debian/Ubuntu) | `sudo apt-get upgrade` | Upgrades packages according to current indexes. |
| Remove a package (Debian/Ubuntu) | `sudo apt-get remove nginx` | Removes the package, normally retaining configuration. |
| List installed packages (Debian/Ubuntu) | `dpkg -l` | Shows installed/package-status information. |
| Install a package (Fedora/RHEL) | `sudo dnf install nginx` | Uses the DNF package manager. |
| Upgrade packages (Fedora/RHEL) | `sudo dnf upgrade` | Updates installed packages. |
| Remove a package (Fedora/RHEL) | `sudo dnf remove nginx` | Uninstalls Nginx. |

## Archives and Compression

| Task | Command | Explanation |
|---|---|---|
| Create a compressed tarball | `tar -czvf project.tar.gz project/` | `c` create, `z` gzip, `v` verbose, `f` output filename. |
| Inspect compressed tarball | `tar -tzf project.tar.gz` | `t` lists archive contents without extracting. |
| Extract compressed tarball | `tar -xzvf project.tar.gz` | `x` extracts, `z` decompresses gzip, `v` lists members. |
| Create an uncompressed tar archive | `tar -cvf project.tar project/` | Creates a `.tar` archive without gzip. |
| Extract a plain tar archive | `tar -xvf project.tar` | Extracts members of an uncompressed tar. |
| Create ZIP of files | `zip files.zip notes.txt script.sh` | Stores the named files in a ZIP archive. |
| ZIP a directory | `zip -r project.zip project/` | `-r` includes nested files and folders. |
| Extract ZIP | `unzip project.zip` | Extracts the ZIP into the current directory. |
| Compress one file | `gzip app.log` | Produces `app.log.gz`, replacing the original by default. |
| Decompress one file | `gunzip app.log.gz` | Restores the uncompressed file. |

## Shell Environment and Editors

| Task | Command | Explanation |
|---|---|---|
| View command search paths | `echo "$PATH"` | Directories searched when you type a command name. |
| Add a directory for this shell | `export PATH="$HOME/bin:$PATH"` | Makes executables in `~/bin` discoverable temporarily. |
| Reload Bash configuration | `source ~/.bashrc` | Runs the Bash config in the current shell. |
| Set a shortcut | `alias ll='ls -lah'` | Creates an alias for the current shell. |
| Show how a command resolves | `type ls` | Reports whether `ls` is an alias, function, or executable. |
| Find a program | `command -v bash` | Portable shell way to locate a command. |
| Find executable path | `which bash` | Displays a binary path on systems with `which`. |
| Edit using Nano | `nano notes.txt` | Editor with shortcuts shown at the bottom. |
| Edit using Vim | `vim notes.txt` | Modal text editor; press `i` to insert. |
| Edit using Vi | `vi notes.txt` | Usually available even on minimal servers. |

### Make `PATH` changes persistent

Add the following line to `~/.bashrc` for new interactive **Bash** shells, then reload it:

```bash
export PATH="$HOME/bin:$PATH"
```

```bash
source ~/.bashrc
command -v my-tool
```

Login shells may load `~/.profile` or `~/.bash_profile` instead; check your shell's startup configuration. For Vim/Vi: **Esc** → `:wq` → **Enter** saves and quits; **Esc** → `:q!` discards unsaved changes; **Esc** → `:%d` deletes every line in the file.

## Bash Scripting

A Bash script is a text file containing shell commands. Start scripts with `#!/usr/bin/env bash` so the system knows how to execute them with Bash.

### Essential Bash syntax

| Task | Command or syntax | Explanation |
|---|---|---|
| Save a variable | `name="Linux"` | Assign without spaces around `=`. |
| Use a variable | `echo "Hello, $name"` | Double quotes allow expansion and protect spaces. |
| Read input | `read -r answer` | Read a line; `-r` treats backslashes literally. |
| Read a script argument | `echo "$1"` | `$1` is the first positional argument. |
| Test file existence | `[[ -f notes.txt ]]` | True when `notes.txt` is a regular file. |
| Make a decision | `if [[ -f notes.txt ]]; then echo found; else echo missing; fi` | Use a condition to choose an action. |
| Repeat across values | `for item in one two; do echo "$item"; done` | Runs once per item. |
| Run while condition is true | `while [[ -f /tmp/ready.flag ]]; do sleep 1; done` | Repeats while that file exists; stop with Ctrl+C. |
| Reuse a command sequence | `hello() { echo 'Hello'; }; hello` | Defines and calls a function. |
| Inspect last exit code | `echo "$?"` | `0` means success by shell convention. |
| Check script syntax | `bash -n script.sh` | Parses Bash without executing commands. |
| Trace script execution | `bash -x script.sh` | Prints commands as Bash executes them. |
| Execute a Bash file | `bash script.sh` | Runs the script even without execute permission. |
| Execute directly | `chmod +x script.sh && ./script.sh` | Marks it executable, then runs it. |

### Practical script: back up a directory

Save as `backup.sh`. This creates a timestamped archive rather than overwriting a previous backup.

```bash
#!/usr/bin/env bash
set -euo pipefail

source_dir="$HOME/project"
backup_dir="$HOME/backups"

if [[ ! -d "$source_dir" ]]; then
    echo "Source directory not found: $source_dir" >&2
    exit 1
fi

mkdir -p "$backup_dir"
archive="$backup_dir/project-$(date +%Y%m%d-%H%M%S).tar.gz"

tar -czf "$archive" -C "$HOME" project
echo "Backup saved: $archive"
```

Run: `bash backup.sh`. Create `~/project` first, or change `source_dir` and the `tar` source path together. `set -euo pipefail` stops on many errors, undefined variables, and pipeline failures.

### Practical script: count errors in a log

Save as `count-errors.sh`.

```bash
#!/usr/bin/env bash
set -euo pipefail

log_file="${1:-app.log}"

if [[ ! -f "$log_file" ]]; then
    echo "Log not found: $log_file" >&2
    exit 1
fi

error_count=$(grep -ic 'error' "$log_file" || true)
echo "Error lines: $error_count"
```

Run: `bash count-errors.sh app.log`. `"${1:-app.log}"` uses the first argument or defaults to `app.log`; `-i` ignores case, `-c` counts matching lines. `|| true` handles zero matches without stopping the script.

### Practical script: check a service

Save as `check-service.sh`. Requires a systemd-based Linux system.

```bash
#!/usr/bin/env bash
set -euo pipefail

service_name="${1:-ssh}"

if systemctl is-active --quiet "$service_name"; then
    echo "$service_name is running"
else
    echo "$service_name is not running"
    exit 1
fi
```

Run: `bash check-service.sh ssh` (use `sshd` on systems where that is the service name). `--quiet` checks the status through the exit code instead of printing output.

### Practical script: warn about disk usage

Save as `check-disk.sh`.

```bash
#!/usr/bin/env bash
set -euo pipefail

threshold=80
usage=$(df -P / | awk 'NR==2 {gsub(/%/, "", $5); print $5}')

if (( usage >= threshold )); then
    echo "WARNING: root filesystem is ${usage}% full"
    exit 1
fi

echo "OK: root filesystem is ${usage}% full"
```

Run: `bash check-disk.sh`. `df -P /` uses a predictable output format, `awk` extracts the usage percentage, and `(( ... ))` performs an integer comparison.

## Scheduling and Automation

`cron` runs commands on a schedule. Use absolute file paths in jobs; cron's environment and `PATH` can be different from your interactive shell.

| Task | Command | Explanation |
|---|---|---|
| Edit your scheduled jobs | `crontab -e` | Open your user crontab in an editor. |
| List scheduled jobs | `crontab -l` | Display your user's crontab. |
| Check cron service | `systemctl status cron` | Debian/Ubuntu service; Fedora/RHEL often uses `crond`. |
| Inspect cron logs | `journalctl -u cron -n 30 --no-pager` | Show recent cron service logs on systemd. |
| View the current date | `date '+%Y-%m-%d %H:%M:%S'` | Helpful when checking whether a job ran on schedule. |

### Cron expression structure

```text
minute hour day-of-month month day-of-week command
  0      2         *        *        *     /path/to/script.sh
```

`0 2 * * *` means **every day at 02:00** according to the cron server's timezone.

For example, after placing the backup script at `/home/USERNAME/scripts/backup.sh`, add this line using `crontab -e` (replace `USERNAME` with your actual login):

```cron
0 2 * * * /bin/bash /home/USERNAME/scripts/backup.sh >> /home/USERNAME/backup.log 2>&1
```

Here, `>>` appends standard output and `2>&1` also sends error messages into the same log. Verify that the directories and paths exist before scheduling it.

---

### Quick practice checklist

- [ ] Navigate folders, copy files, and search for `.log`/`.conf` files.
- [ ] Inspect file ownership and distinguish `chmod` from `chown`.
- [ ] Find a process, check a service, and inspect logs.
- [ ] Test network/DNS connectivity and copy a file using SSH.
- [ ] Check disk usage and diagnose a failed service.
- [ ] Run the four Bash scripts and schedule one safely using cron.

<sub>Linux commands can vary by distribution, privileges, and installed utilities. Practice administrative commands in a disposable VM or lab environment.</sub>

**Acknowledgment:** This guide expands on the original Linux command collection from [farzeen-ali](https://github.com/farzeen-ali/Linux-Commands-for-DevOps-Engineers); the structure, explanations, and scripting examples have been extended for a general Linux audience.
