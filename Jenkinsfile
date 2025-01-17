pipeline {
    agent any

    environment {
        DOMAIN = "dovhyy.site"
        APP_PORT = 8010
        PROMETHEUS_PORT = 9090
        GRAFANA_PORT = 3000
        NGINX_PORT = 80
        ALERT_MEMORY_USAGE = 500000000  // 500MB
        ALERT_CPU_USAGE = 0.8          // 80% CPU
    }

    stages {
        stage('Setup Environment') {
            steps {
                script {
                    sh """
                    mkdir -p ${WORKSPACE}/app
                    mkdir -p ${WORKSPACE}/monitoring/prometheus
                    mkdir -p ${WORKSPACE}/monitoring/grafana
                    mkdir -p ${WORKSPACE}/nginx

                    # Flask Application
                    echo 'from flask import Flask
from prometheus_flask_exporter import PrometheusMetrics

app = Flask(__name__)
metrics = PrometheusMetrics(app)

@app.route("/")
def hello():
    return "Hello, Panda App!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=${APP_PORT})' > ${WORKSPACE}/app/main.py

                    # Dockerfile for Panda App
                    echo 'FROM python:3.11-slim
WORKDIR /app
COPY app/main.py /app/main.py
RUN pip install flask prometheus-flask-exporter
EXPOSE ${APP_PORT}
CMD ["python", "/app/main.py"]' > ${WORKSPACE}/Dockerfile

                    # Prometheus config
                    echo 'global:
  scrape_interval: 15s

rule_files:
  - /etc/prometheus/alerts.yml

scrape_configs:
  - job_name: "panda-app"
    static_configs:
      - targets: ["panda-app:${APP_PORT}"]

  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:${PROMETHEUS_PORT}"]' > ${WORKSPACE}/monitoring/prometheus/prometheus.yml

                    # Prometheus alert rules
                    echo 'groups:
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
      expr: container_memory_usage_bytes{container="panda-app"} > ${ALERT_MEMORY_USAGE}
      for: 1m
      labels:
        severity: warning
      annotations:
        summary: "High memory usage detected"
        description: "Memory usage for Panda App exceeds 500MB."

    - alert: HighCPUUsage
      expr: rate(container_cpu_usage_seconds_total{container="panda-app"}[1m]) > ${ALERT_CPU_USAGE}
      for: 1m
      labels:
        severity: warning
      annotations:
        summary: "High CPU usage detected"
        description: "CPU usage for Panda App exceeds 80%."' > ${WORKSPACE}/monitoring/prometheus/alerts.yml

                    # Nginx config
                    echo 'server {
    server_name panda.${DOMAIN};

    location / {
        proxy_pass http://panda-app:${APP_PORT};
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    }
}

server {
    server_name prometheus.${DOMAIN};

    location / {
        proxy_pass http://prometheus:${PROMETHEUS_PORT};
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    }
}

server {
    server_name grafana.${DOMAIN};

    location / {
        proxy_pass http://grafana:${GRAFANA_PORT};
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    }
}' > ${WORKSPACE}/nginx/default.conf

                    # Docker Compose file
                    echo 'services:
  panda-app:
    build: .
    container_name: panda-app
    ports:
      - "${APP_PORT}:${APP_PORT}"
    networks:
      - monitoring

  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ${WORKSPACE}/monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ${WORKSPACE}/monitoring/prometheus/alerts.yml:/etc/prometheus/alerts.yml
    ports:
      - "${PROMETHEUS_PORT}:${PROMETHEUS_PORT}"
    networks:
      - monitoring

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "${GRAFANA_PORT}:${GRAFANA_PORT}"
    volumes:
      - grafana-data:/var/lib/grafana
    networks:
      - monitoring

  nginx:
    image: nginx:latest
    container_name: nginx
    volumes:
      - ${WORKSPACE}/nginx/default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "${NGINX_PORT}:${NGINX_PORT}"
    networks:
      - monitoring

volumes:
  grafana-data:

networks:
  monitoring:
    driver: bridge' > ${WORKSPACE}/docker-compose.yml
                    """
                }
            }
        }

        stage('Build and Run Infrastructure') {
            steps {
                script {
                    sh """
                    docker-compose -f ${WORKSPACE}/docker-compose.yml down || true
                    docker-compose -f ${WORKSPACE}/docker-compose.yml up -d
                    sleep 30
                    """
                }
            }
        }

        stage('Integration Test') {
            steps {
                script {
                    sh """
                    curl -f http://panda.${DOMAIN} || (echo 'Panda App failed' && exit 1)
                    curl -f http://prometheus.${DOMAIN} || (echo 'Prometheus failed' && exit 1)
                    curl -f http://grafana.${DOMAIN} || (echo 'Grafana failed' && exit 1)
                    """
                }
            }
        }

        stage('Check Logs') {
            steps {
                script {
                    sh """
                    echo "Checking logs..."
                    docker logs panda-app
                    docker logs prometheus
                    docker logs grafana
                    docker logs nginx
                    """
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    sh """
                    docker-compose -f ${WORKSPACE}/docker-compose.yml down
                    docker volume prune -f
                    docker network prune -f
                    """
                }
            }
        }
    }
}