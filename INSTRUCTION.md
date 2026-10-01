# Kubernetes Deployment Instructions

## Prerequisites

- `kubectl` configured to access a Kubernetes cluster
- Access to the image declared in `infrastructure/todoapp-pod.yml`

If you publish the application image under a different Docker Hub account, replace the `image` value in `infrastructure/todoapp-pod.yml` before deploying.

## Apply the manifests

Create the namespace before applying the pods:

```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/todoapp-pod.yml
kubectl apply -f .infrastructure/busybox.yml
```

Wait for both pods to be running:

```bash
kubectl get pods -n todoapp -w
```

The ToDo application provides these health endpoints:

- Liveness: `GET /api/health/live/`
- Readiness: `GET /api/health/ready/`

The readiness endpoint returns `503 Not ready` for the first 40 seconds after the application starts. The pod is ready when the `READY` column shows `1/1`.

## Test with port forwarding

In one terminal, forward local port 8080 to the ToDo pod:

```bash
kubectl port-forward -n todoapp pod/todoapp 8080:8080
```

In another terminal, verify the application and its probes:

```bash
curl -i http://localhost:8080/
curl -i http://localhost:8080/api/health/live/
curl -i http://localhost:8080/api/health/ready/
```

The liveness endpoint should return `200 Alive`. After the 40-second startup period, the readiness endpoint should return `200 Ready`.

## Test from the BusyBox curl pod

Retrieve the application pod IP and make requests from the `busybox` pod:

```bash
TODOAPP_IP=$(kubectl get pod todoapp -n todoapp -o jsonpath='{.status.podIP}')
kubectl exec -n todoapp busybox -- curl -i "http://${TODOAPP_IP}:8080/api/health/live/"
kubectl exec -n todoapp busybox -- curl -i "http://${TODOAPP_IP}:8080/api/health/ready/"
```

This verifies network access from another pod in the same namespace. If readiness initially returns `503`, wait for the startup period and retry.

## Troubleshooting

```bash
kubectl describe pod todoapp -n todoapp
kubectl logs todoapp -n todoapp
kubectl get events -n todoapp --sort-by=.lastTimestamp
```
