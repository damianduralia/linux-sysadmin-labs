# Firewall lab: two firewalls on one system

Ubuntu 24.04 and Rocky Linux 9 VMs.

This is a reproduction of a real fault that was found and fixed in October 2026. I considered it worth documenting, so in Nov 2026 I recreated it.

## Two firewalls on ubuntu

While learning ufw and firewalld I tried to prove that firewalld is not installed on Ubuntu systems. I ran firewall-cmd expecting "command not found" but it answered. It was in fact installed, running and configured. The base Ubuntu firewall ufw was also installed running and configured.

The firewalld presence was most probably a leftover from a learning session in the summer when I first started to study firewalls and the rocky VM was not installed yet. Later I got the rocky VM but ubuntu was not properly cleaned after that first firewalld session. The zone files in /etc/firewalld/zones/ were last saved 19 Jul.

Since two firewalls were configured and running, any packet this system received had to comply with both firewalld and ufw rules, and firewalld running on an Ubuntu system would easily pass unnoticed when troubleshooting a drop/deny packet issue.

## Reproduction

At the start ubuntu had only ufw running, allowing 22 and 443. I turned firewalld back on like it was in Oct, enabled and running, the ufw firewall was untouched. The firewalld zone allowed ssh and https but not 80.

Configured ufw to ALLOW port 80 for nginx. Requested port 80 from rocky with curl and got err 7 No route to host. Since pings worked it's not a routing issue. ubuntu is reachable but firewalld is rejecting the packet.

```console
damian@ubuntu:~$ sudo ufw allow 80/tcp
Rule added
Rule added (v6)
damian@ubuntu:~$ ssh rocky curl -sS -m 5 -I http://ubuntu
damian@rocky's password:
curl: (7) Failed to connect to ubuntu port 80: No route to host
damian@ubuntu:~$ ssh rocky ping -c 2 ubuntu
damian@rocky's password:
PING log01 (192.168.56.102) 56(84) bytes of data.
...
2 packets transmitted, 2 received, 0% packet loss, time 1006ms
```

log01 is the ubuntu system. rocky's /etc/hosts lists it as log01 with ubuntu as an alias.

ufw has port 80 configured with allow but firewalld has no port 80. Any packet has to go through 2 firewalls so it has to comply with both firewalls rules.

## Fix

The fix is to disable, mask or remove firewalld from ubuntu because the standard Ubuntu firewall is ufw. I disabled it and tested the same request from rocky. As expected this time it worked.

```console
damian@ubuntu:~$ sudo systemctl disable --now firewalld
Removed "/etc/systemd/system/multi-user.target.wants/firewalld.service".
Removed "/etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service".
damian@ubuntu:~$ ssh rocky curl -sS -m 5 -I http://ubuntu
damian@rocky's password:
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
...
```

Then I cleaned the env to the pre lab state, removed the 80 rule, ufw only, firewalld off and disabled. Port 80 is denied by ufw again and ufw drops the packet so rocky gives up after the 5 s timeout.

First run Oct 2026. Rerun + logged in Nov 2026

Recorded the whole terminal session with script in [firewalld-and-ufw_2026-11-13.log](../logs/firewalld-and-ufw_2026-11-13.log)
