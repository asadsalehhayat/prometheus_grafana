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

km get pv
km edit pv prometheus-pv
 1099  km get pv
 1100  km get pvc
 1101  km delete pvc prometheus-server
 1102  km get pvc
 1103  km get pv
 1104  km delete pv prometheus-pv
 1105  km edit pv prometheus-pv
 1106  km get pv
 1107  k edit pv onboarding-pv-new
 1108  k delete pv onboarding-pv-new
 1109  k edit pv onboarding-pv-new
 1110  km get pv
 1111  k apply -f prometheus-pv.yaml 
 1112  km get pv
 1113  km get pvc
 1114  
 1116  sudo vi alertmanager-pv.yaml
 1117  km get pv
 1118  km get pvc
 1119  km delete pvc storage-prometheus-alertmanager-0
 1120  km edit pvc storage-prometheus-alertmanager-0
 1121  k apply -f alertmanager-pv.yaml 
 1122  km get pv
 1123  km get pvc
 1124  km get pods
 1125  kubectl patch svc prometheus-server -n monitoring -p '{
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
 1126  helm repo add grafana https://grafana.github.io/helm-charts
 1127  helm repo update
 1128  helm install grafana grafana/grafana --namespace monitoring
 1129  kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
 1130  vi grafana-values.yaml
 1131  helm upgrade grafana grafana/grafana   --namespace monitoring -f grafana-values.yaml
 1132     kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
 1133  kubectl patch svc grafana -n monitoring -p '{
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
 1134  km get pods
 1135  kubectl get pods -n monitoring | grep kube-state-metrics
 1136  kubectl get pods -n monitoring | grep node-exporter
 1137  k get pods
 1138  k delete pod sim-onboarding-775956bb8b-hh5zr
 1139  k get pods
 1140  history
