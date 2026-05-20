# Part 1: Linux File System Hierarchy.

## Core Directories (Must Know):**

**/ (root)** - This directory is the topmost directory. it is the starting point and all the files and dir in your system is located somewhere in root dir.
it is a speacial dir bcoz it has no parent dir.
All the important dir are inside root dir, ex- bin, sbin, opt, sys, etc and many more.

**/home** - this directory is personnel storage space for users. where all the files ad documents for user is stored. each user 
on the system has his own unique home dir.

**/root** - this is private home folder for admin or root user.

**/etc** - this directory is central nerve system for linux. which conatins systemwide configuration files which dictates how operating system,
services and application will work.
it conatins config files for all the services like nginx, ssh,cron.

**/var/log** - this directory is centralized location in linux for all the system logs, errors, appliaction logs,
system events security activities.(this is most importat dir for devops engineer).

**/tmp** - it is standard folder for storing small, short lived temporarry files. every user and serice has access to this folder.

## Additional Directories (Good to Know):

**/bin** - this directory contains all the essential commands binaries required by system to function.

**/sbin** - this directory contains all the boot commands binaries which are used by sysadmin for administration and maintainence.

**/usr/bin** - this is the primary location for command binaries that available to all users.

**/opt** - it stand for optional and is the standard loction for all third party or additional application or software packages.

## few commands
```bash
du -sh /var/log/* 2>/dev/null | sort -h | tail -5

# this will sort the log files dir in log folder and display the 5 largest folders, also if tere will be any error message it will be
discarded using 2>/dev/null
```

```bash

cat /etc/hostname
#it will show IP add of the server
```

# Part 2: Scenario-Based Practice

**Question: How do you check if the 'nginx' service is running?**

```bash
systemctl status nginx
#this command to see the status of the service.

systemctl list-units --type=service --state=running | grep nginx
#this command to see if the service exists in the machine if servivce not found in the above command.

systemctl is-enabled nginx
#this command to see if the service is enabled.
```

**A web application service called 'myapp' failed to start after a server reboot.
What commands would you run to diagnose the issue?
Write at least 4 commands in order.**

```bash
systemctl status myapp
#this command to check the status of the service.

systemctl is-enabled myapp
#to check if the service is enabled.

ps -aux | grep myapp
#this command to check if the service is there and to get it pid.

journalctl -u myapp -n 50
#this to check the logs of the service.

sudo systemctl restart myapp
#this command to restart the service, if it fails kill the service and restart again.

kill -9 [pid]
#this to kill the sleeping service or zombie service. and restart the service again.
```

**Your manager reports that the application server is slow.
You SSH into the server. What commands would you run to identify
which process is using high CPU?**

```bash
top or htop
#this command to check the memory usage fo services.

free -h
#to check if the memory is full and to be sure that the issue is cause of memory being full and not network issue.

ps aux --sort=-%cpu | head -10
#this command to get the process using maximum CPU if the memory is full.
```

**A developer asks: "Where are the logs for the 'docker' service?"
The service is managed by systemd.
What commands would you use?**

```bash

journalctl -u docker >> dockerlogs.txt
#this to store all the logs for docker into a text file and sen it to developer.

```

**A script at /home/user/backup.sh is not executing.
When you run it: ./backup.sh
You get: "Permission denied"
What commands would you use to fix this?**

```bash
ls -lh
#this command to check if the file has executable permision for the user or not.

chmod +x backup.sh
#thsi command to give all users the permisiion to execute the file.
```



