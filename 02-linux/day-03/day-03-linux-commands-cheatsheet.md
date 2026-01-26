# Day 03 – Linux Commands Cheat Sheet

Quick, scannable commands for daily DevOps troubleshooting.

---

## Process Management

| Command           | What it does       | Example                |
| ----------------- | ------------------ | ---------------------- |
| `ps aux`          | List all processes | `ps aux`               |
| `top`             | Live CPU/mem usage | `top`                  |
| `htop`            | Interactive viewer | `htop`                 |
| `grep`            | Filter output      | `ps aux \| grep nginx` |
| `kill <PID>`      | Graceful stop      | `kill 1234`            |
| `kill -9 <PID>`   | Force stop         | `kill -9 1234`         |
| `pkill <name>`    | Kill by name       | `pkill nginx`          |
| `nice` / `renice` | Set priority       | `renice 10 1234`       |

---

## File System & Logs

| Command         | What it does      | Example                       |
| --------------- | ----------------- | ----------------------------- |
| `pwd`           | Current dir       | `pwd`                         |
| `ls -lah`       | List with details | `ls -lah`                     |
| `cd`            | Change dir        | `cd /var/log`                 |
| `mkdir`         | Create dir        | `mkdir app`                   |
| `rm -rf`        | Delete forcefully | `rm -rf tmp/`                 |
| `cp -r`         | Copy dirs         | `cp -r src dst`               |
| `mv`            | Move/rename       | `mv a b`                      |
| `cat`           | Print file        | `cat file`                    |
| `less`          | Paginated view    | `less file`                   |
| `head` / `tail` | First/last lines  | `tail -n 50 app.log`          |
| `tail -f`       | Follow logs live  | `tail -f /var/log/syslog`     |
| `find`          | Search files      | `find /var/log -name "*.log"` |
| `du -sh`        | Dir size          | `du -sh *`                    |
| `df -h`         | Disk usage        | `df -h`                       |

---

## Networking Troubleshooting

| Command      | What it does      | Example                       |
| ------------ | ----------------- | ----------------------------- |
| `ip addr`    | Show IPs          | `ip addr`                     |
| `ping`       | Connectivity test | `ping -c 4 google.com`        |
| `ss -tuln`   | Open ports        | `ss -tuln`                    |
| `curl -I`    | HTTP headers      | `curl -I https://example.com` |
| `wget`       | Download file     | `wget https://...`            |
| `dig`        | DNS lookup        | `dig google.com`              |
| `traceroute` | Path trace        | `traceroute google.com`       |

---

## Service Management (systemd)

| Command             | What it does       | Example                        |
| ------------------- | ------------------ | ------------------------------ |
| `systemctl status`  | Service state      | `systemctl status nginx`       |
| `systemctl start`   | Start service      | `sudo systemctl start nginx`   |
| `systemctl stop`    | Stop service       | `sudo systemctl stop nginx`    |
| `systemctl restart` | Restart            | `sudo systemctl restart nginx` |
| `systemctl enable`  | Auto-start         | `sudo systemctl enable nginx`  |
| `systemctl disable` | Disable auto-start | `sudo systemctl disable nginx` |

---

## Power Combos (Use Daily)

```bash
# Filter errors in live logs
tail -f /var/log/syslog | grep -i error

# Check if app is running and listening
ps aux | grep nginx
ss -tuln | grep 80

# Find large files
find / -type f -size +100M
```

---

## Pro Tips

- Prefer `ss` over `netstat` (faster, modern)
- Use pipes `|` + `grep` to filter noise
- Verify PID before `kill -9`
- Use `man <cmd>` for deep dive

---

## 1-Min Practice

```bash
# Check ports
ss -tuln

# Follow logs
tail -f /var/log/syslog

# DNS test
dig google.com
```

---

**Goal:** Be able to run these from memory during incidents.
