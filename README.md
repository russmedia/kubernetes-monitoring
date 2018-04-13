# gke-metrics
Prometheus based metrics setup for Kubernetes on Google

## Requirements

1. Running Kubernetes cluster
2. `kubectl` configured to use it.

## Setup

1. Create required binding (required only for Google Kubernetes engine):
```
export GCLOUD_USER=your_email_on_gcloud
kubectl create clusterrolebinding "$GCLOUD_USER-cluster-admin-binding" --clusterrole=cluster-admin --user=$GCLOUD_USER
```
2. Create monitoring namespace
```
kubectl create ns monitoring
```
Why: to avoid mixing with business containes and for easy removal if needed.

3. Create configmap for Prometheus:
```
kubectl apply -f prometheus-configmap.yml -n monitoring
```

4. Create node exporters and kube-state-metrics from coreos:
```
kubectl apply -f exporters.yml -n monitoring
```

5. Create Prometheus and proper roles:
```
kubectl apply -f prometheus.yml -n monitoring
```

6. Create grafana for dashboard:
```
kubectl apply -f grafana.yml -n monitoring
```

## Dashboards

1. Get grafana url or ip:
```
kubectl get svc -n monitoring | grep grafana
```

2. Login with `admin/admin` to ip/url found in previous step.

3. Add datasource:
type: prometheus
url: http://prometheus-k8s:9090
access: proxy

4. Import dashboard:
- use `+` -> import on the left panel
- type dashboard id `1621`
- change datasource type to prometheus
- save

Note: if grafana is on public url/ip, change default password!