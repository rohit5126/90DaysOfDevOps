# File Permissions & File Operations Challenge.

## Task 1: Create Files
```bash

touch devops.txt
echo "this is the first line in a file. stay happy and focus on success" > notes.txt
vim script.sh

ls -lh   #to verify permissions
```

## Task 2: Read Files
```bash

cat notes.txt

cat /etc/passwd | head -5

cat /etc/passwd | tail -5

or
tail -5 /etc/passwd
```

## Task 3: Understand Permissions

## Format: rwxrwxrwx (owner-group-others)

**it is divided in 3 spaces. first three belongs tto owner, second three belongs to group, third three belongs to others**

r = "read" = 4
w = "write" = 2
x = "executable" = 1

**r+w+x**

for example:

a file in /opt dir has root owner and no other user can edit it.

permission = (rwxr-xr-x) = ((4+2+1)(4+0+1)(4+0+1)) = 755

if you want to give same permission to another directory - "chmod 755 dir"


# Task 4: Modify Permissions 
```bash
ls -lh

chmod +x script.sh

./script.sh

chmod 444 devops.txt

chmod 640 notes.txt

mkdir project && chmod 755 project

ls -l   #to verify after change

```
# Task 5: Test Permissions

**Try writing to a read-only file - what happens?**
when trying to write using cat or echo it shows permission denied, but while using vim it can be overriden by using '!' while saving the file.

**Try executing a file without execute permission**
getting permission denied error.

## what I learnt today?

**Firstly i learned about the permissions and how it works.
secondly I got to know about few new commands to add user, group and to manage them.
thirdly i learned about chnaging onwer and group of a file or dir and to add users to group for access. It was very important topic but a small topi.
which usually people miss.**


