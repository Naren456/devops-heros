# Linux Fundamentals Assignment

## Student Details

- **Name:** Naren456
- **Enrollment Number:** 24bcs10225

---

## Overview

This assignment covers the core essentials of Linux operating system usage that are required for DevOps, system administration, and cloud operations.

The focus is on understanding how to work efficiently from the command line, manage files, control permissions, monitor processes, and troubleshoot common system issues.

---

## Objectives

By the end of this assignment, the learner should be able to:

- navigate the Linux filesystem
- create, modify, and delete files and directories
- understand users, groups, and file permissions
- manage running processes
- use networking and package management commands
- work with shell scripts and basic automation
- troubleshoot system problems effectively

---

## Core Linux Concepts

### 1. Filesystem and Navigation

Linux uses a hierarchical filesystem. Common directories include:

```bash
/
/bin
/etc
/home
/var
/tmp
/usr
```

Useful commands:

```bash
pwd
ls
ls -la
cd /home
mkdir demo
touch file.txt
cp file.txt backup.txt
mv file.txt newname.txt
rm newname.txt
```

### 2. Permissions

Linux permissions are based on owner, group, and others.

```bash
chmod 755 script.sh
chmod +x script.sh
chown user:group file.txt
```

Permission breakdown:

- `r` = read
- `w` = write
- `x` = execute

### 3. Processes and Services

You can view and manage active processes with:

```bash
ps aux
top
htop
kill 1234
kill -9 1234
```

This is important for monitoring resource usage and stopping services.

### 4. Networking

Linux systems often need networking checks such as:

```bash
ip addr
ifconfig
ping google.com
curl http://localhost
netstat -tulpn
ss -tulpn
```

### 5. Package Management

Common package managers include:

```bash
apt update
apt install nginx
apt upgrade
yum install httpd
dnf install curl
```

### 6. Shell Basics

The shell is the primary interface for system interaction. Common commands:

```bash
echo "Hello"
cat /etc/os-release
uname -a
whoami
date
history
```

---

## Practical Commands

```bash
ls -l /etc
df -h
du -sh /
free -m
uname -a
cat /etc/passwd
grep "root" /etc/passwd
```

These commands help in understanding system health, storage usage, memory, and users.

---

## Shell Scripting Concepts

A shell script can automate repetitive tasks.

```bash
#!/bin/bash
echo "Welcome to Linux"
mkdir -p /tmp/demo
ls /tmp/demo
```

Run it with:

```bash
chmod +x script.sh
./script.sh
```

---

## Troubleshooting Basics

Common troubleshooting tasks:

- checking service status
- reviewing logs in `/var/log`
- verifying network connectivity
- ensuring correct file permissions
- checking whether a process is running

Example:

```bash
journalctl -xe
ls /var/log
tail -f /var/log/syslog
```

---

## Why Linux Matters in DevOps

Linux is the foundation of most cloud infrastructure, servers, containers, and automation pipelines. DevOps engineers regularly use Linux for:

- managing servers
- deploying applications
- writing shell scripts
- working with containers and Kubernetes
- monitoring infrastructure

---

## Key Takeaways

- Linux is command-line driven and highly flexible.
- Filesystem structure and permissions are essential foundations.
- Shell commands help manage systems efficiently.
- Networking and process monitoring are core system tasks.
- Linux knowledge is critical for DevOps and cloud environments.

---

## Final Checklist

- [x] Learned basic Linux commands
- [x] Understood file hierarchy and navigation
- [x] Practiced permissions and ownership
- [x] Explored process and memory management
- [x] Reviewed networking and package tools
- [x] Understood the importance of Linux for DevOps

---

## References

- Linux command line notes
- Course session material for Linux fundamentals
- Practical system administration exercises
