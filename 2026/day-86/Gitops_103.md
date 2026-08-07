# GitOps Project: End-to-End CI/CD Pipeline with AI-BankApp

**Complete gitops pipeline**

## Task 1: Study the AI-BankApp's GitOps CI Pipeline

On Main.yml run add below triggers so the pipeline only runs on chnages made inside required directories which can affect applictaions and also each time when the images is changed inside deployment it should not trigger a run.It only runs when application code changes (src/, pom.xml, Dockerfile) -- not when Kubernetes manifests change.

```
on:
  push:
    branches: [feat/gitops]
    paths:
      - 'src/**'
      - 'pom.xml'
      - 'Dockerfile'
  workflow_dispatch:

```

**add a gitops_bump.yml file in workflows which will update the images and push it to teh repo**

```
name: gitops-image-bump

on:
  workflow_call:
    inputs:
      Tag: { required: true, type: string }

jobs:
  bump:
    runs-on: ubuntu-latest
    permissions: 
      contents: write
    steps:
      - name: code checkout
        uses: actions/checkout@v6

      - name: 'Setup yq'
        uses: dcarbone/install-yq-action@v1
        with:
          version: "v4.44.3"

      - name: 'Check yq'
        run: |
          which yq
          yq --version

      - name: update deployment image
        env:
          Docker_user: ${{ vars.DOCKERHUB_USERNAME }}
          tags: ${{ inputs.Tag }}

        run: |
          yq -i '.spec.template.spec.containers[0].image = env(Docker_user) + "/bankapp:" + env(tags)' k8s/bankapp-deployment.yml

      - name: Commit and Push Changes
        run: |
          # 1. Identify the virtual bot making the change
          git config --global user.name "github-actions[bot]"
          git config --global user.email "github-actions[bot]@users.noreply.github.com"
          
          # 2. Stage, commit, and push the modified deployment file
          git add k8s/bankapp-deployment.yml
          git commit -m "chore: update bankapp image tag to latest code [skip ci]"
          git push


```
**Why [skip ci]? Without it, the commit that updates the manifest would trigger the pipeline again, which would update the manifest again -- an infinite loop. [skip ci] tells GitHub Actions to ignore this commit.**

**add this workflow in the main file and also add a generate tag job inside main to setup tag variable**

```
name: all-check-passed-pushed
on: 
  push:
    branches: [actions]
    paths:
      - 'src/**'
      - 'pom.xml'
      - 'Dockerfile'
      - 'Monitoring/**'
      - 'argoCD/**'
      - 'Helm/**'
  workflow_dispatch: 

permissions:
  contents: write

jobs:
  Code-Quality-Check:
    uses: ./.github/workflows/Code_quality.yml
    with:
      cache: 'maven' 

  Docker-Lint-Scan:
    needs: [Code-Quality-Check]
    uses: ./.github/workflows/docker-lint-scan.yml
    secrets: 
      DOCKERHUB_USERNAME: ${{ vars.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  Sonar-Quality-Scan:
    needs: [Code-Quality-Check]
    uses: ./.github/workflows/SonarScan.yml
    with:
      SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}
    secrets: 
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  Generate-tags:         #to generate tags
    needs: [Docker-Lint-Scan]
    runs-on: ubuntu-latest
    outputs:
      short-sha: ${{ steps.tag.outputs.sha_short }}
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
        
      - name: Set image tag
        id: tag
        run: echo "sha_short=$(git rev-parse --short HEAD)" >> "$GITHUB_OUTPUT"
      
  Docker-Build-Push:
    needs: [Code-Quality-Check, Generate-tags]
    uses: ./.github/workflows/docker-build-push.yml
    with:
      Tag: ${{ needs.Generate-tags.outputs.short-sha }}
    secrets: 
      DOCKERHUB_USERNAME: ${{ vars.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  ArgoCD-deployment:
    needs: [Docker-Build-Push, Generate-tags]
    uses: ./.github/workflows/gitops_bump.yml
    with:
      Tag: ${{ needs.Generate-tags.outputs.short-sha }}

```

<img width="1816" height="412" alt="image" src="https://github.com/user-attachments/assets/5b059642-8dcb-4dd0-8022-19c398e145a6" />

**How to deploy the app**

* Once this is done, get your EKS cluster ready and connect kubectl to your cluster.
* Then install argocd to your cluster using Helm.
* Then for the repo, and change dir to argoCD and run `kubectl apply -f root-app.yml`

**few things to make sure**

* that your EKS cluster has ebs storage configured and storgeclass is gp2
* also it is properly set to sync waves and resource ordering.

#### once applications are deployed you can Test Drift Detection and Recovery

GitOps means the cluster must always match Git. Test what happens when someone makes unauthorized changes.

**Scenario 1 -- Someone scales down the app directly:**
```
kubectl scale deployment bankapp -n bankapp --replicas=1
Check ArgoCD:

argocd app get bankapp
```
**Status should show OutOfSync. With selfHeal: true, ArgoCD will correct it within 3 minutes. Monitor:**

```
kubectl get pods -n bankapp -w
The replica count will return to 4 (or whatever the manifest specifies).
```

**Scenario 2 -- Someone updates the image tag directly:**
```
kubectl set image deployment/bankapp bankapp=nginx:latest -n bankapp
ArgoCD detects the drift and reverts it to the image tag from Git. The BankApp pods restart with the correct image.
```

**Scenario 3 -- Someone deletes a critical resource:**
```
kubectl delete service bankapp-service -n bankapp
ArgoCD recreates it from Git.
```

**View all drift events:**

```
argocd app history bankapp
```
In the ArgoCD UI, click the application and look at the "Events" tab. Every self-heal action is logged with the before/after state.

**Document: In each scenario, how long did ArgoCD take to detect and fix the drift? What would happen if selfHeal was disabled?**
By default it takes 3 minutes for argoCd to detect changes and re sync the app with the github repo. If self heal was disabled manual changes made directly to your cluster resources using kubectl will no longer be automatically reverted.


### Task 5: Reflect on the Complete DevOps Pipeline

S**tep back and look at everything you have built across the entire 90-day challenge that connects to this GitOps pipeline:**

```
[Developer writes code]
    |
[Git push to GitHub]  ........... Day 22-28: Git & GitHub
    |
[GitHub Actions CI]   ........... Day 40-49: GitHub Actions
    |-- Build with Maven
    |-- Run tests
    |-- Build Docker image  ..... Day 29-37: Docker
    |-- Push to DockerHub
    |-- Update K8s manifest
    |-- Commit back to Git
    |
[ArgoCD detects change] ........ Day 84-86: GitOps
    |
[ArgoCD syncs to EKS]  ........ Day 81-83: EKS
    |-- Rolling update
    |-- Health checks pass
    |-- HPA scales as needed ... Day 78-80: Helm (HPA, values)
    |
[Prometheus scrapes metrics] ... Day 73-77: Observability
    |-- Grafana dashboards
    |-- Alerts if something breaks
    |
[App is live with zero downtime]
```











