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

```text
+--------------------------+
|   Kubernetes Cluster     |
| (Nodes, Pods, Services)  |
+------------+-------------+
             |
             v
   +------------------+
   |   Exporters      |  (Node Exporter, kube-state-metrics,
   |   App Metrics    |   Custom App Exporters)
   +--------+---------+
             |
             v
   +------------------+
   |   Prometheus     |
   | (Scrapes & Stores|
   |   Time Series)   |
   +--------+---------+
             |
             v
   +------------------+
   |    Grafana       |
   | (Dashboards &    |
   |   Visualizations)|
   +------------------+
```
# 🛠️ Installation & Configurations

### 📦 Step 1 : Create EKS Cluster

###  Kubernetes cluster (minikube, kind, EKS, GKE, AKS, etc.)

### 🧰 Step 2 : Install kube-prometheus-stack
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
### 🚀 Step 3 : Deploy the chart into a new namespace "monitoring"
```
kubectl create ns monitoring
```
```
helm install monitoring prometheus-community/kube-prometheus-stack \
-n monitoring \
-f ./custom_kube_prometheus_stack.yml
```
### ✅ Step 4 : Install Prometheus Helm Chart on Kubernetes Cluster
```
helm install prometheus prometheus-community/prometheus
```
##### The output of the kubectl get pods :  
```bash
ankitha@ankitha:~$ kubectl get pods
NAME                                           READY   STATUS    RESTARTS   AGE
prometheus-alertmanager-0                      1/1     Running   0          59s
prometheus-kube-state-metrics-5f64969966-6tx24 1/1     Running   0          60s
prometheus-prometheus-node-exporter-skx49      1/1     Running   0          61s
prometheus-prometheus-pushgateway-65bc997fdf-rc46l 1/1 Running   0          60s
prometheus-server-9c64d4bf4-cb9zm              2/2     Running   0          60s
```
### 🛠️ Step 5 : Exposing the Prometheus-server service on Kubernetes
```
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext
```
##### The output of the kubectl get svc :
```bash
ankitha@ankitha:~$ kubectl get svc
NAME                                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes                            ClusterIP   10.96.0.1        <none>        443/TCP        20h
prometheus-alertmanager               ClusterIP   10.107.38.209    <none>        9093/TCP       5h30m
prometheus-alertmanager-headless      ClusterIP   None             <none>        9093/TCP       5h30m
prometheus-kube-state-metrics         ClusterIP   10.99.24.251     <none>        8080/TCP       5h30m
prometheus-prometheus-node-exporter   ClusterIP   10.98.206.69     <none>        9100/TCP       5h30m
prometheus-prometheus-pushgateway     ClusterIP   10.99.196.97     <none>        9091/TCP       5h30m
prometheus-server                     ClusterIP   10.106.132.250   <none>        80/TCP         5h30m
prometheus-server-ext                 NodePort    10.96.142.233    <none>        80:32181/TCP   3s
```

#### Our Prometheus web UI is available now, With the installation of Prometheus on Kubernetes via Helm, the Prometheus instance is now operational within the cluster, and we can reach it by navigating to a browser or using a specific URL.

<img width="1127" height="665" alt="image" src="https://github.com/user-attachments/assets/c36e3087-8ce5-4d14-9797-d7efd4db588f" />

### ⚡Step 6 : Grafana Installation
```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```
### 📈 Step 7 : Install Grafana Helm Chart on Kubernetes Cluster
```
helm install grafana grafana/grafana
```
### 🧭 Step 8 : Exposing the Grafana service on Kubernetes
```
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext
```
##### The output of the kubectl get svc :
```bash
ankitha@ankitha:~$ kubectl get svc
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
grafana                              ClusterIP   10.110.238.0    <none>        80/TCP           2m7s
grafana-ext                          NodePort    10.107.35.57    <none>        80:32434/TCP     5s
kubernetes                           ClusterIP   10.96.0.1       <none>        443/TCP          20h
prometheus-alertmanager              ClusterIP   10.107.38.209   <none>        9093/TCP         5h48m
prometheus-alertmanager-headless     ClusterIP   None            <none>        9093/TCP         5h48m
prometheus-kube-state-metrics        ClusterIP   10.99.24.251    <none>        8080/TCP         5h48m
prometheus-prometheus-node-exporter  ClusterIP   10.98.206.69    <none>        9100/TCP         5h48m
prometheus-prometheus-pushgateway    ClusterIP   10.99.196.97    <none>        9091/TCP         5h48m
prometheus-server                    ClusterIP   10.106.132.250  <none>        80/TCP           5h48m
prometheus-server-ext                NodePort    10.96.142.233   <none>        80:32181/TCP     17m
```

