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
