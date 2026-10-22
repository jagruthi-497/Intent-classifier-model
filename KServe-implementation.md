# Kserve Demonstration for Iris model
# Check current cluster context
```kubectl config current-context```

# View current context details and active namespace
```kubectl config get-contexts $(kubectl config current-context) ```
### Install Cert Manager

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
```

### Install KServe CRDs

```
kubectl create namespace kserve

helm install kserve-crd oci://ghcr.io/kserve/charts/kserve-crd \
  --version v0.16.0 \
  -n kserve \
  --wait
```

### Install KServe controller

```
helm install kserve oci://ghcr.io/kserve/charts/kserve \
  --version v0.16.0 \
  -n kserve \
  --set kserve.controller.deploymentMode=RawDeployment \
  --wait
```

### The default KServe cluster installation lacked the ClusterServingRuntime configuration for sklearn models (which relies on mlserver).
### Resolution when running on free k8 platforms like killercode
### Apply the official KServe v0.11 runtimes manifest to install the required mlserver runtime environment:

```
kubectl apply -f [https://raw.githubusercontent.com/kserve/kserve/release-0.11/install/v0.11
```
### Deploy the Intent Classifier model

```
kubectl create namespace intent

cat <<EOF | kubectl apply -n intent -f -
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: intent-classifier
spec:
  predictor:
    model:
      modelFormat:
        name: sklearn
      storageUri: "<downloadable location>"
      resources:
        requests:
          cpu: "100m"
          memory: "512Mi"
        limits:
          cpu: "1"
          memory: "1Gi"
EOF

kubectl get inferenceservice intent-classifier -n intent
```
# List all pods in the intent namespace
```kubectl get pods -n intent```

# Detailed information about a specific pod
```kubectl describe pod <pod-name> -n intent ```

# List services in the namespace
```kubectl get svc -n intent ```
# this give the anme of the svc
# View predictor container logs to monitor model loading
``` kubectl logs <pod-name> -n intent -c kserve-container --tail=50```
### Port-forward to access the model

```
kubectl -n ml port-forward svc/<svc-name> 8080:80
```

### Inference the Model

```
curl -s -X POST http://localhost:8080/v1/models/intent-classifier:predict \
  -H "Content-Type: application/json" \
  -d '{"instances":["I want to cancel my subscription"]}' | jq
```



