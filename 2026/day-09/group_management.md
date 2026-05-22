# Linux User & Group Management Challenge

## Task 1: Create Users

**create users**
```bash
sudo adduser tokyo
sudo adduser berlin
sudo adduser professor
```
**to check users added successfuly**
```bash
cat /etc/passwd | grep tokyo
```

**create groups**
```bash

sudo adduser --group admins

sudo addgroup developers
#both works the same

#to check groups added
cat /etc/groups
```
**assign users to group**
```bash

sudo usermod -aG developers berlin

sudo usermod -aG admins professor

# to check users group
groups berlin
groups professor

#to check list of users in group
getent group admins

#to remove user for group
sudo usermod -rG admins berlin
```
## Task 2: Shared Directory
