# Tekton CI/CD Setup för AKS WebApp

## Installation

### 1. Installera Tekton Pipelines
```bash
# Installera Tekton Pipelines
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml

# Installera Tekton Triggers
kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml
kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml

# Installera Tekton Dashboard (valfritt)
kubectl apply --filename https://storage.googleapis.com/tekton-releases/dashboard/latest/release.yaml
```

### 2. Installera nödvändiga ClusterTasks
```bash
# Git Clone task
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.9/git-clone.yaml

# Kaniko task (för att bygga Docker images)
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/kaniko/0.6/kaniko.yaml

# Kubernetes actions task
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/kubernetes-actions/0.2/kubernetes-actions.yaml
```

### 3. Skapa Docker credentials secret för ACR
```bash
# För Azure Container Registry
kubectl create secret docker-registry acr-credentials \
  --docker-server=aksgbo.azurecr.io \
  --docker-username=<ACR_USERNAME> \
  --docker-password=<ACR_PASSWORD> \
  --namespace=default
```

### 4. Skapa GitHub webhook secret (för triggers)
```bash
kubectl create secret generic github-webhook-secret \
  --from-literal=secretToken=<YOUR_GITHUB_WEBHOOK_SECRET> \
  --namespace=default
```

### 5. Installera Tekton-resurser för projektet
```bash
# Installera RBAC
kubectl apply -f tekton/rbac.yaml

# Installera Tasks
kubectl apply -f tekton/task-helm-upgrade.yaml

# Installera Pipeline
kubectl apply -f tekton/pipeline.yaml

# Installera Triggers
kubectl apply -f tekton/triggers.yaml
```

## Kör Pipeline manuellt

```bash
# Skapa en PipelineRun
kubectl create -f - <<EOF
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: aks-webapp-run-$(date +%s)
  namespace: default
spec:
  pipelineRef:
    name: aks-webapp-pipeline
  params:
    - name: git-url
      value: https://github.com/YOUR_USERNAME/WebAppAKS.git
    - name: git-revision
      value: main
  workspaces:
    - name: shared-workspace
      volumeClaimTemplate:
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 1Gi
    - name: docker-credentials
      secret:
        secretName: acr-credentials
EOF
```

## Övervaka Pipeline

```bash
# Lista PipelineRuns
kubectl get pipelineruns

# Se detaljer för en specifik run
kubectl describe pipelinerun <pipelinerun-name>

# Se logs
tkn pipelinerun logs <pipelinerun-name> -f

# Eller via Tekton Dashboard
kubectl port-forward -n tekton-pipelines service/tekton-dashboard 9097:9097
# Öppna http://localhost:9097
```

## Konfigurera GitHub Webhook

1. Hitta EventListener URL:
```bash
kubectl get svc -n default
```

2. I ditt GitHub repo:
   - Gå till Settings → Webhooks → Add webhook
   - Payload URL: `http://<EVENTLISTENER_URL>`
   - Content type: `application/json`
   - Secret: samma som i `github-webhook-secret`
   - Events: "Just the push event"

## Tekton Dashboard

För att komma åt Tekton Dashboard:
```bash
kubectl port-forward -n tekton-pipelines service/tekton-dashboard 9097:9097
```

Öppna sedan http://localhost:9097 i din webbläsare.
