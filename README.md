# linux-sysadmin-labs

This repo is a Linux/sysadmin portfolio. I passed CompTIA Linux+ (XK0-006) in October 2026 and here I document the exam concepts on real Linux distros (Rocky and Ubuntu).

## The lab

My lab runs on Oracle VirtualBox on my Windows desktop. Two VMs, each with two virtual NICs. The NAT adapter gives them outbound internet access through the host and the host-only adapter creates a private LAN, 192.168.56.0/24, between the two VMs and my desktop.

```
desktop   Windows 11         192.168.56.1     host
rocky     Rocky Linux 9.8    192.168.56.101   firewalld, SELinux enforcing
ubuntu    Ubuntu 24.04.5     192.168.56.102   ufw, nginx
```

## Table of contents

| Topic | Scope | Log |
|---|---|---|
| [Storage](troubleshooting/storage-fstab-raid.md) | LVM build and grow, RAID 1 disk failure and replace, a broken fstab found and fixed | [12 Nov 2026](logs/storage-fstab-raid_2026-11-12.log) |
| [Firewall](troubleshooting/firewalld-and-ufw.md) | a real fault I found, firewalld and ufw running at the same time on Ubuntu | [13 Nov 2026](logs/firewalld-and-ufw_2026-11-13.log) |

## Logs

For each lab the whole session is recorded with the script command and every session was done by hand.

## AI disclosure

I used AI to organize my tasks and priorities, both while learning and while documenting this repo. I also used Claude to write the break scripts, so I could do cold troubleshooting without knowing the fault in advance. AI was also used in order to clean the raw logs, removing the terminal escape codes so the content is easier to follow. Each session is recorded as a whole, nothing was cut from the sessions.
