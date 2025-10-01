### 1. Monitoring Kubernetes Cluster Components
- Monitoring solutions:
  - Metrics Server
  - Prometheus
  - Elastic Stack
  - Data Dog
  - Dynatrace
- Metrics Server:
  - It retrieves metrics from each of the Kubernetes nodes and pods, aggregates them and store it in the memory.
  - The metrics server is only the in-memory monitoring solution --> you cannot see historical performance data
  - CAdvisor is responsible for retrieving performance metrics from pods and exposing them through the Kubelet API to make the metrics available for the metrics server.
- Viewing Metrics:
  - kubectl top node
  - kubectl top pod

### 2. Application Logs

