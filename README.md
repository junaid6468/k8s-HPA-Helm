# k8s-HPA-Helm

A small example project demonstrating a simple microservice stack (two web services + nginx + redis) deployed to Kubernetes using both raw manifests and a Helm chart, with Horizontal Pod Autoscaler (HPA) examples and CI automation via GitLab CI.

This repository contains:
- A minimal Node.js web application (web/)
- An nginx reverse-proxy (nginx/)
- Kubernetes manifests for manual deployment (kubernetes/)
- A Helm chart to deploy the stack (helm/nginx-redis-app/)
- GitLab CI pipeline for build, push, and (optionally) deploy (.gitlab-ci.yml)

## Features
- Dockerized web and nginx images
- Kubernetes Deployments and Services for web1, web2, nginx and redis
- HPA manifests (web1-hpa.yaml, web2-hpa.yaml) to demonstrate autoscaling
- Helm chart to parameterize and install the app easily
- CI pipeline for automated builds, pushes, and Helm operations

## Stack
- Language(s): JavaScript (web service), Dockerfiles for images
- Runtime: Node.js (simple Express-like server in `web/server.js`)
- Kubernetes: standard k8s manifests + Helm chart
- CI: GitLab CI (`.gitlab-ci.yml`)

## Repo layout

```
.kit/ or root files
helm/
  nginx-redis-app/        Helm chart for deploying the app
    Chart.yaml
    values.yaml
    templates/
      nginx-deployment.yaml
      nginx-service.yaml
      redis-deployment.yaml
      redis-service.yaml
      web1-deployment.yaml
      web1-service.yaml
      web1-hpa.yaml
      web2-deployment.yaml
      web2-service.yaml
      web2-hpa.yaml
kubernetes/
  <same manifests as templates for manual apply>
  nginx-deployment.yaml
  nginx-service.yaml
  redis-deployment.yaml
  redis-service.yaml
  web1-deployment.yaml
  web1-service.yaml
  web1-hpa.yaml
  web2-deployment.yaml
  web2-service.yaml
  web2-hpa.yaml
nginx/
  Dockerfile
  nginx.conf
web/
  Dockerfile
  package.json
  server.js
  package-lock.json
.gitlab-ci.yml
```

## Quick start — local Kubernetes (minikube / kind)
These steps assume you have Docker, kubectl, and either minikube or kind available.

1. Build Docker images
- For local cluster with kind:
  - Build images locally and load them into kind:
    - docker build -t my-registry/web:latest -f web/Dockerfile ./web
    - docker build -t my-registry/nginx:latest -f nginx/Dockerfile ./nginx
    - kind load docker-image my-registry/web:latest
    - kind load docker-image my-registry/nginx:latest
- For minikube:
    - eval $(minikube docker-env)
    - docker build -t my-registry/web:latest -f web/Dockerfile ./web
    - docker build -t my-registry/nginx:latest -f nginx/Dockerfile ./nginx

2. Deploy with Kubernetes manifests (manual)
- kubectl apply -f kubernetes/
  This applies Deployments, Services and HPA manifests under `kubernetes/`.

3. Or deploy with Helm
- From the repo root run:
  - helm install nginx-redis-app ./helm/nginx-redis-app --namespace my-namespace --create-namespace
- To customize, pass a values file:
  - helm install nginx-redis-app ./helm/nginx-redis-app -n my-namespace -f ./helm/nginx-redis-app/values.yaml

4. Verify deployment
- kubectl get pods -n my-namespace
- kubectl get svc -n my-namespace
- kubectl get hpa -n my-namespace

Note: HPAs require metrics; ensure metrics-server is installed in the cluster (for Minikube: `minikube addons enable metrics-server`).

## Helm chart usage
- Chart path: `helm/nginx-redis-app/`
- Default values: `helm/nginx-redis-app/values.yaml`
- Chart contains deployments and HPA YAML templates:
  - web1/web2 deployments and services
  - redis deployment/service
  - nginx deployment/service
  - web1-hpa.yaml and web2-hpa.yaml templates for autoscaling

To render templates locally:
- helm template ./helm/nginx-redis-app

To upgrade after changes:
- helm upgrade nginx-redis-app ./helm/nginx-redis-app -n my-namespace -f ./helm/nginx-redis-app/values.yaml

## Docker image hints
- Web image: Dockerfile at `web/Dockerfile`
- Nginx image: Dockerfile at `nginx/Dockerfile`
- Tags and repository names used by your CI pipeline are defined in `.gitlab-ci.yml` environment variables (for CI-based pushes).
- If pushing to Docker Hub / a registry:
  - docker tag my-registry/web:latest <your-registry>/web:<tag>
  - docker push <your-registry>/web:<tag>

## CI / GitLab
- `.gitlab-ci.yml` provides pipeline stages for validate, build, push, deploy, and cleanup.
- Variables used in CI include: `DOCKER_USERNAME`, `IMAGE_TAG`, and others shown in the pipeline file.
- The pipeline builds both the web and nginx images and optionally deploys the Helm chart.

## Configuration
- Change container images and tags via `helm/nginx-redis-app/values.yaml` or by editing the manifests in `kubernetes/`.
- HPA thresholds and resource requests/limits should be tuned in `values.yaml` (or the HPA templates in the chart).

## Troubleshooting & tips
- HPA shows `Unknown` without metrics-server. Install metrics-server to provide CPU/memory metrics.
- If pods crash: check logs:
  - kubectl logs <pod> -n my-namespace
- If Helm install fails, render templates to inspect:
  - helm template ./helm/nginx-redis-app

## Development
- Run the web service locally:
  - cd web
  - npm install
  - node server.js
  - By default the simple server serves a demo endpoint (inspect `web/server.js`).

## Security & production notes
- This repo is an example/demo — do not use as-is in production.
- Add robust readiness/liveness probes, security contexts, and proper image registries, secrets handling for production use.
- Use resource requests/limits to make HPA decisioning stable.

## Contributing
Feel free to open issues or PRs. Suggested improvements:
- Add README examples for customizing `values.yaml`
- Add a Makefile or scripts to build and load images for kind/minikube automatically
- Add automated tests for the web service

