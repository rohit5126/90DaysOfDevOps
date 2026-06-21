# What is CI/CD?

## Task 1: The Problem

### Think about a team of 5 developers all pushing code to the same repo manually deploying to production.

Write in your notes:

#### What can go wrong?

`A 5-developer team pushing directly to the same repo and manually deploying causes severe configuration drift,
race conditions/overwrite conflicts, and untested code releases that expose users to downtime, while lacking an audit trail to trace failures`

#### What does "it works on my machine" mean and why is it a real problem?

`"It works on my machine" is a classic software phrase. It means code runs perfectly on a developer's local computer, 
but fails completely when deployed to production or shared with colleagues`

**Why It Is a Real ProblemWastes Engineering Time:**
1. Teams spend hours debugging environmental differences instead of writing new features.Creates Blind Spots: 
2. Developers stop trusting bug reports because they cannot recreate the failure locally.Delays Release Cycles: 
3. Testing halts because QA engineers cannot get the app to run on their machines.Causes Production Outages: 
4. Flawed code bypasses local checks and breaks live systems for paying users.


#### How many times a day can a team safely deploy manually?

`Zero times a day is the only truly safe number for manual deployments. 
Manual deployments are never inherently safe because they rely on human memory and lack automated safety guardrails`

## Task 2: CI vs CD

### Continuous Integration -
`Continuous Integration (CI) is the practice of automating the integration of code changes from multiple developers into a single software project. 
It is the direct solution to the "it works on my machine" problem and the risks of manual deployment.`

<img width="777" height="377" alt="image" src="https://github.com/user-attachments/assets/ac7233e1-46a1-4838-875c-f63e64da7fae" />

### Continuous Delivery — 
`Continuous Delivery (CD) is the practice of automatically preparing code changes for a release to production. It picks up exactly where Continuous Integration (CI) leaves off, 
ensuring that every change that passes automated testing is ready to be deployed at the click of a button`

<img width="787" height="382" alt="image" src="https://github.com/user-attachments/assets/5e0c9880-9278-4e9a-b848-bcd882e5e416" />

### Continuous Deployment — 
`Continuous Deployment (CD) is the final stage of pipeline automation where every code change that passes all testing phases is 
launched directly into production automatically, without any human intervention`

<img width="786" height="207" alt="image" src="https://github.com/user-attachments/assets/ea3358d3-b2ce-4e0f-ab50-d8d5883ce4b0" />

## Task 3: Pipeline Anatomy

**Trigger:** `An event—like a code push, a pull request, or a cron schedule—that automatically starts the execution of a pipeline.`

**Stage:** `A high-level, logical division in the pipeline used to group related work sequentially, such as separate phases for compiling, testing, or deploying code.`

**Job:** `A specific collection of tasks executed together on the same machine, running concurrently with other jobs in the same stage by default.`

**Step:** `The smallest execution unit inside a job, representing a single sequential command, script, or pre-packaged task.`

**Runner:** `The physical machine, virtual machine, or isolated container designated to connect to the orchestrator and actually execute the jobs.`

**Artifact:** `A file or set of compiled packages produced by a job that is saved and passed along to subsequent stages or human reviewers`

## Task 4: Draw a Pipeline
```
┌──────────────────────────────────────────────────────────┐
│                      TRIGGER                             │
│  Developer pushes code / creates a Pull Request          │
└──────────────────────────┬───────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────┐
│              STAGE 1: CODE VERIFICATION                  │
│  ┌────────────────────────────────────────────────────┐  │
│  │  JOB: Code Quality & Testing (Runs on Runner A)    │  │
│  │  ├─ Step 1: Checkout repository code               │  │
│  │  ├─ Step 2: Install language dependencies          │  │
│  │  └─ Step 3: Execute Unit & Integration Tests      │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────┬───────────────────────────────┘
                           │
                           ▼ (Passes tests successfully)
┌──────────────────────────────────────────────────────────┐
│              STAGE 2: BUILD PACKAGE                      │
│  ┌────────────────────────────────────────────────────┐  │
│  │  JOB: Docker Containerization (Runs on Runner B)   │  │
│  │  ├─ Step 1: Authenticate with Docker Registry     │  │
│  │  ├─ Step 2: Build image from project Dockerfile   │  │
│  │  └─ Step 3: Push image as an ARTIFACT to Registry  │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────┬───────────────────────────────┘
                           │
                           ▼ (Image successfully pushed)
┌──────────────────────────────────────────────────────────┐
│              STAGE 3: DEPLOYMENT                         │
│  ┌────────────────────────────────────────────────────┐  │
│  │  JOB: Push to Environment (Runs on Runner C)       │  │
│  │  ├─ Step 1: Securely connect to Staging Server     │  │
│  │  ├─ Step 2: Pull down the newly built Docker image │  │
│  │  └─ Step 3: Stop old container & start the new one │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘

```
