# shell cheat sheet

#!/bin/bash

#___________________________________BASIC_________________________________________


name="rohit"
echo "the name is $name"

echo "this operation is running at $(date +%y-%m-%d_%H-%M-%S)"

echo "$0 is always the 0th argumnet"

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

#___________________CONDITIONAL____________________________

name=$1
if [ $name == "pinda" ]; then
        echo "this is from dhurandhar"
else
        echo "this is from another movie"
fi

#________________to check file exist or not_________________________

if [ -f $2 ]; then
        echo " file exist"
else
        echo "does not exist"
fi

#_- use -d for dir_




#________________________loops________________________________________

#______________for-Loops_____________________________________


for i in {1..5}; do
        echo "$i"
done

#(1..5) is the range from 1 to 5


for i in $(cat $2); do  #for i in $(cat text.txt); do
    echo $i
done 


#____________________________while loop______________________-

read -p "entier a number between 1 to 9:  " i

while [ $i -le 10 ]; do
        echo "you are dumb $i times"
        ((i++))

done

#________________________function__________________________________
```bash
read -p "enter name of user:  " name

function useradd(){

        sudo adduser $name

}

useradd

function verify(){
        if [ $(getent passwd $name) ]; then
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



