# File Ownership Challenge (chown & chgrp)

## Task 1: Understanding Ownership 
```bash
ls -lh
**all files are owned by ubuntu user and group ubuntu.
```
## ask 2: Basic chown Operations
```bash
touch devops-file.txt
ls -l
# owner is ubuntu

sudo chown tokyo devops-file.txt
```

## Task 3: Basic chgrp Operations
```bash
touch team-notes.txt

ls -l
# group is ubuntu

sudo addgroup heist

cat /etc/group  #to check the group is added

sudo chgrp hiest team-notes.txt

ls -l # verify the changes
```

## Task 4: Combined Owner & Group Change 

```bash
touch project-config.yaml

sudo chown professor:hiest project-config.yaml

mkdir aap-logs

sudo chown berlin:hiest app-logs

ls -l
```

## Task 5: Recursive Ownership 
```bash
mkdir -p heist-project/vault
mkdir -p heist-project/plans
touch heist-project/vault/gold.txt
touch heist-project/plans/strategy.conf

sudo addgroup planner

sudo chown professor:planner -R heist-project

ls -lR heist-project  #this is to verify all the files and sub dir under heist-project has been changed owner and group
```



