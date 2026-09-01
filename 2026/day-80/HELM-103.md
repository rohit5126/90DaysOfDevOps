## Day 80 -- Helm Project: Multi-Environment Deployment and CI/CD

#### Task 1: Create Environment-Specific Values

**devboard-prod-values.yaml**
```
#statefulset
postgres:
  replicaCount: 1
  image:
    repository: postgres
    tags: "16-alpine"
  service:
    type: ClusterIP
    port: 5432
  volresources:
    requests:
      storage: 2Gi
  resources:
    requests:
      memory: "256Mi"
      cpu: "256m"
    limits:
      memory: "512Mi"
      cpu: "512m"
  storageClassName: gp2
#backend deployment
backend:
  replicaCount: 2
  image:
    repository: rohit5126/devboard-backend
    tags: "7feaccd"
  service:
    type: ClusterIP
    port: 8080
  resources:
    requests:
      memory: "256Mi"
      cpu: "256m"
    limits:
      memory: "512Mi"
      cpu: "512m"
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 4
    targetCPUUtilization: 70
#frontend deployment
frontend:
  replicaCount: 2
  image:
    repository: rohit5126/devboard-frontend
    tags: 7feaccd
  service:
    type: ClusterIP
    port: 4173
    lb: 80
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 500m
      memory: 256Mi
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 4
    targetCPUUtilization: 70
gateway:
  port: 80
  protocol: HTTP
```

**devboard-stage-values.yaml**
```
#statefulset
postgres:
  replicaCount: 1
  image:
    repository: postgres
    tags: "16-alpine"
  service:
    type: ClusterIP
    port: 5432
  volresources:
    requests:
      storage: 1Gi
  resources:
    requests:
      memory: "256Mi"
      cpu: "256m"
    limits:
      memory: "512Mi"
      cpu: "512m"
  storageClassName: gp2
#backend deployment
backend:
  replicaCount: 1
  image:
    repository: rohit5126/devboard-backend
    tags: "7feaccd"
  service:
    type: ClusterIP
    port: 8080
  resources:
    requests:
      memory: "256Mi"
      cpu: "256m"
    limits:
      memory: "512Mi"
      cpu: "512m"
  autoscaling:
    enabled: true
    minReplicas: 1
    maxReplicas: 2
    targetCPUUtilization: 70

#frontend deployment
frontend:
  replicaCount: 1
  image:
    repository: rohit5126/devboard-frontend
    tags: 7feaccd
  service:
    type: ClusterIP
    port: 4173
    lb: 80
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 500m
      memory: 256Mi
  autoscaling:
    enabled: true
    minReplicas: 1
    maxReplicas: 2
    targetCPUUtilization: 70

gateway:
  port: 80
  protocol: HTTP
```

**Depluy**

```
helm install devboard-stage devboard/ -f devboard-stage-values.yaml -n stage --create-namespace

# Staging (render to check)
helm template devboard-prod devboard/ -f devboard-prod-values.yaml

```


#### Task 2: Add Helm Hooks

**add this to your .yml fils in templates/ to deploy them in order.**
```
annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "0"
```

**How hooks work in the AI-BankApp context:**

helm.sh/hook: pre-install,pre-upgrade -- runs before install and before upgrade
This ensures MySQL is up before the BankApp Deployment is created
before-hook-creation -- deletes the old job before creating a new one on re-runs
Combined with init containers in the Deployment, this provides defense-in-depth

**Other useful hook types:**

post-install -- run database migrations after deploy
pre-delete -- backup database before teardown
test -- runs when you execute helm test

#### Task 3: Package and Version the Chart

```
# Lint first
helm lint bankapp/

# Package
helm package bankapp/
```

**Bump the version after changes: Edit bankapp/Chart.yaml:**

version: 0.2.0        # Chart structure changed (added hooks)
appVersion: "1.1.0"    # App version updated

Re-package:

`helm package bankapp/`

Now you have bankapp-0.1.0.tgz and bankapp-0.2.0.tgz

**Install from a package:**

helm install my-bankapp bankapp-0.2.0.tgz -f bankapp/values-dev.yaml -n bankapp --create-namespace


#### Task 4: Understand Helm in the AI-BankApp GitOps Pipeline

**With Helm, the pipeline becomes:**

Developer pushes code
  -> GitHub Actions builds Docker image
  -> Tags with git commit SHA
  -> Updates image.tag in helm-chart/values.yaml (or values-prod.yaml)
  -> Commits the change back to the repo
  -> ArgoCD detects the change and runs helm upgrade on EKS

CI step 

```
- name: update image tag in values.yaml
        env:
          tags: ${{ inputs.Tag }}

        run: |
          # Update backend image tag
          yq -i '.backend.image.tags = env(tags)' devboard/values.yaml
          
          # Update frontend image tag
          yq -i '.frontend.image.tags = env(tags)' devboard/values.yaml

      - name: Commit and Push Changes
        run: |
          # 1. Identify the virtual bot making the change
          git config --global user.name "github-actions[bot]"
          git config --global user.email "github-actions[bot]@users.noreply.github.com"
          
          # 2. Stage, commit, and push the modified deployment file
          git add devboard/values.yaml
          git commit -m "chore: update both frontend and backend image tags to latest code [skip ci]"
          git push

```
**What are the advantages of ArgoCD syncing a Helm chart vs raw manifests?**

ArgoCD syncing a Helm chart instead of raw manifests provides easier configuration management, cleaner Git repositories, and built-in dependency handling.

#### Task 5: Helm Best Practices for Production

1. Always use helm upgrade --install
   
2. Use helm diff before upgrading:
```
   helm plugin install https://github.com/databus23/helm-diff
helm diff upgrade bankapp bankapp/ -f bankapp/values-prod.yaml
```

3. Resource quotas per namespace:

```
# Add to templates/resourcequota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: {{ include "bankapp.fullname" . }}-quota
  namespace: {{ .Release.Namespace }}
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
```

4. Never store real secrets in values.yaml. In production, use:

External Secrets Operator with AWS Secrets Manager
Sealed Secrets
Vault by HashiCorp

#### Task 6: Clean Up and Review

**Check what you have deployed:**

helm list -A

```
helm uninstall devboard-stage -n stage
kubectl delete namespace stage
kind delete cluster --name rohit-cluster

```

