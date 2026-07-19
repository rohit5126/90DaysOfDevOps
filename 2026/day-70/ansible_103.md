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






variable precendence - host_vars > group_vars > playbook vars, and -e overrides everything.




