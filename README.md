# CI/CD Pipeline and Monitoring for Python App

## Project Overview
This project demonstrates the setup of a simple CI/CD pipeline for a containerized Python web application along with system monitoring using Prometheus and Grafana.

### Key Objectives
1. Create a Jenkins CI/CD pipeline for building, testing, and deploying a Python application.
2. Set up monitoring using Prometheus and Grafana.
3. Configure alerts for high CPU and memory usage.

---

## CI/CD Pipeline

### Steps Implemented:
1. **Application Setup**:
   - A simple Python web application is built using Flask.
   - Prometheus metrics are exposed using `prometheus-flask-exporter`.

2. **Pipeline Stages**:
   - **Build Stage**:
     - Creates a Docker image of the application.
     - Stores the image locally.
   - **Deploy Stage**:
     - Deploys the application and supporting services using `docker-compose`.
   - **Test Stage**:
     - Runs integration tests to validate application responses.
   - **Monitoring Configuration**:
     - Deploys Prometheus and Grafana services alongside the app.
     - Configures Prometheus to scrape metrics from the application and set up alerting rules.

### Jenkins Pipeline (`Jenkinsfile`):
The Jenkins pipeline script can be found in the repository: [Jenkinsfile](https://github.com/SamikHub/panda/blob/main/Jenkinsfile)

**Pipeline Overview**:
1. **Setup Environment**:
   - Configures workspace and application structure.
2. **Build and Run Infrastructure**:
   - Uses Docker Compose to deploy the app, Prometheus, Grafana, and Nginx.
3. **Integration Testing**:
   - Verifies the application's HTTP response and service availability.
4. **Log Checking and Cleanup**:
   - Logs container outputs and cleans up Docker resources.

---

## Monitoring Setup

### Prometheus Configuration:
- **Prometheus Configuration File**:
  - Scrapes metrics from the Python application and itself.
  - Configures alerting rules:
    - **High Memory Usage**: Alerts if the app exceeds 500MB memory usage.
    - **High CPU Usage**: Alerts if CPU usage exceeds 80%.

  Example `prometheus.yml`:
  ```yaml
  global:
    scrape_interval: 15s

  rule_files:
    - /etc/prometheus/alerts.yml

  scrape_configs:
    - job_name: "panda-app"
      static_configs:
        - targets: ["panda-app:8010"]

    - job_name: "prometheus"
      static_configs:
        - targets: ["localhost:9090"]
  ```

### Grafana Configuration:
- **Dashboard Setup**:
  - Visualizes metrics for application health and resource usage.
  - Example panels:
    - CPU usage over time.
    - Memory usage over time.
- **Alerts Integration**:
  - Displays Prometheus alerts directly on the dashboard.

### Alert Rules (Prometheus):
Example `alerts.yml`:
```yaml
groups:
  - name: panda-app-alerts
    rules:
    - alert: PandaAppDown
      expr: up{job="panda-app"} == 0
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: "Panda App is down"
        description: "Panda App has stopped responding for more than 2 minutes."

    - alert: HighMemoryUsage
      expr: container_memory_usage_bytes{container="panda-app"} > 500000000
      for: 1m
      labels:
        severity: warning
      annotations:
        summary: "High memory usage detected"
        description: "Memory usage for Panda App exceeds 500MB."

    - alert: HighCPUUsage
      expr: rate(container_cpu_usage_seconds_total{container="panda-app"}[1m]) > 0.8
      for: 1m
      labels:
        severity: warning
      annotations:
        summary: "High CPU usage detected"
        description: "CPU usage for Panda App exceeds 80%."
```

---

## Deployment Instructions

### Prerequisites:
- Docker and Docker Compose installed on the host machine.
- Jenkins instance configured to execute pipelines.

### Steps:
1. Clone the repository:
   ```bash
   git clone https://github.com/SamikHub/panda.git
   cd panda
   ```

2. Run the Jenkins pipeline:
   - Set up the pipeline in Jenkins using the `Jenkinsfile` from the repository.
   - Trigger the pipeline to build, deploy, and test the application.

3. Access the services:
   - Application: `http://<host>:8010`
   - Prometheus: `http://<host>:9090`
   - Grafana: `http://<host>:3000` (default credentials: `admin/admin`)

4. View monitoring and alerts:
   - Import the provided Grafana dashboard JSON into Grafana.
   - Verify that alerts are firing as expected under load.

---

## Repository Structure
```
|-- Jenkinsfile
|-- Dockerfile
|-- docker-compose.yml
|-- app/
|   |-- main.py
|-- monitoring/
|   |-- prometheus/
|   |   |-- prometheus.yml
|   |   |-- alerts.yml
|   |-- grafana/
|-- nginx/
|   |-- default.conf
```

---

## Conclusion
This project demonstrates a fully functional CI/CD pipeline integrated with monitoring and alerting for a Python application. The setup ensures smooth deployment, real-time monitoring, and proactive issue detection.

