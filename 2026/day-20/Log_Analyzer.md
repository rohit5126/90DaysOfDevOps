# Bash Scripting Challenge: Log Analyzer and Report Generator

## Task 1: Input and Validation
```bash
log_file=$1

function inputval(){
        if [ $# == 0 ]; then
                echo " run the script in below format "
                echo " ./log-anal.sh <log_file_path> "
                exit 1
        fi
        if [ -e "$log_file" ]; then
                echo "file exists working on analysis ...... "
                echo "--------------------------------------------------------------------------------------"
        else
                echo "error: File does not exist"
                exit 1
        fi
}

inputval
```

## Task 2: Error Count
```bash
function errorcount(){
        c=$(grep -c $1 /home/ubuntu/serverlog/Systemlogs.log)
        echo " Total count for error logs in the file is :  $c"  
        echo "    "
        echo "-------------------------------------------------------------------------------------"
}

errorcount ERROR
```

## Task 3: Critical Events
```bash
function criticalevent(){
        echo " Total number of Critical events with line number is listed below: "
        echo "      "

        echo "Lno. Date       Time      Event      ID"
        grep -n $1 /home/ubuntu/serverlog/Systemlogs.log
        echo "---------------------------------------------------------------------------------------"

}

criticalevent CRITICAL
```

## Task 4: Top Error Messages
```bash
function topmess(){
        echo " ------------ Top 5 ERROR messaged -------------"
        grep "$1" /home/ubuntu/serverlog/Systemlogs.log | awk '{$1=$2=$3=$4=$5=$6=""; print}' | sort | uniq -c | sort -rn
}

topmess ERROR
```

## Task 5: Summary Report

**log_anal.sh**

```bash
#!/bin/bash


function inputval(){
        log_file=$1
        if [ $# == 0 ]; then
                echo " run the script in below format "
                echo " ./log-anal.sh <log_file_path> "
                exit 1
        fi
        if [ -e "$log_file" ]; then
                echo "file exists working on analysis ...... "
                echo "--------------------------------------------------------------------------------------"
        else
                echo "error: File does not exist"
                exit 1
        fi
}



function errorcount(){
        c=$(grep -c $1 /home/ubuntu/serverlog/Systemlogs.log)
        echo " Total count for error logs in the file is :  $c"  
        echo "    "
        echo "-------------------------------------------------------------------------------------"
}


function criticalevent(){
        echo " Total number of Critical events with line number is listed below: "
        echo "      "

        echo "Lno. Date       Time      Event      ID"
        grep -n $1 /home/ubuntu/serverlog/Systemlogs.log
        echo "---------------------------------------------------------------------------------------"

}


function topmess(){
        echo " ------------ Top 5 ERROR messages -------------"
        grep "$1" /home/ubuntu/serverlog/Systemlogs.log | awk '{$1=$2=$3=$4=$5=$6=""; print}' | sort | uniq -c | sort -rn
        echo "---------------------------------------------------------------------------------------------------------------"
        echo ""
}

```

**log_report.sh**
```bash

#!/bin/bash

set -e

source /home/ubuntu/sysadm/log_anal.sh

t=$(date '+%Y-%m-%d') 
touch "/home/ubuntu/serverlog/log_report_$t.txt"

path="/home/ubuntu/serverlog/log_report_$t.txt"

file=$1

echo " ____________________________LOG_REPORT________________________________" > $path
echo " "
echo "--------date : $t---------" >> $path
b=$(basename $file)
echo " "
echo "---------------file name is $b ----------" >> $path
inputval $file

tl=$(wc -l $file)
echo " "
echo "------------ Number of lines in the file is $tl-------------" >> $path

echo ""

errorcount ERROR >> $path

topmess ERROR >> $path

criticalevent CRITICAL >> $path

echo "report generated successfully and file stored in $path"

```


