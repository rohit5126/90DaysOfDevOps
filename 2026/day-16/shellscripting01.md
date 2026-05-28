# Day 16 – Shell Scripting Basics
## Task 1: Your First Script

```bash
touch hello.sh

echo "hello devops" > hello.sh

chmod 764 hello.sh

./hello.sh
```

## Task 2: Variables
```bash
vim variables.sh
-----------------------------------------------
#!/bin/bash

name="rohit"
role="devops engineer"

echo " I am $name and my job is $role"
------------------------------------------------

```
## Task 3: User Input with read
```bash
vim greet.sh

-----------------------------------------------
#!/bin/bash

read -p "Hi !! what is your name " name

echo " hello $name, what is fav tool"
-----------------------------------------------
```

## Task 4: If-Else Conditions

Operator 
Description	Example
-eq	Equal to	[ $a -eq $b ]
-ne	Not equal to	[ $a -ne $b ]
-gt	Greater than	[ $a -gt $b ]
-ge	Greater than or equal to	[ $a -ge $b ]
-lt	Less than	[ $a -lt $b ]
-le	Less than or equal to	[ $a -le $b ]
String Comparison Operators
Used to compare text strings. 

Operator 
Description	Example
= or ==	True if strings are equal	[ "$a" = "$b" ]
!=	True if strings are not equal	[ "$a" != "$b" ]
-z	True if string is empty	[ -z "$a" ]
-n	True if string is not empty	[ -n "$a" ]
< / >	Lexicographical order (requires [[ ... ]])	[[ "$a" < "$b" ]]

```bash

vim check-number.sh
------------------------------------------------------------
#!/bin/bash

read -p "enetr a number: " n

if [ $n -gt 0 ]; then
        echo " $n is a positive number"
elif [ $n -lt 0 ]; then
        echo "$n is a negative number"
else 
        echo "are you dumb to enter zero"
fi

if [ -f $1 ]; then
        echo "file exist"
else
        echo "no file"
fi
-------------------------------------------------------------
```

## Task 5: Combine It All
```bash
vim check.sh
------------------------------------------------------------------------------------------------------
#!/bin/bash

n=${1:-"invalid"}
if [ $n == "invalid" ]; then
        echo " enter a valid service name"
        exit 1
else
        output=$(systemctl list-units --type=service --state=running | grep $1)
        if [ -z "$output" ]; then
                echo "service is not running or not present or you have enter the incorrect name"
        else
                echo "service is running"
        fi
fi
---------------------------------------------------------------------------------------------------------

./check.sh ssh
```



