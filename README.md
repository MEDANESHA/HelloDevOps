# Project Documentation

## Introduction

This project involves the creation of a Hello World application using Node.js, along with the implementation of a Kubernetes (K8s) cluster through Infrastructure as Code (IaC) using Terraform. Network resources, including an Azure Container Registry, are provisioned to manage network access to the K8s cluster. The setup also includes a node pool distributed across different availability zones, with auto-scaling enabled to increase availability and scalability.

## Continuous Integration (CI)

### Jenkins Pipeline

A Jenkins pipeline has been established to automate the process of checking commits, building Docker images, and pushing the built images to the Azure Container Registry, also to commit the project new image version to the 'deploy-k8s' repository, which contains K8s manifest files. This action will trigger ArgoCD to synchronize with the K8s architecture.

## Continuous Deployment (CD)

### ArgoCD Setup

1. **Namespace Creation**: A K8s namespace for ArgoCD has been created.
2. **Configuration Installation**: Necessary configurations for ArgoCD have been installed within the namespace.
3. **Integration with GitHub**: ArgoCD is configured to integrate with the GitHub repository (`deploy-k8s`), housing K8s manifest files.

### K8s Manifests

1. **Resource Creation**: Pod, Service, Deployment, and Horizontal Pod AutoScaler resources have been created.
2. **AutoScaling Configuration**: The Horizontal Pod AutoScaler is configured to scale pod replications based on CPU usage.

## Monitoring

### Prometheus

1. **Installation**: Prometheus has been installed using Helm.
2. **External IP Configuration**: An external IP address has been patched to Prometheus to facilitate dashboard access.
3. **Integration with Grafana**: Prometheus is added as a data source in Grafana's configuration.

### Grafana

1. **Installation**: Grafana is installed using Helm.
2. **External IP Configuration**: Similar to Prometheus, Grafana is configured with an external IP address.
3. **Dashboards and Alerts**: A dashboard template has been added to Grafana, including an Alert triggred if the number of restarts for the HelloWorld pod exceeds a certain threshold, and an alert triggred if CPU or memory usage for the HelloWorld container goes above a predefined threshold.
