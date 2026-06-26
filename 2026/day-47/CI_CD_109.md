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
