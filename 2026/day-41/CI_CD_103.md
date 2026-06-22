# Triggers & Matrix Builds

## Task 1: Trigger on Pull Request

```
name: Trigger
on:
  pull_request:
    branches: master
jobs:
  first:
    runs-on: ubuntu-latest
    steps:
      - name: PR
        run: echo "PR check running for branch ${{ github.ref_name }}"

```

### Verify: Does it show up on the PR page?

<img width="1242" height="425" alt="image" src="https://github.com/user-attachments/assets/2eb5a3cd-e6fb-4a3f-b44e-a81b7c718e9a" />

--------------------------------------------

## Task 2: Scheduled Trigger

```
name: trigger2
on:
  schedule:
    - cron: '0 9 * * 1'
  workflow_dispatch: 
jobs:
  second:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v4
        
      - name: t2
        run: echo "scheduled job for the pull request"
```
