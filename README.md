# prometheus_grafana



 ```ls```

sudo mkdir -p /mnt/data/prometheus-alertmanager
sudo chmod 777 /mnt/data/prometheus-alertmanager

sudo mkdir -p /mnt/data/prometheus
sudo chmod 777 /mnt/data/prometheus

sudo mkdir -p /mnt/data/grafana
sudo chmod 777 /mnt/data/grafana



cat prometheus-pv.yaml 

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
    path: /mnt/data/prometheus   # <-- directory on your RHEL node
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
  volumeName: prometheus-pv   # bind PVC to PV explicitly
```


cat alertmanager-pv.yaml 

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

cat grafana-pv.yaml 

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
    path: /mnt/data/grafana   # directory on RHEL node
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
```helm install prometheus prometheus-community/prometheus --namespace monitoring```

```km get pods```

delete the exisitig pv, pvcs and apply the one above from files.
k apply -f prometheus-pv.yaml
k apply -f alertmanager-pv.yaml
k apply -f grafana-pv.yaml

```kubectl patch svc prometheus-server -n monitoring -p '{
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

helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana --namespace monitoring

kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
vi grafana-values.yaml

```
persistence:
  enabled: true
  existingClaim: grafana
  accessModes:
    - ReadWriteOnce
  size: 2Gi
```

helm upgrade grafana grafana/grafana   --namespace monitoring -f grafana-values.yaml

kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

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

km get pods
kubectl get pods -n monitoring | grep kube-state-metrics
kubectl get pods -n monitoring | grep node-exporter
 1137  k get pods
 1138  k delete pod sim-onboarding-775956bb8b-hh5zr
 1139  k get pods
 1140  history
