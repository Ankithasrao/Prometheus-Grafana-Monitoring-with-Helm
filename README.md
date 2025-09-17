# 📊 Prometheus & Grafana Monitoring on Kubernetes (Helm)

[![Helm](https://img.shields.io/badge/Helm-Chart-blue?logo=helm)](https://helm.sh/)  
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Cluster-326ce5?logo=kubernetes)](https://kubernetes.io/)  
[![Grafana](https://img.shields.io/badge/Grafana-Dashboards-F46800?logo=grafana)](https://grafana.com/)  
[![Prometheus](https://img.shields.io/badge/Prometheus-Metrics-E6522C?logo=prometheus)](https://prometheus.io/)

A Kubernetes monitoring stack using **Prometheus** & **Grafana**, deployed with **Helm**.  


---

## 🚀 Overview
- 📡 **Prometheus** → Scrapes metrics from Kubernetes and applications  
- 📊 **Grafana** → Visualizes metrics with dashboards  
- ⚡ **Exporters** → Node, kube-state-metrics, and app-level exporters  
- 🔔 (Optional) **Alertmanager** → Alerting system for Prometheus  

---

## 🏗️ Architecture
+--------------------+ +------------------+
| Kubernetes Cluster | ----> | Prometheus |
| (Nodes, Pods, etc.)| | (Scraping) |
+--------------------+ +------------------+
|
v
+---------------+
| Grafana |
| (Dashboards) |
+---------------+

# 🛠️ Installation & Configurations

### 📦 Step 1: Create EKS Cluster

##### Kubernetes cluster (minikube, kind, EKS, GKE, AKS, etc.)

### 🧰 Step 1: Install kube-prometheus-stack
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
### 🚀 Step 3: Deploy the chart into a new namespace "monitoring"
```
kubectl create ns monitoring
```
```
helm install monitoring prometheus-community/kube-prometheus-stack \
-n monitoring \
-f ./custom_kube_prometheus_stack.yml
```
#### ✅ Step 4: Verify the Installation
```
kubectl get all -n monitoring
```
* ##### Prometheus UI:
```
kubectl port-forward service/prometheus-operated -n monitoring 9090:9090
```
##### NOTE: If you are using an EC2 Instance or Cloud VM, you need to pass --address 0.0.0.0 to the above command. Then you can access the UI on instance-ip:port

* ##### Grafana UI:
```
kubectl port-forward service/monitoring-grafana -n monitoring 8080:80
```
* ##### Alertmanager UI:
```
kubectl port-forward service/alertmanager-operated -n monitoring 9093:9093
```
