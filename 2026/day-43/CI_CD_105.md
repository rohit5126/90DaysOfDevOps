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
      output1: ${{ steps.step2.outputs.files }}    #this will always be inside a job
      output2: ${{ steps.step3.outputs.test }}
      output3: ${{ steps.step4.outputs.test }}
    steps:
      - name: code
        uses: actions/checkout@v7
      - id: step2
        run: |
          {                            #ths syntax is used for multi line output.
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
    needs: job1      #important line must be there or the env variable will not accept the output.
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
-------------------------------------------------
```
name: sfhosted
on:
  push:
    branches: master
    paths: .github/workflows/self_hosted.yml
  workflow_dispatch:

jobs:
  job1:
    outputs:
      out1: ${{ steps.step1.outputs.text }}
      out2: ${{ steps.step2.outputs.text }}
      out3: ${{ steps.step3.outputs.text }}
      
    runs-on: [self-hosted, linux-practice]
    strategy:
      fail-fast: false
    steps:
      - id: step1
        run: |
          version=$(docker --version)     #this syntax is always used for oneline output.
          echo "text=$version" >> "$GITHUB_OUTPUT"
      - id: step2
        run: |
          version=$(whoami)
          echo "text=$version" >> "$GITHUB_OUTPUT"
      - id: step3
        run: |
          version=$(hostname)
          echo "text=$version" >> "$GITHUB_OUTPUT"
   
  job2:
    runs-on: ubuntu-latest
    needs: job1
    env:
      d: ${{ needs.job1.outputs.out1 }}
      w: ${{ needs.job1.outputs.out2 }}
      h: ${{ needs.job1.outputs.out3 }}
    steps:
      - name: docker
        run: echo "$d"
      - name: whoami
        run: echo "$w"
      - name: hostname
        run: echo "$h"
```
-------------------------------------------------------------

## Task 4: Conditionals

### In GitHub Actions, you use the if conditional to programmatically control whether a job or step should run

```
name: condition
on:
  push:
    branches: master
    paths: .github/workflows/conditional.yml
  workflow_dispatch:

jobs:
  job1:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
    steps:
      - name: step1
        if: ${{ github.ref_name == 'master' }}  #you can find this variables in github context official doc
        uses: actions/checkout@v7
      - id: step2
        if: ${{ github.event_name != 'workflow_dispatch' }}
        run: exit 1
      - name: step3
        if: failure()
        run: echo "last step failed"
      - name: step4
        if: ${{ github.event_name == 'push' || success() }}  #successs or faiure depends on status of previous step.
        run: echo "correct event"

```
----------------------------------------------------------------

## Task 5: Putting It Together

**make sure this script exist in all the branches fr it to work in both branch**

```
name: sp
on:
  push:  
    branches: 
      - '**'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: lintjob
        run: echo "lint job running"
  test:
    runs-on: ubuntu-latest
    steps:
      - name: testjob
        run: echo "test job is running"
  summary:
    runs-on: ubuntu-latest
    needs: [ lint,test ]
    steps:
      - name: branch
        if: ${{ github.ref_name == 'master' }}
        run: echo "this is master branch"
      - name: feature
        if: ${{ github.ref_name == 'dev' }}
        run: echo "this dev branch"
      - name: commit
        run: echo " ${{ github.event.head_commit.message }}"
```
