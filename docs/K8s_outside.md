Monitor Kubernetes cluster sử dụng Prometheus và Grafana external. 

# Mô hình 

```
|------------| - Node exporter : 9100
| Kubernetes | - Kube-state-metrics : 8080,8081
|------------|
     ^^
     ||
     ||
|------------|      |---------|   
| Prometheus | ===> | Grafana |  
|------------|      |---------|
```

# Các thành phần sử dụng 

Trên Kubernetes cluster: 
- [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics) để generate ra các metric về state của các object. 
- [node exporter](https://github.com/prometheus/node_exporter) để thu thập metric phần cứng và OS của máy node. 

Trên Grafana:
- [Kubernetes plugin](https://github.com/grafana/kubernetes-app) hỗ trợ một vài dashboard mẫu 

Trên Prometheus:
- Cấu hình để pull metric bằng [kubernetes_sd_config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config)

# Cấu hình 
### Deploy kube-state-metrics on Kubernetes cluster

```
git clone https://github.com/locvx1234/prometheus-notes.git
kubectl apply -f kubernetes
```

Deployment `kube-state-metrics` được tạo ra với pod nhận IP `hostNetwork` để Prometheus có thể scrape vào trong.  

### Deloy node-exporter on Kubernetes cluster

```
kubectl apply -f grafanak8s-node-exporter-ds.yaml
```

DaemonSet `node-exporter` được tạo ra với các pod chạy trên các máy worker node 

### Config prometheus querier 

Tạo một `serviceAccount` và bind role cho nó

```
kubectl create sa prometheus -n kube-system
kubectl create clusterrolebinding prometheus-querier --clusterrole=prometheus-querier --serviceaccount=kube-system:prometheus
kubectl create -f prometheus-querier.yaml
```


Để Prometheus gọi được vào API kubernetes, cần pass bươc xác thực. Tạo 2 file `ca.crt` và `token` tại đường dẫn `/etc/prometheus/` trên Prometheus server.

- Get `ca.crt` tại `/etc/kubernetes/pki/ca.crt` trên K8S-master

- Get `token` bằng cách:

```
SECRET=$(kubectl get sa prometheus  -n kube-system -o jsonpath='{.secrets[0].name}')
kubectl describe secret -n kube-system $SECRET
```

Trên Prometheus server:

Thêm vào file config prometheus `/etc/prometheus/prometheus.yml` với các target như trong file `prometheus-k8s-config.yml` sau đó restart service

### Config grafana 

Install:
```
grafana-cli plugins install grafana-kubernetes-app
```

Sau đó restart grafana 

Chọn logo kubenetes => Cluster => +New Cluster 

Điền các trường Name, URL (K8S API), xác thực bằng CA cert và TLS client Auth, datasource trỏ tới prometheus server

Các trường về key, cert được lấy trong file `.kube/config` và decode base64 

![new_cluster](https://raw.githubusercontent.com/locvx1234/prometheus-notes/master/images/new_cluster.png)

Sau khi điền xong, chọn `Save` để hoàn tất 

### Bejoy !!!

![dashboard](https://raw.githubusercontent.com/locvx1234/prometheus-notes/master/images/dashboard-k8s.png)


### Ref

https://medium.com/thg-tech-blog/monitoring-multiple-kubernetes-clusters-88cf34442fa3

https://medium.com/@nieldw/curling-the-kubernetes-api-server-d7675cfc398c