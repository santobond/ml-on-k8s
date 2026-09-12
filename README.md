# ML Model Serving on Kubernetes

End-to-end ML model deployment on Kubernetes — containerized inference service with health checks, autoscaling-ready architecture, and automated self-healing via Kubernetes Deployments.

## What this project demonstrates

- Training and serializing a scikit-learn model
- Wrapping a model in a FastAPI inference service
- Containerizing the service with Docker
- Deploying to Kubernetes with health checks, resource limits, and replicas
- Verifying traffic routing through a Kubernetes Service
- Observing Kubernetes' self-healing (Deployment reconciliation)

## Architecture

```
train.py → model.joblib → app.py (FastAPI) → Docker image
→ Kubernetes Deployment (2 replicas, readiness probe, resource limits)
→ Kubernetes Service (ClusterIP)
```

## Stack

- **Model:** scikit-learn (RandomForestClassifier, trained on the Iris dataset)
- **API:** FastAPI + Uvicorn
- **Container:** Docker
- **Orchestration:** Kubernetes (tested on kind)

## Project structure

```
ml-on-k8s/
├── train.py          # Trains and saves the model
├── model.joblib       # Serialized model
├── app.py             # FastAPI inference service
├── Dockerfile
└── k8s/
    ├── deployment.yaml
    └── service.yaml
```

## Running locally

```bash
python train.py
uvicorn app:app --reload
curl -X POST localhost:8000/predict -H "Content-Type: application/json" -d "[5.1, 3.5, 1.4, 0.2]"
```

## Running on Kubernetes (kind)

```bash
docker build -t ml-demo:v1 .
kind create cluster --name mlops-demo
kind load docker-image ml-demo:v1 --name mlops-demo
kubectl apply -f k8s/
kubectl port-forward svc/ml-demo 8080:80
curl -X POST localhost:8080/predict -H "Content-Type: application/json" -d "[5.1, 3.5, 1.4, 0.2]"
```

## Design notes

- `imagePullPolicy: IfNotPresent` is required since the image is loaded locally into
  kind rather than pulled from a registry.
- The readiness probe on `/healthz` ensures Kubernetes doesn't route traffic to a
  pod before the model has finished loading.
- Resource requests/limits prevent a single replica from starving cluster resources.

## Next steps

- [ ] Serve via KServe for autoscaling and standardized `InferenceService` API
- [ ] Add GPU-backed inference for a larger model
- [ ] Add a RAG pipeline with an in-cluster vector database
