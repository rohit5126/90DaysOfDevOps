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

## Task 5: Team Workspace
```bash
sudo adduser nairobi #to add new user

cat /etc/passwd # to check if user is added

sudo addgroup project-team #to add group

cat /etc/group #to check if group is added

sudo usermod -aG project-team nairobi #to add nairobi to the new group project-team
sudo usermod -aG project-team tokyo   #to add tokyo to the new group project-team

groups nairobi #checking nairobi groups

getent group project-team # to see list of users in project-team

sudo mkdir /opt/team-workspace #creating a new dir
cd /opt
ls
ls -lh # checking the permissions and other details for the dir.

sudo chown :project-team team-workspace  #changing group ownership of the dir.

ls -lh

sudo chmod 775 team-workspace #adding write permission for the group

ls -lh  #confirming the changes
su - nairobi #log in as user nairobi to test
```
