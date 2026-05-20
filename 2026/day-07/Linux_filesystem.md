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

# this will sort the log files dir in log folder and display the 5 largest folders, also if tere will be any error message
it will be discarded using 2>/dev/null
```

```bash

cat /etc/hostname
#it iwll show IP add of the server
```
