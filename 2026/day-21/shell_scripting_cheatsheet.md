# Shell Scripting Cheat Sheet: My Own Reference Guide.

#___________________________________BASIC_________________________________________

```bash

name="rohit"

echo "the name is $name"

echo "this operation is running at $(date +%y-%m-%d_%H-%M-%S)"

echo "$0 is always the 0th argument which is script name"

echo "this is first args $1"

echo "this is second args $2"

echo "to see count of arguments run $#"

read -p "to take input from user use this commnd:  " input

echo $input


#for multi line comment.

<<comment

this 
is 
for multi 
line commment

comment
```
#___________________CONDITIONAL___________________________________
```bash
name=$1
if [ $name == "pinda" ]; then
        echo "this is from dhurandhar"
else
        echo "this is from another movie"
fi
```
#________________to check file exist or not_________________________
```bash
if [ -f $2 ]; then
        echo " file exist"
else
        echo "does not exist"
fi

#______use -d for dir_______
```

#________________________loops________________________________________

#______________for-Loops_____________________________________

```bash
for i in {1..5}; do
        echo "$i"
done

#(1..5) is the range from 1 to 5


for i in $(cat $2); do  #for i in $(cat text.txt); do
    echo $i
done 

```
#____________________________while loop______________________-
```bash
read -p "entier a number between 1 to 9:  " i

while [ $i -le 10 ]; do
        echo "you are dumb $i times"
        ((i++))

done
```
#________________________function__________________________________
```bash
read -p "enter name of user:  " name

function useradd(){

        sudo adduser $name

}

useradd

function verify(){
        if [ $(getent passwd $name) ]; then  #if checks if the exit code of the command is 0 or not, o means true and anything else means false
                echo "user created successfully"
        else
                echo "no user detected"

        fi
}

verify
```
**to use function in another script**

source ./functions.sh

verify

useradd

**to get past argument error**

set default value of first argument.

n=${1:-"defaultvalue"}

**$? shows the exit code of last ran command**

**$@ breaks your entered args at every empty space.**

**"$@" take every arg into a list**

Operator 
Description	Example
-eq	Equal to	[ $a -eq $b ]

-ne	Not equal to	[ $a -ne $b ]

-gt	Greater than	[ $a -gt $b ]

-ge	Greater than or equal to	[ $a -ge $b ]

-lt	Less than	[ $a -lt $b ]

-le	Less than or equal to	[ $a -le $b ]

String Comparison Operators Used to compare text strings. 

Operator 
Description	Example
= or ==	True if strings are equal	[ "$a" = "$b" ]

!=	True if strings are not equal	[ "$a" != "$b" ]

-z	True if string is empty	[ -z "$a" ]

-n	True if string is not empty	[ -n "$a" ]

< / >	Lexicographical order (requires [[ ... ]])	[[ "$a" < "$b" ]]