###  Our Garafana web UI is available now, and we can reach it by navigating to a browser or using a specific URL.

<img width="1457" height="892" alt="image" src="https://github.com/user-attachments/assets/f6fc18c7-b631-4a4b-a835-6394b0d0a383" />

#### To get the password for admin for the Grafana Login page, run the below command on a new terminal.
```
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```
### 📊 Step 9 : Add Prometheus as data source

### To add Prometheus as the data source, follow these steps:

* On the Welcome to Grafana Home page, click Add your first data source:
  
* Select Prometheus as the data source:

<img width="1635" height="630" alt="image" src="https://github.com/user-attachments/assets/20047ba3-daa6-4744-9299-790e9030fc20" />

Include the Prometheus URL from the browser.

<img width="1917" height="769" alt="image" src="https://github.com/user-attachments/assets/07dbd291-7e5f-4a70-9975-f806a8b899e8" />

Click on ‘Save and Next’.

You have successfully added the Prometheus as Data source.

<img width="1906" height="394" alt="image" src="https://github.com/user-attachments/assets/4aaad10d-8946-49ec-81b7-b5504c9b88b1" />


### ⚡Step 10 : Grafana Dashboard

In this section, our focus will be on importing a Grafana Dashboard to streamline the process.

#### To import a Grafana Dashboard, follow these steps:

 ##### * Get the Grafana Dashboard ID from the - [![Grafana public Dashboard library](https://grafana.com/grafana/dashboards/)
  
<img width="1885" height="1000" alt="image" src="https://github.com/user-attachments/assets/b332e9f0-7174-44ab-84c0-9e5c8532dfe8" />

 ##### * Now go to the search dashboard, and search for Kubernetes :
  
  <img width="1011" height="781" alt="image" src="https://github.com/user-attachments/assets/44e9b79d-d027-4bc6-a257-7b846a4be5cc" />
  
 ##### * Select Dashboard and copy the Dashboard ID:
 
  <img width="1681" height="847" alt="image" src="https://github.com/user-attachments/assets/9a3036bf-5271-411f-9586-fc6314694535" />
  
 ##### * Go back to the Grafana home, and go to the dashboard on left corner
 
  <img width="1087" height="810" alt="image" src="https://github.com/user-attachments/assets/eac95a19-9990-4639-ba46-bb316ff8ffb3" />
  
 ##### * Now click on the new->import
 
  <img width="1908" height="456" alt="image" src="https://github.com/user-attachments/assets/2b8d0ed6-3940-49b0-895e-c57584581e4c" />
  
 ##### * Add the Grafana Id, and click on ‘Load’
 
  <img width="1062" height="441" alt="image" src="https://github.com/user-attachments/assets/b8181fd4-7d99-408f-adb0-4b628544b87a" />
  
 ##### * Select a Prometheus Data Source and Click Import
 
  <img width="1180" height="664" alt="image" src="https://github.com/user-attachments/assets/b857cbd9-e376-48bc-9ff1-76d9a3cebd59" />
  
 ##### * It will the launch the Dashboard shown below:
 
  <img width="1909" height="946" alt="image" src="https://github.com/user-attachments/assets/a29228ea-89bc-4663-90cc-83f7883b508b" />

##### Use this dashboard to monitor and observe the Kubernetes cluster metrics. It displays the following Kubernetes cluster metrics:
* Network I/O pressure.
* Cluster CPU usage.
* Cluster Memory usage.
* Cluster filesystem usage.
* Pods CPU usage.

<img width="1842" height="861" alt="image" src="https://github.com/user-attachments/assets/0fe75b80-bdc4-4fa1-b426-28dc4c91dca0" />

<img width="1843" height="850" alt="image" src="https://github.com/user-attachments/assets/758fff79-a67f-449a-b5bb-4c0c19863ba9" />

