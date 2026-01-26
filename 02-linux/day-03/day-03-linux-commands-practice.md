# Day 03 – Linux Commands Practice & Nginx Service Management

## Overview

Today focused on building strong Linux command-line skills and applying them in real-world troubleshooting scenarios. The goal was to understand process management, file system navigation, and networking diagnostics.

Additionally, a practical exercise was performed by installing and managing an Nginx web server.

---

## Objectives

- Practice essential Linux commands
- Explore system logs
- Monitor processes and services
- Troubleshoot networking
- Install and manage a web server (Nginx)

---

## Commands Executed & Observations

### 1. Finding Log Files

```bash
sudo find /var/log -name "*.log"
```

**Outcome:**

- Discovered important system logs like:
  - `/var/log/syslog`
  - `/var/log/auth.log`
  - `/var/log/dpkg.log`

**Learning:**

- Logs are the first place to investigate issues in Linux systems

---

### 2. Real-Time Log Monitoring

```bash
tail -f /var/log/syslog
```

**Observation:**

- Detected AWS SSM-related errors:
  - EC2 role misconfiguration

**Learning:**

- `tail -f` is critical for live debugging
- Helps monitor applications and system behavior in real time

---

### 3. Process Inspection

```bash
ps aux | grep nginx
```

**Outcome:**

- Initially, no Nginx process was running

**Learning:**

- Used to verify if a service is active

---

### 4. Process Termination Attempt

```bash
kill -9 12524
```

**Outcome:**

- Error: No such process

**Learning:**

- Always verify PID before killing a process
- `kill -9` forcefully terminates processes

---

### 5. Checking Open Ports

```bash
ss -tuln
```

**Outcome:**

- Identified open ports:
  - Port 22 (SSH)
  - Port 53 (DNS)

**Learning:**

- Useful for debugging service exposure and connectivity issues

---

### 6. Network Connectivity Test

```bash
ping -c 4 google.com
```

**Outcome:**

- 0% packet loss

**Learning:**

- Confirms internet connectivity

---

### 7. DNS Resolution

```bash
dig google.com
```

**Outcome:**

- Successfully resolved domain to multiple IP addresses

**Learning:**

- Verifies DNS functionality

---

## Nginx Service Management (Hands-on)

### 8. Install Nginx

```bash
sudo apt install nginx -y
```

**Outcome:**

- Nginx installed successfully
- Service enabled automatically

---

### 9. Verify Running Process

```bash
ps aux | grep nginx
```

**Observation:**

- Found:
  - Master process
  - Worker processes

**Learning:**

- Nginx runs using a master-worker architecture

---

### 10. Verify Port 80

```bash
ss -tuln | grep 80
```

**Outcome:**

- Port 80 is listening on all interfaces

**Learning:**

- Confirms that the web server is accessible

---

### 11. Stop Nginx Service

```bash
sudo systemctl stop nginx
```

---

### 12. Check Service Status

```bash
sudo systemctl status nginx
```

**Outcome:**

- Service status: inactive (dead)

**Learning:**

- `systemctl` is the standard tool for service management

---

## Key Learnings

- Logs are essential for troubleshooting system issues
- `tail -f` is widely used in real-time debugging
- Always validate processes before killing them
- `ss` is preferred over `netstat` in modern systems
- Service lifecycle management is critical in DevOps
- Nginx follows a master-worker process model

---

## Real-World Relevance

These skills are directly applicable to:

- Debugging production outages
- Monitoring application logs
- Verifying service availability
- Troubleshooting network issues

---

## Conclusion

Day 03 provided hands-on experience with essential Linux commands and introduced real-world troubleshooting practices. Managing Nginx reinforced understanding of service operations, making this a strong foundation for DevOps workflows.

---
