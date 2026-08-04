# ArgoCD Deep Dive: Sync Strategies, Rollbacks, and Multi-App Management

## Task 1: Understand Sync Strategies

**Automated sync**

```
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

**Manual sync**

```
syncPolicy: {}
```
ArgoCD detects drift but does NOT auto-correct
A human must click "Sync" or run argocd app sync
Good for production where you want a review gate

**Try switching to manual sync:**

update the application file.
run `kubectl apply -f applications.yml`
make some changes in the repo and push it to github wait for few minutes and then refresh the application on argoCD,
you will see out of sync, you can see the difference in the resource which is changed.
inside resource click on sync to syncronize your app deployment.

---

## Task 2: Sync Waves and Resource Ordering

The AI-BankApp has dependencies: MySQL must be running before the BankApp starts. ArgoCD handles this with sync waves -- annotations that control the order of resource creation.

The sync order becomes:
```
Wave -2: Namespace, EBS          (infrastructure)
Wave -1:  ConfigMap, Secret       (configuration)
Wave  0: postgres statefulet      (databases)
Wave  1: backedn deployment       (backend application)
Wave  2: frontend deployment       (frontend application, gateway and networking policies)
```

ArgoCD processes each wave in order. Resources in the same wave sync in parallel. ArgoCD waits for each wave to be healthy before moving to the next.

Commit and push these changes. ArgoCD will re-sync and you will see the ordered deployment in the UI.

---

## Task 3: ArgoCD Rollbacks

**check the history**

```
argocd app history devboard
```

**roll back to previous version**

```
argocd app rollback devboard 0
{"level":"fatal","msg":"rpc error: code = FailedPrecondition desc = rollback cannot be initiated when auto-sync is enabled","time":"2026-08-04T07:28:11Z"}

#you need to stop autoSync.
```

**Important: Rollback is a temporary fix. It does not change Git. The proper GitOps rollback is:**
```
# In your fork
git revert HEAD
git push
```

**Document: What is the difference between ArgoCD rollback and git revert? Which is the GitOps-correct approach?**

the difference is rollback is a temporary fix and requires autosync to be disabled, where as git revert is permanent fix and supports autosync. also git revert is correct gitops approach as git is only source of truth.

---

## Task 4: App of Apps Pattern

with this pattern you can manage multiple applications with one root .yml file, 
```
folder structure-
-root.yml
-devbaord.yml
-bankapp.yml
-monitoring.yml
```

**IMP**
```
#add this line in all the other apps yml file inside metadata. to make the child application

  finalizers:
    - resources-finalizer.argocd.argoproj.io

#do not addin root application
```

**note: Make sure your chnaged are psuhed to github repo as argocd sync using github repo**

`kubectl apply -f root.yml`

all your 4 applications will be deployed.


---

## Task 5: ArgoCD Notifications

