
# Week 6: GitHub Actions and Observability

## What This Lab Does

This lab uses GitHub Actions to automatically check OpenTofu
files when changes are submitted through a pull request.

It also uses Helm to install Prometheus on a Kubernetes
cluster so we can monitor our application's performance.

## Requirements

- OpenTofu
- Helm
- Kubernetes with the Week 5 application running
- GitHub CLI with workflow permissions

## Install Prometheus

Run these commands:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus \
  --version 29.35.0 \
  --namespace monitoring \
  --create-namespace \
  --values week-6/prometheus-values.yaml
```

## Check Prometheus

Open a terminal and run:

```bash
kubectl port-forward --namespace monitoring service/prometheus-server 9090:80
```

In another terminal:

```bash
./week-6/pq.sh 'up'
```

## Remove Prometheus

```bash
helm uninstall prometheus --namespace monitoring
```

## Known Limitation

Prometheus uses temporary storage in this lab. If its pod
restarts, the stored monitoring history can be lost.
For security, Prometheus is only accessed through a local
port-forward instead of being exposed publicly.
