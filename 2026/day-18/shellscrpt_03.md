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
#!/bin/bash

set -euo pipefail
n=2
if [ $n == 0 ]; then
        echo "great"
else
        echo "not great"
fi

getent passwd tokyo >/dev/null 2>&1 | df -h >/dev/null
echo $?

echo "file creation completed"
echo "if this runs that means set is not workinig"
```

## Task 4: Local Variables
```bash
#!/bin/bash

read -p "enetr a number: " n

function number(){
        local m=5
        echo "global varaible is $n"
        echo "local variabke is $m"
}

number

echo "global varaible work everywhere so here it is $n"

echo "local varaibe does not $m"
```

## Task 5: Build a Script — System Info Reporter
Sort by Usage Percentage (Busiest first):
df -h | sort -hrk 5
-h: Compares human-readable numbers (e.g., 2K, 1G).
-r: Reverses the result (largest or highest percentage first).
-k 3: Specifies the 3rd column, which is used column.

```bash
#!/bin/bash

#function to print hostname and OS info

function info(){
        re=$(hostname && uname -o)
        echo $re
}

function uptimeinfo(){
        re=$(uptime -p)
        echo $re
}

function diskusage(){
        echo "the disk usage list is below "
        echo "Name             size  used   avail "
        df -h | sort -hrk 3 | head -5

}
