# Tekton CI/CD Setup för AKS WebApp

## Översikt

Tekton är en cloud-native CI/CD-lösning som körs direkt i Kubernetes-klustret.

**Status:** ✅ Fungerar  
**App URL:** http://20.123.122.15

## Snabbstart

```powershell
# Kör pipelinen
kubectl apply -f tekton/serviceaccount.yaml
kubectl apply -f tekton/pipeline-simple.yaml
kubectl create -f tekton/test-pipelinerun.yaml

# Följ loggar
tkn pipelinerun logs -f --last
```

## Installation

### 1. Installera Tekton Pipelines

```powershell
# Installera Tekton Pipelines
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml

# Installera Tekton Dashboard (valfritt)
kubectl apply -f https://storage.googleapis.com/tekton-releases/dashboard/latest/release.yaml
```

### 2. Skapa ACR credentials secret

```powershell
# Hämta credentials
$ACR_USERNAME = az acr credential show --name aksgbo --query username -o tsv
$ACR_PASSWORD = az acr credential show --name aksgbo --query passwords[0].value -o tsv

# Skapa secret
kubectl create secret docker-registry acr-credentials `
  --docker-server=aksgbo.azurecr.io `
  --docker-username=$ACR_USERNAME `
  --docker-password=$ACR_PASSWORD
```

### 3. Installera Pipeline-resurser

```powershell
# ServiceAccount med RBAC (krävs för Helm)
kubectl apply -f tekton/serviceaccount.yaml

# Fungerande pipeline
kubectl apply -f tekton/pipeline-simple.yaml
```

## Kör Pipeline

### Med test-pipelinerun.yaml

```powershell
kubectl create -f tekton/test-pipelinerun.yaml
```

### Manuellt

```powershell
@"
apiVersion: tekton.dev/v1
kind: PipelineRun
metadata:
  generateName: aks-webapp-run-
spec:
  pipelineRef:
    name: aks-webapp-simple
  params:
    - name: git-url
      value: "https://github.com/gusbogSogeti/WebAppAKS.git"
    - name: git-revision
      value: "main"
    - name: image-url
      value: "aksgbo.azurecr.io/aks-webapp"
    - name: image-tag
      value: "latest"
  workspaces:
    - name: shared-workspace
      emptyDir: {}
    - name: docker-credentials
      secret:
        secretName: acr-credentials
"@ | kubectl create -f -
```

## Övervaka Pipeline

```powershell
# Lista PipelineRuns
kubectl get pipelineruns

# Se loggar för senaste körning
tkn pipelinerun logs -f --last

# Öppna Dashboard
kubectl port-forward svc/tekton-dashboard -n tekton-pipelines 9097:9097
# Gå till http://localhost:9097
```

## Pipeline-struktur

### pipeline-simple.yaml (Rekommenderad)

Den fungerande pipelinen använder **inline tasks** för att undvika workspace-problem:

```
┌─────────────────────────────────────────────────────────────────┐
│                    aks-webapp-simple                            │
├─────────────────────────────────────────────────────────────────┤
│  1. build-and-push                                              │
│     ├── Init: git clone                                         │
│     └── Step: kaniko build & push                               │
│                                                                 │
│  2. deploy (runAfter: build-and-push)                           │
│     ├── Step 1: git clone                                       │
│     └── Step 2: helm upgrade --install                          │
└─────────────────────────────────────────────────────────────────┘
```

### Filer

| Fil | Beskrivning | Status |
|-----|-------------|--------|
| `pipeline-simple.yaml` | Huvudpipeline med inline tasks | ✅ Fungerar |
| `serviceaccount.yaml` | ServiceAccount + RBAC för Helm | ✅ Krävs |
| `test-pipelinerun.yaml` | Färdig PipelineRun för test | ✅ Fungerar |
| `pipeline.yaml` | Original pipeline | ⚠️ Workspace-problem |
| `task-*.yaml` | Separata tasks | ⚠️ Används ej |
| `triggers.yaml` | GitHub webhooks | ⚠️ Ej testat |

## Felsökning

### Pod har inte rätt permissions

```powershell
# Kontrollera RBAC
kubectl describe rolebinding tekton-build-sa-binding
```

### Kan inte klona från GitHub

GitHub-repot måste vara **public** eller ha credentials konfigurerade.

### Kaniko hittar inte Dockerfile

Pipelinen klonar till `/workspace/source/AksWebApp/` och Dockerfile finns där.

### Helm kan inte skapa resurser

ServiceAccount behöver permissions för:
- pods, services, configmaps, secrets
- deployments, replicasets, statefulsets
- networkpolicies, horizontalpodautoscalers
- serviceaccounts, roles, rolebindings

## Nästa steg

1. **GitHub Webhooks** - Automatisk triggning vid push
2. **Flera miljöer** - Dev/Staging/Production
3. **Tester** - Lägg till test-steg i pipelinen
4. **Notifications** - Slack/Teams-integration

## Resurser

- [Tekton Documentation](https://tekton.dev/docs/)
- [Tekton Hub](https://hub.tekton.dev/)
- [Kaniko](https://github.com/GoogleContainerTools/kaniko)
