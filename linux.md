# Linux Recap (DevOps Focus)

## 1. The Basics

Linux is a **kernel**. A distribution (Ubuntu, Debian, RHEL, Fedora, Alpine) bundles the kernel with tools, a package manager, and sensible defaults.

The kernel manages:
- Processes
- Memory
- Devices
- The filesystem

Everything else runs on top of it.

---

## 2. Filesystem Layout

Everything hangs off `/`. **Everything is a file**, including devices and sockets.

| Directory | Purpose |
|-----------|---------|
| `/etc` | Configuration files |
| `/var` | Changing data: logs (`/var/log`), caches, spools |
| `/home` | User home directories |
| `/usr` | Installed programs and libraries |
| `/tmp` | Temporary files |
| `/proc`, `/sys` | Live views of kernel and process state |
| `/dev` | Device files |
| `/bin`, `/sbin` | Essential binaries |
| `/opt` | Optional / third-party software |
| `/root` | Root user's home |

---

## 3. Core Commands

### Navigation and files
```bash
ls -la          # list files with details
cd /path        # change directory
pwd             # print working directory
cp src dst      # copy
mv src dst      # move / rename
rm -r dir       # remove (careful)
mkdir -p a/b/c  # create nested directories
find / -name "*.log"
ln -s target link   # symbolic link
```

### Viewing text
```bash
cat file
less file
head -n 20 file
tail -n 50 file
tail -f /var/log/syslog   # follow live
```

### Searching and processing text
```bash
grep -rni "error" /var/log
awk '{print $1, $3}' file
sed 's/old/new/g' file
cut -d: -f1 /etc/passwd
sort file | uniq -c | sort -nr
wc -l file
```

### Pipes and redirection
```bash
cmd1 | cmd2        # pipe output of cmd1 into cmd2
cmd > file         # overwrite file with output
cmd >> file        # append output
cmd 2> err.log     # redirect errors
cmd > out.log 2>&1 # redirect both
```

---

## 4. Users, Groups, and Permissions

### Permission model
Each file has permissions for **owner**, **group**, and **others**: read (`r`=4), write (`w`=2), execute (`x`=1).

```bash
chmod 755 file        # rwxr-xr-x
chmod u+x script.sh   # add execute for owner
chown user:group file
```

### User management
```bash
useradd -m alice
passwd alice
usermod -aG sudo alice
userdel -r alice
id alice
```

Key files: `/etc/passwd`, `/etc/group`, `/etc/shadow`.

### sudo
`sudo` runs a command as root. Follow **least privilege**: root can do anything, including break everything.

---

## 5. Processes and Services

### Inspecting processes
```bash
ps aux
top        # or htop
pgrep nginx
```

### Signals
```bash
kill <pid>       # SIGTERM: polite stop
kill -9 <pid>    # SIGKILL: forced stop
kill -HUP <pid>  # reload config (for many daemons)
```

### Background jobs
```bash
cmd &       # run in background
jobs
fg %1
nohup cmd & # survive terminal close
```

### systemd
```bash
systemctl start|stop|restart nginx
systemctl enable nginx     # start at boot
systemctl status nginx
journalctl -u nginx        # service logs
journalctl -u nginx -f     # follow logs
```

---

## 6. Package Management

| Family | Tool | Examples |
|--------|------|----------|
| Debian / Ubuntu | `apt` | `apt update`, `apt install nginx`, `apt remove nginx` |
| RHEL / Fedora | `dnf` / `yum` | `dnf install nginx` |
| Alpine | `apk` | `apk add nginx` |

---

## 7. Networking

```bash
ip a                 # addresses
ip r                 # routes
ss -tulpn            # listening ports and owning processes
ping host
curl -I https://example.com
dig example.com      # DNS lookup
traceroute host
```

### Remote access and transfer
```bash
ssh user@host
ssh-keygen -t ed25519      # key-based auth (best practice)
scp file user@host:/path
rsync -avz src/ user@host:/dst/
```

### Firewalls
- `ufw` (simple, Ubuntu)
- `iptables` / `nftables` (lower level)

---

## 8. Storage

```bash
lsblk          # block devices
df -h          # filesystem usage
du -sh *       # directory sizes
mount / umount
```

- Persistent mounts are defined in `/etc/fstab`.
- Typical disk workflow: **partition, format (`mkfs`), mount**.
- **LVM** adds flexible volume management (PV, VG, LV) on top of raw disks.

---

## 9. Shell Scripting

```bash
#!/bin/bash
set -euo pipefail

name="world"

if [ -f /etc/os-release ]; then
  echo "Hello, $name"
fi

for f in *.log; do
  echo "Processing $f"
done

greet() {
  echo "Hi, $1"
}
greet "Satwik"
```

Essentials:
- Shebang (`#!/bin/bash`)
- Variables, conditionals, loops, functions
- Exit codes (`$?`): `0` means success
- `set -euo pipefail` for safer scripts
- `chmod +x script.sh` to make it executable

---

## 10. Troubleshooting Mindset

Work through these in order:

1. **Logs**: `journalctl`, `/var/log`, `dmesg`
2. **Resources**: `top`, `free -h`, `df -h`
3. **Network**: `ss`, `curl`, `ping`, `dig`
4. **Permissions and config**: ownership, modes, syntax errors in config files

Most problems come down to one of these four.

---

## 11. Why It Matters for DevOps

- Docker containers are Linux processes isolated with **namespaces** and **cgroups**.
- Kubernetes nodes are Linux machines.
- CI runners, servers, and cloud VMs are almost all Linux.
- Strong fundamentals make everything above them (containers, orchestration, automation) easier to debug.

---

## Quick Cheat Sheet

| Task | Command |
|------|---------|
| Disk usage | `df -h` |
| Memory | `free -h` |
| Who's listening | `ss -tulpn` |
| Follow a log | `tail -f file` |
| Service status | `systemctl status <svc>` |
| Service logs | `journalctl -u <svc> -f` |
| Find text | `grep -rni "text" path` |
| Find files | `find / -name "name"` |
| Fix permissions | `chmod` / `chown` |
| Kill a process | `kill <pid>` |