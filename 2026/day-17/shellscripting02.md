# Shell Scripting: Loops, Arguments & Error Handling.

## Task 1: For Loop
```bash
#!/bin/bash

x=("apple" "mango" "lcihi" "gauva" "kela")

for i in ${x[@]}; do 
        echo "$i"
done

for i in {1..5}; do
        echo "$i"
done
```
## Task 2: While Loop
```bash
#!/bin/bash

read -p "enter a number of your choice: " n

while [ $n -gt 0 ]; do
        echo "$n"
        ((n--))
        $(sleep 1)
done

echo "you are done"
```

## Command-Line Arguments
```bash
#!/bin/bash
n=${1:-Usage: ./greet.sh}
if [ -z $1 ]; then
        echo "$n"

else
        echo " hello $1, what is fav tool"
fi

echo "$# total args"

echo "script name $0"

for i in $@; do
        echo "$i"
done
```

## Task 4: Install Packages via Script
```bash
#!/bin/bash
x=("nginx" "wget" "curl")
if [ "$EUID" -ne 0 ]; then   #to check if user running the script is root or not
        echo " run the script as root user "
        exit 1
fi
for i in "${x[@]}"; do
    if dpkg -s "$i" >/dev/null 2>&1; then  # by using this command we are sending both the stdout and stderr to /dev/null(discard). 
        echo "$i is installed"
    else
        echo "$i is not installed"
        echo "installing..........$i"
        apt-get install "$i" -y
        echo "$i installed successfully"
        sleep 2
    fi
done
```
**by using this command 'dpkg -s "$i" >/dev/null 2>&1' we are sending both the stdout and stderr to /dev/null(discard). >/dev/null this send stdout to discard. 2>&1 this send stderr to same place where stdout goes.
so commands print nothing only uses exit code in if. so when command gives exit code 0 which is true.if condition satisfies and return what is inside.
and when command gives any other exit code it becomes false and goes to else condition.**


## Task 5: Error Handling
```bash
#1/bin/bash

set -e

mkdir /tmp/devops-test || echo "dir already exist"

cd /tmp/devops-test

touch file1.txt
```
