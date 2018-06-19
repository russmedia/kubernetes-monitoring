# kubernetes-monitoring
Prometheus based metrics setup for Kubernetes on Google.

All clouds and platforms hosting Kubernetes can utilize this approach, but some details needs to be adjusted i.e. `PersistentVolumeClaim` part.

![sample dashboard](images/sample_dashboard.png)

## Requirements

1. Running Kubernetes cluster
2. `kubectl` configured to use it.

## Setup

1. Create required binding (required only for Google Kubernetes engine):
```
export GCLOUD_USER=your_email_on_gcloud
kubectl create clusterrolebinding "$GCLOUD_USER-cluster-admin-binding" --clusterrole=cluster-admin --user=$GCLOUD_USER
```

**Please note!** If you have ```Legacy authorization	Enabled``` on your cluster master, the above rolebinding will not work. Instead add Kuberentes Cluster Admin privilage to your active account from **IAM & admin  -> [your user account] -> Kubernetes Cluster Admin.**
Login again with ```gcloud auth login``` and initiate your cluster connection again (you have a **connect** button in the console right next to the cluster)

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

- empty Grafana with persistent volume:

```
cd grafana/empty
kubectl apply -f grafana.yml -n monitoring
```

or 

- preloaded Grafana with persistent volume (please consider kubectl apply message limit):

```
cd grafana/preloaded
kubectl apply -f grafana-configmap.yml -n monitoring # configuration for Grafana
kubectl apply -f grafana-dashboards.yml -n monitoring # dashboards for Grafana
kubectl apply -f grafana.yml -n monitoring
```

## Dashboards

Dashboards are loaded automatically from files in `grafana-dashboards.yml`.
It is preferable approach to keep them in version controll repository, altough pvc is used for `/var/lib/grafana` path.

## 1. Manual adding dashboard from grafana.org

1. Get grafana url or ip:
```
kubectl get svc -n monitoring | grep grafana
```

2. Login with `admin/admin`(or other credentials if defults were changed) to ip/url found in previous step.

3. Add datasource( if not loaded automatically):
type: prometheus
url: http://prometheus-k8s:9090
access: proxy

4. Import dashboard:
- use `+` -> import on the left panel
- type dashboard id `1621`
- change datasource type to prometheus
- save

Note: if grafana is on public url/ip, change default password!
