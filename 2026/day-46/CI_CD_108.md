# Reusable Workflows & Composite Actions

## Task 1: Understand workflow_call

**In GitHub Actions, workflow_call is a trigger that allows you to create reusable workflows. By defining on: workflow_call in a YAML file, 
you can centrally manage standard CI/CD logic and invoke it from multiple other workflows or repositories,
eliminating duplicate code and simplifying security governance**

<img width="1185" height="545" alt="image" src="https://github.com/user-attachments/assets/cf66c413-4b3b-45c9-a9cb-4a338e5687c7" />


**How it WorksThe**

**Callee (Reusable Workflow)**: `Must include on: workflow_call at the top of the file. You can define specific inputs (for non-sensitive data) 
and secrets (for passwords and credentials) that the workflow will accept.`

**The Caller (Calling Workflow)**: `Uses the uses keyword within a job to invoke the reusable workflow's file path`

**Where must a reusable workflow file live?:**
`.github/workflows/`

How is calling a reusable workflow different from using a regular action (uses:)?

----------------------------------------------------------------

## Task 2: Create Your First Reusable Workflow

```
name: RE
on:
  workflow_call:
    outputs:
      data:
        description: "version id "
        value: ${{ jobs.job1.outputs.out1 }}
    inputs:
      env:
        type: string
        default: 'staging'
        required: true
      app_name:
        type: string
        required: false
    secrets:
      token:
        required: true
  
jobs:
  job1:
    outputs:
      out1: ${{ steps.v1.outputs.version }}
    runs-on: ubuntu-latest
    steps:
      - name: code
        uses: actions/checkout@v7
      - name: print
        run: echo " building ${{ inputs.app_name }} for ${{ inputs.env }}"
      - name: token
        run: echo "dokcer token is set ${{ secrets.token }}"
      - id: v1
        run: |
          echo "version=v1.0.app" >> "$GITHUB_OUTPUT"
        
```

-------------------------------------------

## Task 3: Create a Caller Workflow and =Add Outputs to the Reusable Workflow

```
name: CB
on: 
  workflow_dispatch:
jobs:
  build:
    uses: ./.github/workflows/resuable-workflow.yml
    with:
      env: prod
      app_name: devboard
    secrets: 
      token: ${{ secrets.DOCKER_TOKEN }}
  job2:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "${{ needs.build.outputs.data }}"
```

<img width="1302" height="582" alt="image" src="https://github.com/user-attachments/assets/39158418-7151-4748-8d2f-3fae22861c7a" />


 **Does the second job print the version from the reusable workflow?**
`yes it does print`


------------------------------------------------------

## Task 5: Create a Composite Action

action.yml
```
name: go-setup-test
description: 'this action file runs go setup, test the code and lint it'

inputs:
  go-version:
    default: '1.23'
  go-version-file: 
    required: true
    type: string
  cache-dependency-path: 
    required: true
    type: string
  code-path: 
    required: true
    type: string

outputs:
  out:
    value: ${{ steps.status.outputs.pass }}

runs:
  using: 'composite'
  steps:
   - name: go-steup
     uses: actions/setup-go@v6
     with:
       go-version: ${{ inputs.go-version }}
       go-version-file: ${{ inputs.go-version-file }}
       cache-dependency-path: ${{ inputs.cache-dependency-path }}
   - name: run go-format
     shell: bash
     run: go fmt main.go
     working-directory: ${{ inputs.code-path }}

   - name: run go-vet
     shell: bash
     run: go vet main.go
     working-directory: ${{ inputs.code-path }}

   - name: testing
     shell: bash
     run: go test
     working-directory: ${{ inputs.code-path }}

   - id: status
     shell: bash
     if: success()
     run: |
       echo "pass='all test cases passed'" >> "$GITHUB_OUTPUT"
```
Workflow file

```
backend:
    runs-on: ubuntu-latest
    steps:
      - name: code-checkout
        uses: actions/checkout@v7
      - name: go-setup-test
        id: setup
        uses: ./.github/actions/go-setup/
        with:
          go-version: 1.23
          go-version-file: backend/go.mod
          cache-dependency-path: backend/go.sum
          code-path: backend/
      - name: status
        run: |
          echo " ${{ steps.setup.outputs.out }}"
        
      - name: docker login
        uses: docker/login-action@v4
        with:
          username: ${{ vars.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: docker build and push
        if: ${{ github.ref_name == 'master' }}
        uses: docker/build-push-action@v7
        with:
          context: backend/
          push: true
          tags: ${{ vars.DOCKER_USERNAME }}/devboard-backend:latest
```

<img width="1270" height="597" alt="image" src="https://github.com/user-attachments/assets/64b010bc-a95c-4c53-88cd-73628e4a48a2" />

<img width="362" height="745" alt="image" src="https://github.com/user-attachments/assets/0aba7d43-5ac3-4a34-a92e-63ea4e9b2908" />

**it must be named either action.yml or action.yaml if you want GitHub to recognize it automatically**

**You can place the folder containing your action.yml file anywhere in your repository. GitHub only cares that the file itself is named action.yml and that you point to its containing folder**
