# Introduction to GitOps and ArgoCD

## Task 1: Understand GitOps

1. **what is gitops?**

Gitops is an operation framework which uses git as a single source of truth using declarative files for deployement in k8s. it uses argocd for deployment.
the full lifecycle is below:

code updated in github -> code quality checks and CI is done through github actions -> docker image is built and pushed to dockerhub or ECR -> CD workflow is triggered and app is deployed using argoCD on EKS.

EKS cluster is configured using terraform.

Its core components are declarative specifications, version control in Git, and automated reconciliation agents.

---

2. **GitOps vs traditional CI/CD:**

| Aspect | Traditional CI/CD | GitOps |
|--------|------------------|--------|
| Deployment trigger | CI pipeline runs `kubectl apply` | Git commit triggers sync |
| Source of truth | Pipeline scripts | Git repository |
| Drift detection | None | Continuous reconciliation |
| Rollback | Re-run pipeline or manual | `git revert` |
| Audit trail | Pipeline logs | Git history |
| Access control | Pipeline needs cluster credentials | Only ArgoCD has cluster access |
| Security | CI server has broad cluster access | Developers push to Git, never to the cluster |

---

3. **The AI-BankApp's GitOps flow:**
```
Developer pushes code to feat/gitops
         |
    [GitHub Actions CI]
    - Build Maven project
    - Run tests
    - Build Docker image
    - Push to DockerHub (tagged with git SHA)
    - Update image tag in k8s/bankapp-deployment.yml
    - Commit the change back to Git
         |
    [ArgoCD watches the repo]
    - Detects the new commit
    - Compares k8s/ manifests with live cluster
    - Syncs the change (rolling update)
    - BankApp pods restart with the new image
         |
    [Zero human intervention after git push]
```

4. **Four GitOps principles (from OpenGitOps):**

Declarative -- the desired state is expressed declaratively (Kubernetes YAML)
Versioned and immutable -- the desired state is stored in Git (versioned, auditable)
Pulled automatically -- agents (ArgoCD) pull the desired state, not pushed by CI
Continuously reconciled -- agents continuously compare desired vs actual and correct drift

---

## Task 2: Access ArgoCD on Your EKS Cluster

`helm install argocd argo/argo-cd --namespace argocd --create-namespace -f Helm/argocd/install_values.yml`

```
#install_values.yml

configs:
  params:
    server.insecure: true
#this is only done because we are not using secure protocol https for this app
```

**argoCD admin account password**
```
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

**port forwarding to access it in local**
```
kubectl port-forward svc/argocd-server -n argocd 8080:80 --address 0.0.0.0
```
---

**install argo cd cli**

```
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/

# Verify
argocd version --client
```
**connect argocd cli to your cluster**
```
argocd login --skip-test-tls --plaintext localhost:8080  #this is command
Username: admin
Password:

'admin:login' logged in successfully    #this is outut
Context 'localhost:8080' updated
```
---

## Task 3: AI-BankApp's ArgoCD Application Manifest

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: bankapp
  namespace: argocd  #namespace for argocd pod and service running
spec:
  project: default
  source:
    repoURL: https://github.com/rohit5126/AI-bankapp.git
    targetRevision: actions
    path: k8s    #path of k8s manifest files
  destination:
    server: https://kubernetes.default.svc   #cluster name in which argo CD is running or the cluster which in which you want to run the application
    namespace: newbankapp
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

```

---

## Task 4: Deploy the AI-BankApp via ArgoCD

```
kubectl apply -f applications.yml  

kubectl get all -n newbankapp

argocd app get bankapp
```

## Task 5: Explore ArgoCD's Live View

Click on the bankapp application in the ArgoCD UI. You will see:

**The resource tree:**

<img width="1906" height="986" alt="Screenshot From 2026-08-03 17-31-52" src="https://github.com/user-attachments/assets/6622d89c-e279-44c5-8036-e3123b772254" />

<img width="1906" height="986" alt="image" src="https://github.com/user-attachments/assets/8491c270-886b-4555-8b4d-89dddad73f08" />

you can click on any resource to see its details.

---

## Task 6: Test Self-Healing


**ArgoCD's selfHeal: true means it reverts any manual changes made directly to the cluster.**

```
kubectl scale deployment -n devboard-app backend-deployment --replicas=1

kubectl get all -n devboard-app

Within 3-5 minutes, ArgoCD detects the drift and scales it back to the value defined in Git (4 replicas, or whatever the HPA decides). Check the ArgoCD UI -- you will see a sync event.

```

**This is the core GitOps promise:** The cluster always matches Git. Manual changes do not survive. All changes must go through Git (pull requests, review, merge).
