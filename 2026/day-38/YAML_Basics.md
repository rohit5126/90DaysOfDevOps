# YAML Basics

## Task 1: Key-Value Pairs

```
sudo apt-get install yamllint 

yamllint -v
```
```
person:
  name: Rohit Kumar
  role: software engineer
  exp: 4 years
  learning: True
```
## Task 2: Lists
```
person:
  name: Rohit Kumar
  role: software engineer
  exp: 4 years
  learning: True
  tools:
    - linux
    - python
    - github
    - docker
    - kubernetes
  hobbies: [ sports, bike, car, food ]
```

## Task 3: Multi-line Strings
<img width="895" height="500" alt="image" src="https://github.com/user-attachments/assets/2aab298e-1f51-466a-87e3-eb5bc07a4bf7" />

<img width="836" height="442" alt="image" src="https://github.com/user-attachments/assets/0fcac61a-15ed-462f-bc4d-d935133193d4" />


## Task 5: Validate Your YAML
```
---
person:
  name: Rohit Kumar
  role: software engineer
  exp: 4 years
  learning: "True"
  tools:
    - linux
    - python
    - github
    - docker
    - kubernetes
  hobbies: [sports, bike, car, food]
  cred:
    user: rkuma10
    pass: Calendar2839
server:
  ip: 192.45.87.86
  port: 5643
  script: |
    echo "this is script"
    if [ -f $path ]; then
      echo "true"
    fi
  add: >
    1176 sector 46
    gurgaon.near
    reefit gym
```


