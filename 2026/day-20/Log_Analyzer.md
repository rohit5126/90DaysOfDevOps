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
```bash

