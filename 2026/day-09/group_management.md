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
## Task 2: Shared Directory\

**create a dir in /opt**
```bash
sudo mkdir /opt/dev-projects
```

**set groups owners to developers**
```bash
sudo chown :developers /opt/dev-projects  # to change group owner for dir.

sudo chown tokyo /opt/dev-projects # to change user owner for dir.

sudo chown berlin:admins /opt/dev-projects #to change group and user owner for dir.

#to test if chnages has been made.
ls -lh # to see user and groud owner details for files and dir.
```
**Set permissions to 775 (rwxrwxr-x)**
```bash

chmod 775 /opt/dev-project # to give group access to write in a file or dir.
```

**Test by creating files as tokyo and berlin**
```bash

su - berlin #to change user and it requires pass.

sudo su tokyo # it does not require pass as you are using root to login

#you cannot log back in as root user after because berlin does not have access. you must have access to root credentials.

#now craete a file and test if everything works fine.
```


