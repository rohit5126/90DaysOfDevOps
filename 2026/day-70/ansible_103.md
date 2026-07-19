# Variables, Facts, Conditionals and Loops

Your playbooks work, but they are static -- same packages, same config, same behavior on every server. Real infrastructure is not like that.
Web servers need Nginx, app servers need Node.js, production gets more memory than dev. Today you make your playbooks smart.

Variables, facts, conditionals, and loops turn a rigid script into flexible automation that adapts to each host, each group, and each environment.


## Task 1: Variables in Playbooks

```
---
- name: "variable demo file"
  hosts: app
  become: true

  vars:
    app_name: "my-app"
    app_port: "8080"
    app_dir: "/opt"
    packages: 
      - unzip
      - nginx
      - wget

  tasks:
    - name: update
      shell: "apt-get update"
    - name: print
      debug:
        msg: " deploying {{ app_name }} on port {{ app_port }}"
    - name: app dir update
      copy:
        src: ./index.html
        dest: "{{ app_dir }}"
        owner: ubuntu
        mode: "0750"

    - name: install required package
      apt: 
        name: "{{ packages }}"
        state: present 
```
**Now, override a variable from the command line:**

ansible-playbook variables-demo.yml -e "app_name=my-custom-app app_port=9090"

**Verify: Does the CLI variable override the playbook variable?**

yes it does.

---

## Task 2: group_vars and host_vars

demo.yml
```
---
- name: "variable demo file"
  hosts: application
  become: true

  tasks:
    - name: update
      shell: "apt-get update"
    - name: print
      debug:
        msg: " deploying i env {{ app_env }} on port {{ http_port }}"

    - name: install required package
      apt: 
        name: "{{ common_packages }}"
        state: present
```

<img width="342" height="369" alt="image" src="https://github.com/user-attachments/assets/56f33094-0157-4ced-a0eb-0ba46b6a4573" />

<img width="361" height="376" alt="image" src="https://github.com/user-attachments/assets/98de3a3e-6c64-46e9-a1be-9008da3d2187" />

<img width="1520" height="446" alt="Screenshot From 2026-07-19 13-55-20" src="https://github.com/user-attachments/assets/b05aac1c-6d06-4b3c-98fe-eb59f13c11b4" />

<img width="1380" height="446" alt="Screenshot From 2026-07-19 13-55-42" src="https://github.com/user-attachments/assets/328602e8-e04a-4fc7-8797-705166ac79f9" />



variable precendence - host_vars > group_vars > playbook vars, and -e overrides everything.

---

## Task 3: Ansible Facts -- Gathering System Information

```
---
- name: Facts demo
  hosts: app
  tasks:
    - name: Show OS info
      debug:
        msg: >
          Hostname: {{ ansible_facts['hostname'] }},
          OS: {{ ansible_facts['distribution'] }} {{ ansible_facts['distribution_version'] }},
          RAM: {{ ansible_facts['memtotal_mb'] }}MB,
          IP: {{ ansible_facts['default_ipv4']['address'] }}

    - name: Show all network interfaces
      debug:
        var: ansible_facts['interfaces']

```

---

## Task 4: Conditionals with when

```
---
- name: Conditional tasks demo
  hosts: application
  become: true

  tasks:
    - name: Install Nginx (only on web servers)
      apt:
        name: nginx
        state: present
      when: "'web' in group_names"

    - name: Install MySQL (only on db servers)
      apt:
        name: mysql-server
        state: present
      when: "'db' in group_names"

    - name: Show warning on low memory hosts
      debug:
        msg: "WARNING: This host has less than 1GB RAM"
      when: ansible_memtotal_mb < 1024

    - name: Run only on Amazon Linux
      debug:
        msg: "This is an Amazon Linux machine"
      when: ansible_distribution == "Amazon"

    - name: Run only on Ubuntu
      debug:
        msg: "This is an Ubuntu machine"
      when: ansible_distribution == "Ubuntu"

    - name: Run only in production
      debug:
        msg: "Production settings applied"
      when: app_env == "prod"

```
Verify: Are tasks correctly skipping on hosts that don't match the condition? `yes`

## Task 5: Loops

```
---
- name: money heist
  hosts: app
  become: true

  vars_files: 
    - vars/variable.yml

  tasks:
    - name: create user and group
      user:
        name: "{{ item.name }}"
        group: "{{ item.group }}"
        state: present
      loop: "{{ users }}"

    - name: install packages
      package:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"
```

**What is the difference between loop and the older with_items**

The primary difference is that loop is the modern, native keyword for looping over lists in Ansible, while with_items is an older, specialized lookup plugin

---

## Register, Debug, and Combine Everything

**Server Report **

```
---
- name: server report generation file
  hosts: application
  become: true

  tasks:
    - name: disk usage
      shell: df -h / | grep /dev/root
      register: disk_usage
    
    - name: memory usage
      command: free -h 
      register: memory_usage

    - name: List of running services
      shell: systemctl list-units --state=running --type=service
      register: list_service

    - name: Generate report
      debug:
        msg: 
          - "====================Server Report================================"
          - "Hostname: {{ ansible_facts['hostname'] }}"
          - "OS: {{ ansible_facts['distribution'] }} {{ ansible_facts['distribution_version'] }}"
          - "RAM: {{ ansible_facts['memtotal_mb'] }}MB USED: {{ memory_usage.stdout }}"
          - "IP: {{ ansible_facts['default_ipv4']['address'] }}"
          - "DISK: {{ disk_usage.stdout }}"
          - "Running services: {{ list_service.stdout_lines | length }}"
    - name: Disk critical
      debug:
        msg: "Disk is critically low"
      when: "'9[0-9]%' in disk_usage.stdout or '100%' in disk_usage.stdout "

    - name: create a report file
      copy:
        content: |
          Hostname: {{ ansible_facts['hostname'] }},
          OS: {{ ansible_facts['distribution'] }} {{ ansible_facts['distribution_version'] }},
          RAM: {{ ansible_facts['memtotal_mb'] }}MB USED: {{ memory_usage.stdout }},
          IP: {{ ansible_facts['default_ipv4']['address'] }},
          DISK: {{ disk_usage.stdout }},
          Checked at: {{ ansible_facts.date_time.iso8601 }}
        dest: /home/ubuntu/report-{{ inventory_hostname }}.txt

```












