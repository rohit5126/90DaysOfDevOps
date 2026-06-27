# Advanced Triggers: PR Events, Cron Schedules & Event-Driven Pipelines

## Task 1: Pull Request Event Types

```
name:
on: 
  pull_request: 
    types: [opened, synchronize, reopened, closed]

jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - run: echo " ${{ github.event.action }}"
      - run: echo "${{ github.event.pull_request.title }}"
      - run: echo " ${{ github.event.pull_request.user.login }}"
      - name: branch
        run: |
          echo "Source branch is: ${{ github.head_ref }}"
          echo "Target branch is: ${{ github.base_ref }}"
      - name: merge
        if: ${{ github.event.pull_request.merged == true }}
        run: echo " PR merged"
```

## Task 2: PR Validation Workflow

```
name: pr_check
on: 
  workflow_dispatch: 
  pull_request:
    branches: [dev, main]

jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - name: code
        uses: actions/checkout@v7
        with:
          fetch-depth: 0
      - name: pr_check
        id: size_check
        uses: pelotech/github-action-file-size-checker@v0.2.1
        with:
          max_file_size_kib: '50'
  job2:
    runs-on: ubuntu-latest
    steps:
      - name: branch-check
        if: >-
          !(startsWith(github.head_ref, 'feature/') || 
          startsWith(github.head_ref, 'docs/') || 
          startsWith(github.head_ref, 'fix/'))
        run: |
          echo "invalid branch name"
          exit 1
        
  job3:
    runs-on: ubuntu-latest
    steps:
      - name: pr_body_check
        continue-on-error: true
        if: github.event.pull_request.body == ''
        run: echo "warning -PR description is empty"
```

# Task 3: Scheduled Workflows (Cron Deep Dive)

```
name: schedule
on:  
  schedule:
    - cron: '1 */1 * * *'
  workflow_dispatch: 
  push:
    branches: '**'
jobs: 
  health:
    env:
      url: ${{ 'https://google.com/' }}
    runs-on: ubuntu-latest
    steps:
      - name: curl
        run: |
          STATUS_CODE=$(curl --silent --output /dev/null --write-out "%{http_code}" $url)
          echo "HTTP Status Code: $STATUS_CODE"
          if [ $STATUS_CODE -ne 200 ]; then
            echo "wrong status code"
            exit 1
          fi
```

# Task 5: workflow_run — Chain Workflows Together

```

name: workflow-run
on:
  workflow_run:
    workflows: ['schedule']
    types: [completed]

jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - name: status
        if: ${{ github.event.workflow_run.conclusion == 'success' }}
        run: |
          echo " curl successful "
      - name: error
        if: failure()
        run: |
          echo " curl unsuccessful"
          exit 1
```

# Task 6: repository_dispatch — External Event Triggers

```
name: repo-dispatch
on: 
  repository_dispatch:
    types:
      - deploy-request

jobs:
 job1:
   runs-on: ubuntu-latest
   steps:
     - run: echo "${{ github.event.client_payload.environment }}"

```

