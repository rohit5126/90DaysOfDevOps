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

helm 
