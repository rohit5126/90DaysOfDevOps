Day 18 – Shell Scripting: Functions & intermediate Concepts.

## Task 1: Basic Functions
```bash
#!/bin/bash

function greet(){
        echo "Hello $1"

}

function add(){
        sum=$(($1+$2))
        echo "$sum"

}

greet rohit

read -p "enter two numbers: " a b

add a b
```

## Task 2: Functions with Return Values
```bash
#!/bin/bash

function disku(){
        re=$(df -h | awk 'NR==2 {print $4}')
        echo $re

}

function memory(){
        re=$(free -h | awk 'NR==2 {print $4}')
        echo $re
}

result1=$(disku)
echo "Free Disk space available is $result1"

result2=$(memory)
echo "Free memory avaialble is $result2"
```
## Task 3: Strict Mode — set -euo pipefail

Document: What does each flag do?
**By default, shell scripts continue running the next line even if a previous command failed or there is an undefined variable**

**set -e → this makes shell script exit immediately if previous commnads gives non zero exit code**

**set -u → when there is an undefined variable in the script it exits immediately with error - unbound variable**

**By default, the exit status of a pipeline is simply the exit status of the last command in the chain**

**set -o pipefail → when there is pipeline command in script this set ensures that the script only continues if both the commands in pipeline
is success, else the script will immediately exit.**
```bash
