# Introduction to Ansible and Inventory Setup

## Task 1: Understand Ansible

**What is configuration management? Why do we need it?**

Configuration management is the process of tracking and controlling the hardware, software, and settings of a system to ensure it performs exactly as intended

**How is Ansible different from Chef, Puppet, and Salt?**

Ansible is Agentless
Ansible (Declarative/Procedural Hybrid)
Ansible (Push)

**What does "agentless" mean? How does Ansible connect to managed nodes?**

Agentless means you do not install any special software on the target systems (the managed nodes).

**For Linux/Unix Nodes**: Ansible connects using SSH (Secure Shell). It authenticates using your existing SSH keys or passwords, just like a human administrator logging into a remote terminal.

**For Windows Nodes**: Ansible connects using WinRM (Windows Remote Management) or OpenSSH for Windows.

**For Network Devices**: Ansible connects via SSH or vendor-specific APIs (like netconf) to manage switches, routers, and firewalls.


**Draw or describe the Ansible architecture:**

**Control Node** -- the machine where Ansible runs (your laptop or a jump server)
**Managed Nodes** -- the servers Ansible configures (your EC2 instances)
**Inventory**-- the list of managed nodes
**Modules** -- units of work Ansible executes (install a package, copy a file, start a service)
**Playbooks** -- YAML files that define what to do on which hosts

```

       +-----------------------------------------------------------+

       |                       CONTROL NODE                        |
       |  (Your laptop, Jenkins, or a dedicated management server) |
       +-----------------------------------------------------------+
                                     |
         +---------------------------+---------------------------+

         |                           |                           |
         v                           v                           v
+-----------------+         +-----------------+         +-----------------+

|    PLAYBOOKS    |         |    INVENTORY    |         |  ANSIBLE CORE   |
| (YAML files defining|     | (List of server |         |    & MODULES    |
| automation tasks)|         |  IPs & groups)  |         | (Built-in code) |
+-----------------+         +-----------------+         +-----------------+

         |                           |                           |
         +---------------------------+---------------------------+
                                     |
                                     | (Compiles code & reads targets)
                                     v
                        +-------------------------+

                        |  SSH / WinRM CONNECTION |  <--- No agent needed!
                        +-------------------------+
                                     |
           +-------------------------+-------------------------+

           |                         |                         |
  (Pushes temp script)      (Pushes temp script)      (Pushes temp script)

           |                         |                         |
           v                         v                         v
+--------------------+    +--------------------+    +--------------------+

|    MANAGED NODE    |    |    MANAGED NODE    |    |   NETWORK DEVICE   |
|    (Web Server)    |    |   (Database API)   |    |  (Switch/Router)   |
|                    |    |                    |    |                    |
| * Runs Python      |    | * Runs Python      |    | * Runs Netconf API |
| * Deletes script   |    | * Deletes script   |    | * Saves config     |
+--------------------+    +--------------------+    +--------------------+

```


## Task 2: Set Up Your Lab Environment

**setup lab enviroment using terraform module.**

## Task 3: Install Ansible

**install ansible on local machine**

```
# macOS
brew install ansible

# Ubuntu/Debian
sudo apt update
sudo apt install ansible -y

# Amazon Linux / RHEL
sudo yum install ansible -y
# or
pip3 install ansible

# Verify
ansible --version

```

**Document:** On which machine did you install Ansible? Why is it only needed on the control node?

on host machine or local machine, which is the control node.

## Task 4: Create Your Inventory File

first step should be to ssh each instance or add them manually to your .ssh/known-hosts.

The inventory tells Ansible which servers to manage. Create a project directory and your first inventory:

```
[server]
dev1 ansible_host=16.16.57.50
dev2 ansible_host=13.53.214.58

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/rohit/Downloads/k8s-key.pem
```


## Task 5: Run Ad-Hoc Commands

1. **Check uptime on all servers:**
```bash
ansible all -i inventory.ini -m command -a "uptime"
```

2. **Check free memory on web servers only:**
```bash
ansible web -i inventory.ini -m command -a "free -h"
```

3. **Check disk space on all servers:**
```bash
ansible all -i inventory.ini -m command -a "df -h"
```

4. **Install a package on the web group:**
```bash
ansible web -i inventory.ini -m yum -a "name=git state=present" --become
```
(Use `apt` instead of `yum` if running Ubuntu)

5. **Copy a file to all servers:**
```bash
echo "Hello from Ansible" > hello.txt
ansible all -i inventory.ini -m copy -a "src=hello.txt dest=/tmp/hello.txt"
```

6. **Verify the file was copied:**
```bash
ansible all -i inventory.ini -m command -a "cat /tmp/hello.txt"
```

**Document:** What does `--become` do? When do you need it?

the become command is used to be sudo user and run commands require root privileges.

-----------------------------------------

## Task 6: Explore Inventory Groups and Patterns

**Create a group of groups**

```
[server]
dev1 ansible_host=16.16.57.50
dev2 ansible_host=13.53.214.58


[web]
web1 ansible_host=13.61.155.255
web2 ansible_host=13.60.219.148

[db]
db1 ansible_host=13.60.9.5
db2 ansible_host=16.171.116.232

[application]
dev1
dev2
web1
web2

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/rohit/Downloads/k8s-key.pem

```

```
  651  ansible web  -m ping
  653  ansible db -m ping
  655  ansible application  -m ping
```

**Use patterns:**

```
ansible 'web:app' -i inventory.ini -m ping        # OR: web or app
ansible 'all:!db' -i inventory.ini -m ping        # NOT: all except db
```



