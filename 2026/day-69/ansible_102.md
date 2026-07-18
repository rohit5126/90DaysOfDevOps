# Ansible Playbooks and Modules

## Task 1: Your First Playbook

```
---
- name:  install and enable nginx service 
  hosts: app
  become: true

  tasks:
    - name: update
      shell: apt-get update
    - name: install nginx
      apt:
        name: nginx
        state: present 
    - name: enable nginx 
      service:
        name: nginx
        state: started
        enabled: true
    - name: copy index page
      copy:
        src: /home/rohit/ansible/index.html
        dest: /var/www/html/index.html
```

**Verify:** Curl the web server's public IP. Do you see your custom page?

yes I am a able to see custom web page.

--------------------------------------------------

## Task 2: Understand the Playbook Structure

```
---                             #yaml docsument start for ansible
- name:  install and enable nginx service  #name of the play
  hosts: app                    #inventory or group of hosts 
  become: true                  # run tasks as root be sudo

  tasks:                       
    - name: update              #first task and its name
      shell: apt-get update     #module name
    - name: install nginx       
      apt:                      #module name 
        name: nginx             #module args
        state: present
```

**What is the difference between a play and a task?**

A play is an organizational container that maps a specific group of servers (hosts) to a set of actions. A task is a single, 
specific action or step inside that play

**Can you have multiple plays in one playbook?**
yes

**What does become: true do at the play level vs the task level?**

When you put become: true at the play level, it acts as a global setting for that entire play

When you put become: true inside a specific task, it applies only to that single action.


**What happens if a task fails -- do remaining tasks still run?**

No, by default, if a task fails on a specific host, Ansible stops executing any remaining tasks for that host in the current play

--------------------------------------------------

## Task 3: Learn the Essential Modules


Practice each of these modules by writing a playbook called `essential-modules.yml` with multiple tasks:

1. **`yum`/`apt`** -- Install and remove packages:
```yaml
- name: Install multiple packages
  yum:
    name:
      - git
      - curl
      - wget
      - tree
    state: present
```

2. **`service`** -- Manage services:
```yaml
- name: Ensure Nginx is running
  service:
    name: nginx
    state: started
    enabled: true
```

3. **`copy`** -- Copy files from control node to managed nodes:
```yaml
- name: Copy config file
  copy:
    src: files/app.conf
    dest: /etc/app.conf
    owner: root
    group: root
    mode: '0644'
```

4. **`file`** -- Create directories and manage permissions:
```yaml
- name: Create application directory
  file:
    path: /opt/myapp
    state: directory
    owner: ec2-user
    mode: '0755'
```

5. **`command`** -- Run a command (no shell features):
```yaml
- name: Check disk space
  command: df -h
  register: disk_output

- name: Print disk space
  debug:
    var: disk_output.stdout_lines
```

6. **`shell`** -- Run a command with shell features (pipes, redirects):
```yaml
- name: Count running processes
  shell: ps aux | wc -l
  register: process_count

- name: Show process count
  debug:
    msg: "Total processes: {{ process_count.stdout }}"
```

7. **`lineinfile`** -- Add or modify a single line in a file:
```yaml
- name: Set timezone in environment
  lineinfile:
    path: /etc/environment
    line: 'TZ=Asia/Kolkata'
    create: true
```

Create a `files/` directory with a sample `app.conf` file for the copy task. Run the playbook against all servers.

**Document:** What is the difference between `command` and `shell`? When should you use each?

The primary difference is that the command module bypasses the system's shell (like bash or sh) and executes binaries directly, 
while the shell module runs commands inside a shell environment on the target host.

**You should use command for almost all basic commands. It is faster, more secure, and less prone to unpredictable environment issues**

**You should only use shell if your command relies on features built into the shell processor itself.**


## Task 4: Handlers -- Restart Services Only When Needed

Handlers are tasks that run only when triggered by a notify. This avoids unnecessary service restarts.

```

---                      #yaml docsument start for ansible
- name:  install and enable nginx service  #name of the play
  hosts: app                   #inventory or group of hosts 
  become: true                  # run tasks as root be sudo

  tasks:                       
    - name: update             #first task and its name
      shell: apt-get update     #module name
    - name: install nginx       
      apt:                      #module name 
        name: nginx             #module args
        state: present 

    - name: copy index page
      copy:
        src: /home/rohit/ansible/index.html
        dest: /var/www/html/index.html
      notify: restart nginx 


  handlers:
    - name: restart nginx 
      service:
        name: nginx
        state: restarted
```
**this handler will only run if the .html file will be updated. if it is not and task is in ok state and not changed the handler will not run.**

---

## Task 5: Dry Run, Diff, and Verbosity

Before running playbooks on production, always preview changes first.

1. **Dry run (check mode)** -- shows what would change without changing anything:
```bash
ansible-playbook install-nginx.yml --check
```

2. **Diff mode** -- shows the actual file differences:
```bash
ansible-playbook nginx-config.yml --check --diff
```

3. **Verbosity** -- increase output detail for debugging:
```bash
ansible-playbook install-nginx.yml -v       # verbose
ansible-playbook install-nginx.yml -vv      # more verbose
ansible-playbook install-nginx.yml -vvv     # connection debugging
```

4. **Limit to specific hosts:**
```bash
ansible-playbook install-nginx.yml --limit web-server
```

5. **List what would be affected without running:**
```bash
ansible-playbook install-nginx.yml --list-hosts
ansible-playbook install-nginx.yml --list-tasks
```

**Document:** Why is `--check --diff` the most important flag combination for production use?

The combination of --check and --diff is considered the absolute gold standard for production deployments because it provides complete visibility and safety.
Together, they allow you to perform a dry run that shows you exactly what changes will be made to your live environment before a single file or setting is actually modified.

---

## Task 6: Multiple Plays in One Playbook

```
---                      #yaml docsument start for ansible
- name:  install and enable nginx service  #name of the play
  hosts: app                   #inventory or group of hosts 
  become: true                  # run tasks as root be sudo

  tasks:                       
    - name: update             #first task and its name
      shell: apt-get update     #module name
    - name: install nginx       
      apt:                      #module name 
        name: nginx             #module args
        state: present 

    - name: copy index page
      copy:
        src: /home/rohit/ansible/index.html
        dest: /var/www/html/index.html
      notify: restart nginx 

    - name: enable nginx 
      service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: restart nginx 
      service:
        name: nginx
        state: restarted
    

- name: copy nginx.html in web servers
  hosts: web
  become: true

  tasks:
    - name: update server
      command: apt-get update
    - name: copy app.conf
      copy:
        src: /home/rohit/ansible/index.html
        dest: /home/ubuntu/
        owner: ubuntu 
        mode: '0777'
```


        


