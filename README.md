# devops-cicd-assignment-

# Static Site CI/CD Pipeline

A complete CI/CD pipeline that builds, tests, and deploys a containerized static website using Jenkins, Docker, AWS, and Kubernetes — with Prometheus and Grafana for monitoring.

## Architecture

```
Developer → Git Push → GitHub → Jenkins Pipeline
    → Build Docker Image → Run Tests → Push to docker hub
    → Deploy to EC2 / Kubernetes(k8s) → Prometheus/Grafana Monitoring
```

## Tech Stack

| Layer | Tool |
|---|---|
| Source control | GitHub |
| CI/CD orchestration | Jenkins |
| Containerization | Docker (Nginx base image) |
| Container registry | Docker hub |
| Compute | AWS EC2 |
| Orchestration | Kubernetes (k3s) |
| Monitoring | Prometheus + Grafana |

## Project Structure

```
.
├── Dockerfile
├── .env.example
├── index.html
├── css/
├── js/
├── Jenkinsfile
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── monitoring/
│   └── prometheus.yml
└── README.md
```

## Prerequisites

- Docker
- AWS CLI configured (`aws configure`)
- kubectl (if deploying to Kubernetes)
- Jenkins with Docker Pipeline, Dockerhub, and SSH Agent plugins

## Local Development

Build and run the container locally:

```bash
docker build -t static-site .
docker run -d -p 8080:80 --name static-test static-site
curl -I http://localhost:8080
```

Stop the test container:

```bash
docker stop static-test && docker rm static-test
```

## CI/CD Pipeline

The Jenkins pipeline (`Jenkinsfile`) runs the following stages on every push to `main`:

1. **Checkout** — pulls latest code from GitHub
2. **Build** — builds the Docker image
3. **Test** — spins up the container and runs a health check
4. **Push Image** — pushes the tagged image to Docker hub
5. **Deploy** — deploys the new image to EC2 (or Kubernetes)

## Deployment

### Option 1: EC2 (Docker)

```bash
docker pull <ecr-repo>:latest
docker run -d --name static-site -p 80:80 --restart unless-stopped <ecr-repo>:latest
```

### Option 2: Kubernetes

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

## Monitoring

Prometheus scrapes host and container metrics via `node-exporter` and `cAdvisor`; Grafana visualizes them.

```bash
docker run -d --name node-exporter -p 9100:9100 prom/node-exporter
docker run -d --name cadvisor -p 8082:8080 -v /var/run/docker.sock:/var/run/docker.sock:ro gcr.io/cadvisor/cadvisor
docker run -d --name prometheus -p 9090:9090 -v $(pwd)/monitoring/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
docker run -d --name grafana -p 3000:3000 grafana/grafana
```

- Grafana: `http://<ec2-ip>:3000` (default login `admin` / `admin`)
- Add Prometheus data source: `http://<ec2-ip>:9090`
- Recommended dashboards: **1860** (Node Exporter Full), **193** (cAdvisor)

## Live Application

`http://<ec2-public-ip>` (update once deployed)

## Deliverables

- [x] GitHub repository
- [x] Dockerfile
- [x] Jenkinsfile
- [x] Deployment configuration (Docker + Kubernetes manifests)
