# Mô hình 

|------------| - Node exporter : 9100
| Kubernetes | - Kube-state-metric : 8080,8081
|------------|
     ^^
     ||
     ||
|------------|      |---------|   
| Prometheus | ===> | Grafana |  
|------------|      |---------|



# Cácthành phần sử dụng 

- Node exporter : chạy dưới dạngDaemonSets trên các máy node 
- Kube-state-metric 

- 



# Cấu hình 



