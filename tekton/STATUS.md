# Tekton Pipeline Status

## ✅ Allt fungerar nu!

**Senaste lyckade körning:** `full-run-172728`

## Fungerande komponenter:

| Komponent | Status | Fil |
|-----------|--------|-----|
| **Helm Chart** | ✅ Fungerar | `helm/aks-webapp/` |
| **Tekton Pipelines** | ✅ Installerat | - |
| **Tekton Dashboard** | ✅ Installerat | - |
| **Git Clone** | ✅ Fungerar | Inline i pipeline |
| **Kaniko Build** | ✅ Fungerar | Inline i pipeline |
| **Helm Deploy** | ✅ Fungerar | Inline i pipeline |
| **GitHub Actions** | ✅ Fungerar | `.github/workflows/` |

## 🎯 Fungerande Pipeline

Den fungerande pipelinen är **`pipeline-simple.yaml`** som använder inline tasks:

```powershell
# Kör pipelinen
kubectl apply -f tekton/pipeline-simple.yaml
kubectl create -f tekton/test-pipelinerun.yaml
```

### Pipeline-steg:
1. **build-and-push** - Klonar repo, bygger Docker-image med Kaniko, pushar till ACR
2. **deploy** - Klonar repo igen, kör Helm upgrade för att deploya

## 📊 Konfiguration

### ACR
- **Registry:** `aksgbo.azurecr.io`
- **Image:** `aksgbo.azurecr.io/aks-webapp:latest`
- **Secret:** `acr-credentials`

### AKS
- **Cluster:** `akscluster`
- **Resource Group:** `aksrg`
- **App URL:** http://20.123.122.15

### GitHub
- **Repo:** https://github.com/gusbogSogeti/WebAppAKS.git
- **Branch:** `main`
- **Synlighet:** Public (krävs för Tekton utan GitHub-credentials)

## 🔧 Filer som används

| Fil | Syfte |
|-----|-------|
| `pipeline-simple.yaml` | ✅ Fungerande pipeline med inline tasks |
| `serviceaccount.yaml` | ServiceAccount med RBAC för Helm |
| `test-pipelinerun.yaml` | PipelineRun för att testa |
| `pipeline.yaml` | ⚠️ Original pipeline (har workspace-problem) |
| `task-*.yaml` | ⚠️ Separata tasks (används ej av pipeline-simple) |

## 📝 Lärdomar

1. **Workspace-mappning:** git-clone och kaniko catalog tasks har olika workspace-paths. Lösning: använd inline tasks.
2. **Kaniko script:** Kaniko-imagen har ingen shell. Lösning: använd `command`/`args` istället för `script`.
3. **RBAC för Helm:** ServiceAccount behöver permissions för serviceaccounts, roles, rolebindings.
4. **GitHub access:** Repo måste vara public eller ha SSH-keys/tokens konfigurerade.
5. **emptyDir vs PVC:** emptyDir fungerar bättre än volumeClaimTemplate för workspaces.

## 🚀 Hur man kör

```powershell
# 1. Se till att alla resurser är installerade
kubectl apply -f tekton/serviceaccount.yaml
kubectl apply -f tekton/pipeline-simple.yaml

# 2. Skapa en ny PipelineRun
kubectl create -f tekton/test-pipelinerun.yaml

# 3. Följ loggar
tkn pipelinerun logs -f --last

# 4. Eller öppna Tekton Dashboard
kubectl port-forward svc/tekton-dashboard -n tekton-pipelines 9097:9097
# Öppna http://localhost:9097
```
