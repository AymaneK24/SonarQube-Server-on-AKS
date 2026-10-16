# SonarQube Server on AKS

Deploy a production-ready SonarQube server on Azure Kubernetes Service (AKS)
using Infrastructure as Code, with centralized authentication through
Microsoft Entra ID (SAML 2.0 SSO).

## Overview

This project automates the full lifecycle of a SonarQube deployment on Azure:

- **Infrastructure**: the AKS cluster and its supporting Azure resources
  [resource group, networking, ...] are provisioned with **Terraform**.
- **Application**: SonarQube is deployed on the cluster with a **Helm chart**,
  with values customized for [persistence, resources, ingress, ...].
- **Authentication**: SonarQube is configured at initial setup to use
  **SAML SSO with Microsoft Entra ID**, so users sign in with their
  corporate identity instead of local accounts.

## Architecture

<p align="center">
    <img src="Screenshot%202026-04-26%20130149.png" alt="Project screenshot" width="700">
</p>

User → Ingress → SonarQube (AKS) → [PostgreSQL]
                     ↑
               SAML SSO ↔ Microsoft Entra ID

## Tech stack

| Layer          | Tool                                  |
|----------------|---------------------------------------|
| Cloud          | Microsoft Azure                       |
| Orchestration  | Azure Kubernetes Service (AKS)        |
| IaC            | Terraform                             |
| Packaging      | Helm                                  |
| Code quality   | SonarQube                             |
| Identity (SSO) | Microsoft Entra ID (SAML 2.0)         |

## Prerequisites

- Azure subscription and Azure CLI (`az login`)
- Terraform >= [version]
- kubectl and Helm 3
- An Entra ID tenant with rights to register an Enterprise Application

## Getting started

1. **Provision the cluster**
```bash
   cd terraform
   terraform init
   terraform apply
```
2. **Connect to AKS**
```bash
   az aks get-credentials --resource-group <rg> --name <cluster>
```
3. **Deploy SonarQube with Helm**
```bash
   helm repo add sonarqube https://SonarSource.github.io/helm-chart-sonarqube
   helm upgrade --install sonarqube sonarqube/sonarqube -f helm/values.yaml -n sonarqube --create-namespace
```
4. **Configure SAML SSO with Entra ID**
   - Create an Enterprise Application in Entra ID (non-gallery app).
   - Set the Identifier (Entity ID) and Reply URL (ACS) to your SonarQube URL
     (`/oauth2/callback/saml`).
   - Copy the Login URL, Entra Identifier and signing certificate into
     SonarQube under *Administration → Authentication → SAML*.

## Repository structure

├── terraform/   # AKS and Azure infrastructure
├── helm/        # values.yaml for the SonarQube chart
└── assets/      # screenshots and diagrams

## Screenshots


<p align="center">
  <img src="Screenshot%202026-03-03%20122709.png" alt="Project screenshot" width="700">
  <img src="Screenshot%202026-03-03%20141506.png" alt="Project screenshot" width="700">
</p>
## Cleanup

terraform destroy

