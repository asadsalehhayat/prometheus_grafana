# Prometheus & Grafana Setup on Kubernetes

This guide explains how to deploy Prometheus and Grafana on a Kubernetes cluster using Helm, with persistent storage configured using hostPath PersistentVolumes (PVs) and PersistentVolumeClaims (PVCs).

---

## Prerequisites

- Kubernetes cluster with `kubectl` access
- Helm installed
- Namespace `monitoring` created:

```bash
kubectl create namespace monitoring
```


Step 1: Prepare Directories on Node

Create directories for Prometheus, Alertmanager, and Grafana data and set permissions:

```
sudo mkdir -p /mnt/data/prometheus-alertmanager
sudo chmod 777 /mnt/data/prometheus-alertmanager

sudo mkdir -p /mnt/data/prometheus
sudo chmod 777 /mnt/data/prometheus

sudo mkdir -p /mnt/data/grafana
sudo chmod 777 /mnt/data/grafana
```


Step 2: Install Prometheus

```

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus --namespace monitoring

```



Patch Prometheus service to NodePort:


```
kubectl patch svc prometheus-server -n monitoring -p '{
  "spec": {
    "type": "NodePort",
    "ports":[
      {
        "name":"http",
        "port":80,
        "targetPort":9090,
        "nodePort":31190
      }
    ]
  }
}'

```


Step 3: Create PersistentVolumes and PersistentVolumeClaims

prometheus-pv.yaml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: prometheus-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data/prometheus
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prometheus-server
  namespace: monitoring
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  volumeName: prometheus-pv
```

alertmanager-pv.yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: prometheus-alertmanager-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data/prometheus-alertmanager
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: storage-prometheus-alertmanager-0
  namespace: monitoring
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  volumeName: prometheus-alertmanager-pv
```
grafana-pv.yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: grafana-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data/grafana
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana
  namespace: monitoring
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  volumeName: grafana-pv
```



kubectl delete pv,pvc --all -n monitoring
kubectl apply -f prometheus-pv.yaml
kubectl apply -f alertmanager-pv.yaml
kubectl apply -f grafana-pv.yaml



```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana --namespace monitoring
```


kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo



Create grafana-values.yaml to use existing PVC:


```
persistence:
  enabled: true
  existingClaim: grafana
  accessModes:
    - ReadWriteOnce
  size: 2Gi
```



helm upgrade grafana grafana/grafana --namespace monitoring -f grafana-values.yaml




```
kubectl patch svc grafana -n monitoring -p '{
  "spec": {
    "type": "NodePort",
    "ports":[
      {
        "name":"http",
        "port":80,
        "targetPort":3000,
        "nodePort":31191
      }
    ]
  }
}'

```


```
kubectl get pods -n monitoring
kubectl get pods -n monitoring | grep kube-state-metrics
kubectl get pods -n monitoring | grep node-exporter



```

Access URLs

Prometheus: http://<NODE_IP>:31190

Grafana: http://<NODE_IP>:31191
