# Ultimate DevSecOps Pipeline: Github Actions, Docker, Trivy, OWASP ZAP, Trufflehog, Kubernetes, ArgoCD, Prometheus Grafana and Telegram all together.

[![Homelab | Proxmox](https://img.shields.io/badge/Homelab-Proxmox-orange.svg)](https://www.proxmox.com/)
[![Kubernetes | K3s](https://img.shields.io/badge/Kubernetes-K3s-blue.svg)](https://k3s.io/)
[![CI Pipeline](https://img.shields.io/badge/CI-GitHub_Actions-lightgrey.svg)](https://github.com/features/actions)
[![CD Pipeline](https://img.shields.io/badge/CD-ArgoCD-orange.svg)](https://argoproj.github.io/cd/)
[![Security | OWASP ZAP](https://img.shields.io/badge/Security-OWASP_ZAP-green.svg)](https://www.zaproxy.org/)
[![Security | TruffleHog](https://img.shields.io/badge/Security-TruffleHog-lightgrey.svg)](https://github.com/trufflesecurity/trufflehog)
[![Security | Trivy](https://img.shields.io/badge/Security-Trivy-lightgrey.svg)](https://aquasecurity.github.io/trivy/)
[![Monitoring | Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-red.svg)](https://prometheus.io/)
[![Monitoring | Grafana](https://img.shields.io/badge/Monitoring-Grafana-red.svg)](https://grafana.com/)

## 📖 Project Overview

This repository houses a complete, end-to-end DevSecOps and Site Reliability Engineering (SRE) homelab architecture. Built on top of a lightweight Kubernetes cluster (K3s) running on Proxmox, this project demonstrates a fully automated pipeline for a Python FastAPI application. 

The architecture enforces a "Shift-Left" security approach, GitOps continuous deployment, automated dynamic vulnerability scanning, and proactive observability with real-time incident alerting via Telegram.

This project was based in this image 
![Architecture_Diagram](devsecops-pipeline.jpeg)

## 🏗️ Architecture Flow

The pipeline is divided into three core phases:

1. **Continuous Integration (CI) & Security Scanning:** Code pushed to GitHub triggers Actions that scan for exposed secrets, build the Docker container, and scan the OS/dependencies for vulnerabilities.
2. **Continuous Deployment (CD) & Dynamic Testing:** ArgoCD detects changes in the Git repository and automatically synchronizes the K3s cluster. Once deployed, an ephemeral OWASP ZAP container launches to perform dynamic application security testing (DAST).
3. **Observability & Feedback Loop:** The `kube-prometheus-stack` constantly scrapes metrics. Alertmanager evaluates rules and dispatches critical alerts (e.g., `CrashLoopBackOff` or `KubePodNotReady`) directly to a Telegram bot for immediate SRE response.

## 🛠️ Technology Stack

* **Infrastructure:** Proxmox VE, Debian, K3s (Lightweight Kubernetes)
* **Application:** Python, FastAPI
* **CI / Automation:** GitHub Actions, Docker
* **Continuous Deployment:** ArgoCD (GitOps)
* **Security (DevSecOps):**
  * *Secret Scanning:* TruffleHog
  * *Container Scanning:* Trivy
  * *DAST (Dynamic Scanning):* OWASP ZAP (Zed Attack Proxy)
* **Observability (SRE):**
  * *Metrics & Time-Series DB:* Prometheus
  * *Dashboards:* Grafana
  * *Alert Routing:* Alertmanager
  * *Notifications:* Telegram Bot API

## 📂 Project Structure

```
.
├── .github/
│   └── workflows/
│       └── ci-pipeline.yaml      # GitHub Actions CI workflow
├── app/
│   ├── main.py                   # FastAPI application source code
│   ├── requirements.txt
│   └── Dockerfile                # Multi-stage container definition
├── k8s/
│   ├── deployment.yaml           # App Deployment and Service definitions
│   └── dast-job.yaml             # OWASP ZAP ephemeral Job definition
├── monitoring/
│   ├── alertmanager-secret.yaml  # Helm values for Telegram routing & Grafana NodePort
│   └── ...
└── README.md
```

## 🚀 Deployment Guide

### Prerequisites
* A running K3s/Kubernetes cluster.
* `kubectl` and `helm` installed and configured.
* ArgoCD installed in the cluster.

### 1. Application Deployment (GitOps)
This project strictly follows GitOps principles. Do not apply manifests manually.
1. Connect this GitHub repository to your ArgoCD instance.
2. Create an ArgoCD Application tracking the `k8s/` directory.
3. Any pushes to the `main` branch will automatically be reconciled by ArgoCD.

### 2. DAST Security Scanning
The OWASP ZAP scan runs as a Kubernetes Job. ArgoCD handles its deployment, but requires specific security contexts and ephemeral volumes (`emptyDir`) to execute successfully inside the cluster.
* Review scan results via container logs:
  \`kubectl logs -l job-name=zap-dast-scan\`

### 3. Monitoring & Alerting Setup
The observability stack is deployed via the Prometheus Community Helm chart.

1. Add the Helm repository:
   \`helm repo add prometheus-community https://prometheus-community.github.io/helm-charts\`
2. Create the monitoring namespace:
   \`kubectl create namespace monitoring\`
3. Deploy the stack using the custom values file (which contains the Telegram Bot configuration and Grafana NodePort):
   \`helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring -f monitoring/alertmanager-secret.yaml\`

> **⚠️ Security Warning:** Never commit the real Telegram `bot_token` to version control. Inject it directly into the server or use a secret management tool like External Secrets Operator.

## 📊 Observability & Incident Response

* **Grafana Dashboards:** Accessible via NodePort. Standard `Compute Resources / Namespace (Pods)` dashboards provide real-time CPU, Memory, and Network I/O metrics.
* **Alerting Rules:** Configured to wait for a 15-minute grace period (`for: 15m`) on failing pods before dispatching critical alerts to the designated Telegram Chat ID, minimizing alert fatigue.

## 🤝 Contribution
This is a personal homelab project designed to demonstrate SRE and DevSecOps capabilities, but suggestions, issues, and pull requests are always welcome!
