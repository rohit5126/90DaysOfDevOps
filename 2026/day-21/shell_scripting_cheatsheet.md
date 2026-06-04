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

for file in *.log; do
        echo $file
done

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

set default value of first argument. but $1 will still be empty if no args is passed. use $n in rest of the script if an argument is passed it will be assigned to n.

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

Operator               evaluation

-e                  The file or directory exists, regardless of its type.

-f                 The path exists and is a regular file (not a directory or device).

-d                  The path exists and is a directory.

-s                   The file exists and has a size greater than zero (not empty).

-r                   The file exists and grants read permission to the user.

-w                    The file exists and grants write permission to the user.

-x                   The file exists and grants execute permission to the user.

-o                  The file exists and is owned to the user.

-h/-l                The file exists and is a symbolic link.

-------------------------------------------------------------------------------------------------

Document the most useful flags/patterns for each:
1. `grep` — search patterns, `-i`, `-r`, `-c`, `-n`, `-v`, `-E`
   '''bash

   grep -c "error" file.log

   grep -n 'critical" file.log

   grep -i "error" file.log #case insensitive.

   grep -r "error" /home/ubuntu/backup

   grep -v "error" file.log #will search for all lines without error in it.
   ```
3. `awk` — print columns, field separator, patterns, `BEGIN/END`
   ```bash
   awk '{print $1, $3}' data.txt #prints 1st and 3rd column

   awk -F':' '{print $1}' /etc/passwd #this will change seperator to : by default it is white space.

   awk '/ERROR/ {print $0}' server.log #this will print lines that contain particula patter(ERROR)

   awk '$1 > 100 {print $1, $3}' rr.txt #to provide a condition

   awk '{sum += $1} END {print "Total Sum:", sum}' rr.txt
   
   #BEGIN blocks execute before any text is read.
   
   #END blocks execute after all input has been processed.

   awk -v limit=150 '$1 > limit' rr.txt #this takes a limit as 150 and prints all value > 150 in $1
   ```
   
5. `sed` — substitution, delete lines, in-place edit
   ```bash
   # Search & Replace: Replace the first occurrence of a word per line:
   sed 's/old-word/new-word/' filename.txt
   # Global Replacement: Replace every occurrence of a word per line:
   sed 's/old-word/new-word/g' filename.txt
   # Delete Lines: Delete blank lines in a file:
   sed '/^$/d' filename.txt
   # In-place Editing: Modify the actual file instead of just outputting the result to the terminal:
   sed -i 's/word/replacement/g' filename.txt
   ```
7. `cut` — extract columns by delimiter
   ```bash
   # Pull the 1st field from a comma-separated string
   echo "John,Doe,30,Engineer" | cut -d ',' -f 1
   # Output: John

   # Grab the 1st and 4th fields from the system passwd file
   cut -d ':' -f 1,4 /etc/passwd
   ```
9. `sort` — alphabetical, numerical, reverse, unique
    ```bash
    Alphabetical        |  Sort(None) |  sort names.txt
    Reverse Order       |     -r      |  sort -r names.txt
    Numeric Value Sort  |     -n      |  sort -n numbers.txt
    Unique Values Only  |     -u      |  sort -u items.txt
    Case-Insensitive    |     -f      |  sort -f mixed_case.txt
    Human-Readable Sizes|     -h      |  sort -h sizes.txt (e.g., 1K, 2M, 3G)
    Save In-Place       |     -o      |  sort -o file.txt file.txt
    ```
    
11. `uniq` — deduplicate, count
    ```bash
    Option  |      Purpose                                           |        PracticalExample
    Default |   Removes sequential duplicate lines.                  |      sort files.txt | uniq                            
    -c      | Prefixes each line with its count of occurrences.      |      sort files.txt | uniq -c             
    -d      | Prints only the lines that are duplicated.             |      sort files.txt | uniq -d                              
    -u      | Prints only unique lines (completely skips duplicates).|     sort files.txt | uniq -u
    -i      | Ignores differences in uppercase and lowercase.        |     sort files.txt | uniq -i
    
13. `tr` — translate/delete characters
    
15. `wc` — line/word/char count
    ```bash
    Option    |       Description        |    Output Example
     -l       | Counts newlines / lines. |    42 file.txt
     -w       | Counts words (delimited  
          byspaces, tabs, or newlines).  |   350 file.txt
     -m       |Counts characters.        |   2048 file.txt
     -c       |Counts bytes              
    (multi-byte characters like emojis   
    take up more bytes).                 |   2048 file.txt
     -L       |Prints the length of the
    longest line in characters.          |   17 file.txt
    ```
    
17. `head` / `tail` — first/last N lines, follow mode
-----------------------------------------------------------------------------------------------------------

# Task 6: Useful Patterns and One-Liners
Include at least 5 real-world one-liners you find useful. Examples:
- Find and delete files older than N days
  ## find /home/ubuntu/backup/*.gz -maxdepth 1 -type f mtime +N -delete
- Count lines in all `.log` files
  ## wc -l *.log
- Replace a string across multiple files
  ## sed 's/echo/print/g' *.sh
- Check if a service is running
  ## systemctl status nginx | grep -i active
- Monitor disk usage with alerts
```bash
#!/bin/bash

# --- CONFIGURATION ---
THRESHOLD=80                    # Alert trigger percentage (e.g., 80%)
EMAIL="admin@yourdomain.com"    # Destination email for alerts
HOSTNAME=$(hostname)            # Server name for context

# --- MONITORING LOGIC ---
# 1. df -Ph gets POSIX-compliant human-readable disk data.
# 2. grep -vE excludes standard pseudo-filesystems.
# 3. awk extracts the use percentage and the mount point partition.
df -Ph | grep -vE '^Filesystem|tmpfs|cdrom|devtmpfs' | awk '{ print $5 " " $6 }' | while read -r output; do
    
    # Extract numerical percentage value
    usage=$(echo "$output" | awk '{print $1}' | sed 's/%//g')
    partition=$(echo "$output" | awk '{print $2}')
    
    # Check if partition usage meets or exceeds the threshold
    if [ "$usage" -ge "$THRESHOLD" ]; then
        ALERT_MSG="Warning: Partition '$partition' on $HOSTNAME is at ${usage}% capacity!"
        echo "$ALERT_MSG"
        
        # Un-comment the line below to enable email alerts (requires mailutils/mailx)
        # echo "$ALERT_MSG" | mail -s "CRITICAL: Disk Space Alert on $HOSTNAME" "$EMAIL"
    fi
done
```
- Parse CSV or JSON from command line
```bash
# input.csv content:
# John,Doe,30,New York

# Skip headers if necessary, then read column by column
while IFS="," read -r first_name last_name age city; do
    echo "$first_name lives in $city."
done < input.csv
```

- Tail a log and filter for errors in real time
## tail -F /path/to/your/app.log | grep -i "error"



