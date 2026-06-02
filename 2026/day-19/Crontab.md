# Day 19 – Shell Scripting Project: Log Rotation, Backup & Crontab

## Task 1: Log Rotation Script
```bash
#!/bin/bash

set -e
function check(){
        echo " run script with an argument "
        echo "./log_rotation source_path dest_path "
}

if [ $# == 0 ]; then
        check
fi

src=$1
dest=$2
timestamp=$(date '+%Y-%m-%d_%H-%M-%S')

function rotation(){

        tar -cvzf "$dest/backup_$timestamp.gz" "$src" > /dev/null 2>&1 
        echo "backup generated successfully for nginx $timestamp"

}

rotation
stdate=$3
endate=$4

function delete_log(){
        backup=($(find "$dest"  -maxdepth 1 -type "f" -newermt "$1" ! -newermt "$2"))

        if [ "${#backup[@]}" -eq 0 ]; then
                echo "No log file exist for time period provided"
        else
                for bk in "${backup[@]}"; do
                        rm -f $bk
                        echo "older logs deleted successfully"
                done
        fi
}

delete_log $stdate $endate
```

**addition info for this**

**Compresses .log files older than 7 days using gzip**

**Deletes .gz files older than 30 days**

```bash
tar -cvzf "$dest/"*.log "$src" #to convert all the log file is a dir.


function rotation(){
        find "$src"*.log -maxdepth 1 -type "f" | wc -l  #to get all the files getting compressed.
        tar -cvzf "$dest/backup_$timestamp.gz" "$src"*.log > /dev/null 2>&1
        echo "backup generated successfully for nginx $timestamp"

}

backup=($(find "/dest"*.gz -maxdepth 1 -type "f" -mtime +30   #to get files olderthan 30 days

function delete_log(){
        backup=($(find "$dest"*.gz  -maxdepth 1 -type "f" -mmin -130))  #to get files modified in last 130 mins

        if [ "${#backup[@]}" -eq 0 ]; then

                echo "No log file exist for time period provided"
        else
                echo "no of files deleted ${#backup[@]}"   #to get number of files deleted
                for bk in "${backup[@]}"; do
                        #rm -f $bk
                        echo $bk
                        echo "older logs deleted successfully"
                done
        fi
}
```

## Task 2: Server Backup Script

```bash
#!/bin/bash

src=$1
dest=$2

set -e

timestamp=$3

function server_backup() {
        tar -cvzf "$dest/backup_$timestamp.tar.gz" "$src" > /dev/null 2>&1
        echo "backup completed for server"
        du -sh "$dest/backup_$timestamp.tar.gz"


}

server_backup
```




