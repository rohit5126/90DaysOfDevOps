# Your First GitHub Actions Workflow


## Task 1: Set Up

Create the folder structure: .github/workflows/
Create a file with .yml for setting up workflow file.
----------------------------------

## Task 2: Hello Workflow
```
name: Hello
on:
  workflow_dispatch:
jobs:
  code-checkout-hello:
    runs-on: ubuntu-latest
    steps:
      - name: code_checkout
        uses: actions/checkout@v7
      - name: hello
        run: echo "hello from github actions"
```
--------------------------------------

## Task 4: Add More Steps
Update hello.yml to also:

```
name: Hello
on:
  workflow_dispatch:
jobs:
  code-checkout-hello:
    runs-on: ubuntu-latest
    steps:
      - name: code_checkout
        uses: actions/checkout@v7
      - name: hello
        run: echo "hello from github actions"
      - name: date
        run: date
      - name: branch-name
        run:  echo " the branch is ${{ github.ref_name }}"
      - name: files-in-repo
        run: ls -l
      - name: host-os
        run: cat /etc/os-release

```
------------------------------------

<img width="1097" height="462" alt="image" src="https://github.com/user-attachments/assets/89cec0d0-2d68-4964-a192-2955ab11c8bb" />
