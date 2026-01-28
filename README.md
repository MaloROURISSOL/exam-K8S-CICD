# Exam CI/CD + Kubernetes - Malo Rourissol

## Architecture

API REST (Node.js Express) conteneurisée sous Docker et déployée sur Kubernetes, avec une touche de CI/CD.

Composants :
- **Application** : API Express avec les endpoints `/health` et `/api/info`
- **Docker** : deps -> test -> runtime
- **CI/CD** : GitHub Actions avec 3 jobs (build, push, health)
- **Kubernetes** : 2 réplicas + ConfigMap + Service + Probes

## Choix techniques
Dockerfile :
- Image Alpine en LTS V22
- build pour séparer test/prod
- `npm ci` pour build

CI/CD :
- Pipeline bloquante entre jobs
- Test automatique du endpoint `/health` après push

Kubernetes :
- `replicas: 2`
- `maxUnavailable: 0`
- ConfigMap pour configuration APP_PORT

## Commandes de base
### Docker Local

```bash
docker build -t exam-api:local .
docker run --rm -p 3000:3000 -e APP_PORT=3000 exam-api:local
curl http://localhost:3000/health
```

### Kubernetes (Minikube)

```bash
# Démarrer
minikube start

# Déployer
kubectl apply -f k8s/

# Vérifier
kubectl get pods
kubectl get svc

# Accéder
kubectl port-forward svc/exam-api-svc 8080:80
# Dans un autre terminal :
curl http://localhost:8080/health

# Dashboard
minikube dashboard

# Arrêter
minikube stop
```

### CI/CD

Push sur `main` déclenche automatiquement :
1. Build & Tests
2. Push Image DockerHub
3. Health Check `/health`

Secrets requis dans GitHub :
- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`