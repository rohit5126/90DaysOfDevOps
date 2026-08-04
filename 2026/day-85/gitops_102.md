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

## 
