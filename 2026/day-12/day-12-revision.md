# Take a one-day pause to consolidate everything from Days 01–11 so you don’t forget the fundamentals you just built.

## Processes & services
```bash
ps -aux

top -p [pid]

systemctl status nginx

ps -ef

systemctl cat nginx
```
## File skills
```bash
ls -lh

ls -la

df -h

free -h

touch file1.txt

chown tokyo:admins file1.txt

chmod 644 file.txt
```

## Cheat sheet refresh
```bash

#five commands I will use first in the incident
```bash

systemctl status service

free -h

ps -aux

df -h

journalctl -u nginx -n 50
```

## Mini Self-Check (write short answers in day-12-revision.md)
1. Which 3 commands save you the most time right now, and why?
ps -aux - because it shows process along with memory and disk usage.
nc -zv localhost 80 - it shows it the port for the service is working.
systemctl list-units --type=service --state=running - it shwos all the running services

2. How do you check if a service is healthy? List the exact 2–3 commands you’d run first.
systemctl status service
systemctl is-enabled service

sudo netstat -tulnp | grep service



3. How do you safely change ownership and permissions without breaking access? Give one example command.
chown user:group file

chgrp group file


4. What will you focus on improving in the next 3 days?
practising commands as much as possible.
increase my voacbulary of commands


