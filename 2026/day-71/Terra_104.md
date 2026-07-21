# Roles, Galaxy, Templates and Vault

Your playbooks are getting bigger. Tasks, variables, handlers, files -- all living in one YAML file that grows longer every day. In real projects, you manage dozens of servers with different roles -- web servers, databases, monitoring agents, load balancers. You need a way to organize, reuse, and share automation.

Today you learn Ansible Roles (the standard way to structure automation), Jinja2 Templates (dynamic config files), Ansible Galaxy (the community marketplace), and Ansible Vault (secrets management).

## Task 1: Jinja2 Templates

```
# Managed by Ansible -- do not edit manually
server {
    listen {{ http_port | default(80) }};
    server_name {{ ansible_hostname }};

    root /var/www/{{ app_name }};
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/{{ app_name }}_access.log;
    error_log /var/log/nginx/{{ app_name }}_error.log;
}

```

playbook.yml

```
---
- name: Nginx installation and setup
  hosts: web
  become: true

  tasks:
    - name: nginx installation
      apt:
        name: nginx
        state: present 
    
    - name: deploy template
      template:
        src: templates/nginx.conf.j2
        dest: "/etc/nginx/conf.d/{{ app_name }}.conf"
        owner: "root"
        mode: "0644"
      notify: Restart nginx
    
    - name: Deploy index page
      copy:
        content: "<h1>{{ app_name }}</h1><p>Host: {{ ansible_hostname }} | IP: {{ ansible_default_ipv4.address }}</p>"
        dest: "/var/www/{{ app_name }}/index.html"

  handlers:
    - name: Restart nginx
      services:
        name: nginx
        state: restarted

```

## Task 2: Understand the Role Structure

**An Ansible role has a fixed directory structure. Each directory has a specific purpose:**

```
roles/
  webserver/
    tasks/
      main.yml         # The main task list
    handlers/
      main.yml         # Handlers (restart services, etc.)
    templates/
      nginx.conf.j2    # Jinja2 templates
    files/
      index.html       # Static files to copy
    vars/
      main.yml         # Role variables (high priority)
    defaults/
      main.yml         # Default variables (low priority, easily overridden)
    meta/
      main.yml         # Role metadata and dependencies
```

**generte a skeleton dir**

`ansible-galaxy init roles/webserver`

**What is the difference between vars/main.yml and defaults/main.yml?**

In an Ansible role, both defaults/main.yml and vars/main.yml are used to define variables, but they differ significantly in variable precedence (which variable wins when names conflict).

---

## Task 3: Build a Custom Webserver Role

It is like a terraform module. we ceate tasks , handlers, variables, templates seperately inside web role. while ruuning the playbook we assign this role to the particular host we want.

```
- name: configure nginx server
  hosts: web
  become: true
  roles:
    - role: web
```

so the role is created as a module and it is ready to run in any host. it is not fixed for a particular host.


## Task 4: Ansible Galaxy -- Use Community Roles

Ansible Galaxy is a marketplace of pre-built roles.

**Search for roles:**

ansible-galaxy search nginx --platforms EL
ansible-galaxy search mysql

**Install a role from Galaxy:**

ansible-galaxy install geerlingguy.docker

**Check where it was installed:**

ansible-galaxy list

**Use the installed role -- create docker-setup.yml:**

```
- name: install dokcer
  hosts: web
  become: true
  roles:
    - role: geerlingguy.docker
```

**Use a requirements file for managing multiple roles. Create requirements.yml:**

```
---
roles:
  - name: geerlingguy.docker
    version: "7.4.1"
  - name: geerlingguy.ntp
```

`ansible-galaxy install -r requirements.yml`

**Using a requirements.yml file shifts your workflow from manual, error-prone commands to automated infrastructure as code.**


## Task 5: Ansible Vault -- Encrypt Secrets

`Never put passwords, API keys, or tokens in plain text. Ansible Vault encrypts sensitive data.

**Create an encrypted file:**

`ansible-vault create group_vars/db/vault.yml`

It will ask for a vault password, then open an editor. Add:

vault_db_password: SuperSecretP@ssw0rd
vault_db_root_password: R00tP@ssw0rd123
vault_api_key: sk-abc123xyz789

Save and exit. Open the file with cat -- it is fully encrypted.

<img width="832" height="270" alt="image" src="https://github.com/user-attachments/assets/282f6a38-06d3-454c-8837-e05d961e0e54" />


**Edit an encrypted file:**
`ansible-vault edit group_vars/db/vault.yml`

**View without editing:**
`ansible-vault view group_vars/db/vault.yml`

**Encrypt an existing file:**
`ansible-vault encrypt group_vars/db/secrets.yml`

**to run playbook with password**
ansible-playbook db-setup.yml --ask-vault-pass

**Use a password file (better for CI/CD):**
```
echo "YourVaultPassword" > .vault_pass
chmod 600 .vault_pass
echo ".vault_pass" >> .gitignore
```

ansible-playbook db-setup.yml --vault-password-file .vault_pass

**Or set it in ansible.cfg:**

[defaults]
vault_password_file = .vault_pass


--ask-vault-pass: Requires an interactive TTY session to accept a keyboard-typed password.
--vault-password-file: Directs Ansible to read the password directly from a file or script. This allows your pipeline to run completely hands-free at any time of day.

---



