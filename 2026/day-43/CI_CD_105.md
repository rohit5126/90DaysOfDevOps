# Jobs, Steps, Env Vars & Conditionals

## Task 1: Multi-Job Workflow

```

name: multijob
on:
  push: 
    branches: master
    paths: .github/workflows/multijob.yml

jobs:
  code:
    runs-on: [self-hosted, linux-practice]
    steps:
      - name: code checkout
        uses: actions/checkout@v7
  build:
    runs-on: [self-hosted, linux-practice]
    needs: code
    steps:
      - name: build
        run: echo "build successful"
  test:
    runs-on: [self-hosted, linux-practice]
    needs: build
    steps:
      - name: test
        run: echo "test completed, all test passed"
  deploy:
    needs: test
    runs-on: [self-hosted, linux-practice]
    steps:
      - name: deploy
        run: echo "ready to deploy"
```

<img width="1885" height="501" alt="image" src="https://github.com/user-attachments/assets/0bf1c3f3-0541-4ce7-bbf4-488ad125eebe" />

---------------------------------------

## Task 2: Environment Variables

```
name: envvar
on:
  workflow_dispatch: 
env: #this is workflow level variable
  appname: myapp
jobs:
  code:
    runs-on: ubuntu-latest
    env: #this is job level variable
      environment: staging
    steps:
      - name: code-checkout
        uses: actions/checkout@v7
      - name: staging area
        run: echo " this is $environment area"
  test:
    runs-on: ubuntu-latest
    steps:
      - name: testing
        env:   #this is step level variable
          version: 1.0.0
        run: echo "this is testing for $appname $version"
      - name: details
        run: echo "this is sha fo the commit ${{ github.sha }}"  #github context variables
      - name: owner
        run: echo "this is owner ${{ github.repository_owner }}"
```

-----------------------------------------

## Task 3: Job Outputs

```
name: joboutput

on:
  workflow_dispatch: 

jobs:
  job1:
    runs-on: ubuntu-latest
    outputs:
      output1: ${{ steps.step2.outputs.files }}
      output2: ${{ steps.step3.outputs.test }}
      output3: ${{ steps.step4.outputs.test }}
    steps:
      - name: code
        uses: actions/checkout@v7
      - id: step2
        run: |
          {
             echo "files<<EOF"
             ls -l
             echo "EOF" 
          }  >> "$GITHUB_OUTPUT"
      - id: step3
        run: echo "test=world" >> "$GITHUB_OUTPUT"
      - id: step4
        run: |
          text=$(date)
          echo "test=$text" >> "$GITHUB_OUTPUT"
  job2:
    runs-on: ubuntu-slim
    needs: job1
    env:
      ro: ${{ needs.job1.outputs.output1 }}
      ar: ${{ needs.job1.outputs.output2 }}
      da: ${{ needs.job1.outputs.output3 }}

    steps:
     - name: print
       run: echo " $ro "
     - name: date
       run: echo "$da"

```
