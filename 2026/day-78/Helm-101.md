# Introduction to Helm and Chart Basics

## Task 1: Understand Helm Concepts

**What is Helm?**

A Helm chart is a package of pre-configured Kubernetes resources. Acting like a "recipe" or "installer", it bundles together all the YAML files, templates, and configurations needed to deploy complex applications to a Kubernetes cluster with a single command

**Core concepts:**

Chart -- a collection of files that describe a set of Kubernetes resources (Deployment + Service + ConfigMap + Secret = one chart)
Release -- a running instance of a chart in a cluster. You can install the same chart multiple times with different release names
Repository -- a place where charts are stored and shared (like DockerHub for images)
Values -- configuration that customizes a chart for each deployment (replicas, image tag, resource limits)

**Why Helm over raw manifests?**

1. Dynamic Configuration vs. Hardcoded Values
2.Single-Command Lifecycle Management
3. Native Rollbacks and History
4. Automated Dependency Handling

## Helm Commands

```

 1536  helm package nginx-chart

 1544  helm install mysql ./nginx-chart
 


 1560  helm history mysql 

 1562  helm rollback mysql 2

 1572  helm show values ./nginx-chart
 1573  helm repo add bitnami https://charts.bitnami.com/bitnami

 1581  helm repo update

 1594  helm search repo bitnami/mysql

 1602  helm install bankapp-mysql

 1611  helm upgrade mysql nginx-chart -f current_values.yml 
 
 1615  helm upgrade mysql nginx-chart --set env.MYSQL_ROOT_PASSWORD=test --set env.MYSQL_DATABASE=mydb
 1618  kubectl get all
 1619  kubectl exec -it mysql-nginx-chart-dcf9c4f5b-2qsv5 -- sh
 1622  helm get values mysql > current_values.yml
 1623  helm uninstall mysql 
```
