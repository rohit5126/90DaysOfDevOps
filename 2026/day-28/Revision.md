### Navigate the file system, create/move/delete files and directories
```
cd
touch
mkdir
rm , rm -rvf
mv
cp
```
### Manage processes — list, kill, background/foreground
```
ps -aux
top
htop
kill
kill -9
&
```
### Work with systemd — start, stop, enable, check status of services
```
systemctl start/stop/restart/status/enable
systemctl list-units --type=service --state=running
systemctl is-enabled nginx
```
### Read and edit text files using vi/vim or nano
```
vim 
cat
```
### Troubleshoot CPU, memory, and disk issues
```
df -h
free -h
du -sh
top
```

### Explain the Linux file system hierarchy (/, /etc, /var, /home, /tmp, etc.)
```
/ -root
/etc - all the config files and user details.
/home - stores all the user dir
/tmp - store temporary files.
/var - the log files
/bin - all teh binary for commands.
```

### Create users and groups, manage passwords
```
adduser
addgroup
sudo usermod -aG <groupname> <username>
getent group <groupname>
getent passwd <useranme>
```

###  Set file permissions using chmod (numeric and symbolic)
### Change file ownership with chown and chgrp
```
chmod
chown -R owner:group file/dir
chgrp group file/dir

read=4
write=2
execute=1

chmod 754 file.txt
```
### Create and manage LVM volumes
```
lsblk
pvcreate <name>
vgcreate <name> <pv>
lvcreate -L 10G -n <name> <vg>
pvs
vgs
lvs
mkfs.ext4 <lv path>
mkfs -t ext4 <pv>
mount <lv path> <mount path>
lvextend  -L +5G /dev/cool_grp/cool_lv

resize2fs /dev/cool_grp/cool_lv 
df -h
```
### Check network connectivity — 
```
ping <ipadd>
ss -tulnp
nc -zv localhost 80
dig <url>
nslookup <url>
curl localhost:80
```
### Shell Scripting
```
read -p "enter a path" p
num=$1

if [ -f $p ]; then
    echo "file"
elseif [  -d $p ]; then
    echo "dir"
else
    echo "noob"
for i in {1..$num}; do
    echo "$i"
done
j=0
while [ $j -le $num ]; do
    echo "$j"
  ` j=$((j+1))
done

function add(){
    sum=$1+$2
    echo $sum
}

function 3 4
set -euo pipefail
```
#Git & GitHub
```
git init
git add .
git status
git config --global user.name
git config --global user.email
git config --list
git commit -m "thjij"
git checkout -b devops
git branch
git remote -v
git remote add orgin
git push origin master
git pull origin master
git fetch --all
git merge devops
git reabse master
git stash
git stash pop
git cherry-pick <commit-id>
git merge devops --squash HEAD~4
git reset <commit-id> --soft/--hard/--mixed
git revert <commit>
```
Task 3: Quick-Fire Questions
Answer these from memory (no Googling). Then verify your answers:

What does chmod 755 script.sh do?  give owner RWX, group RX, Other RX.

What is the difference between a process and a service? an instance of a program is process and specialized process is a  service.

How do you find which process is using port 8080? ss -tulnp | grep 8080

What does set -euo pipefail do in a shell script? -e exits when error, -u exist when unassigned variable, -o pipefail exits if any command in a 
pipe is failed

What is the difference between git reset --hard and git revert? reset --hard  will delete all the changes, while revert wil generate another 
commit.

What branching strategy would you recommend for a team of 5 developers shipping weekly?

What does git stash do and when would you use it?git stash stores your current work in backgroud which you can resume again using stash pop

How do you schedule a script to run every day at 3 AM? 00 03 * * * bash script.sh in crontab

What is the difference between git fetch and git pull? git fetch only download the changes while git pull downloads and force them into current working dir

What is LVM and why would you use it instead of regular partitions?LVm is logical volume manager, we use it bcoz it is easily resizable and 
movable and easy to manage istead of hard drives.


