# Tekton Quickstart Guide

## Steg 1: Installera Tekton i AKS

```powershell
# Installera Tekton Pipelines
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml

# Vänta tills alla pods är ready
kubectl wait --for=condition=ready pod --all -n tekton-pipelines --timeout=300s

# Installera Tekton Triggers (för GitHub webhooks)
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml

# Vänta på triggers
kubectl wait --for=condition=ready pod --all -n tekton-pipelines --timeout=300s

# Installera Tekton Dashboard (valfritt men rekommenderat)
kubectl apply -f https://storage.googleapis.com/tekton-releases/dashboard/latest/release.yaml
```

## Steg 2: Installera ClusterTasks

```powershell
# Git Clone task
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.9/git-clone.yaml

# Kaniko task (för Docker builds)
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/kaniko/0.6/kaniko.yaml

# Kubernetes actions
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/kubernetes-actions/0.2/kubernetes-actions.yaml
```

## Steg 3: Skapa ACR Credentials Secret

```powershell
# Hämta ACR credentials från Azure
$ACR_NAME = "aksgbo"
$ACR_USERNAME = az acr credential show --name $ACR_NAME --query username -o tsv
$ACR_PASSWORD = az acr credential show --name $ACR_NAME --query passwords[0].value -o tsv

# Skapa Kubernetes secret
kubectl create secret docker-registry acr-credentials `
  --docker-server=aksgbo.azurecr.io `
  --docker-username=$ACR_USERNAME `
  --docker-password=$ACR_PASSWORD `
  --namespace=default

# Verifiera
kubectl get secret acr-credentials
```

## Steg 4: Installera Projekt-resurser

```powershell
# Från projekt-roten
cd C:\Users\gboger\source\repos\WebAppAKS

# Installera RBAC
kubectl apply -f tekton/rbac.yaml

# Installera custom tasks
kubectl apply -f tekton/task-helm-upgrade.yaml

# Installera pipeline
kubectl apply -f tekton/pipeline.yaml

# Verifiera installation
kubectl get pipelines
kubectl get tasks
```

## Steg 5: Kör Pipeline Manuellt

```powershell
# Skapa en PipelineRun
kubectl create -f - <<EOF
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: aks-webapp-run-manual
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

Eller använd PowerShell:

```powershell
# Skapa en temporär YAML-fil
@"
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: aks-webapp-run-$(Get-Date -Format 'yyyyMMdd-HHmmss')
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
"@ | kubectl apply -f -
```

## Steg 6: Övervaka Pipeline

```powershell
# Lista alla PipelineRuns
kubectl get pipelineruns

# Se status för senaste run
kubectl get pipelineruns --sort-by=.metadata.creationTimestamp

# Se detaljer
kubectl describe pipelinerun <pipelinerun-name>

# Följ logs (kräver tkn CLI)
tkn pipelinerun logs <pipelinerun-name> -f

# Eller via kubectl
kubectl logs -l tekton.dev/pipelineRun=<pipelinerun-name> -f
```

## Steg 7: Installera Tekton CLI (tkn) - Valfritt

```powershell
# Via Chocolatey
choco install tektoncd-cli

# Eller ladda ner manuellt från
# https://github.com/tektoncd/cli/releases
```

Med tkn CLI:

```powershell
# Lista pipelines
tkn pipeline list

# Starta pipeline interaktivt
tkn pipeline start aks-webapp-pipeline --showlog

# Lista runs
tkn pipelinerun list

# Se logs
tkn pipelinerun logs -f
```

## Steg 8: Öppna Tekton Dashboard

```powershell
# Port forward till dashboard
kubectl port-forward -n tekton-pipelines service/tekton-dashboard 9097:9097

# Öppna i webbläsare
start http://localhost:9097
```

## Steg 9: Konfigurera GitHub Webhook (Valfritt)

```powershell
# Först, installera triggers
kubectl apply -f tekton/triggers.yaml

# Skapa webhook secret
kubectl create secret generic github-webhook-secret `
  --from-literal=secretToken=YOUR_SECRET_TOKEN `
  --namespace=default

# Hitta EventListener service URL
kubectl get svc -n default | Select-String eventlistener

# Använd denna URL i GitHub webhook settings
```

I GitHub repository:
1. Gå till Settings → Webhooks → Add webhook
2. Payload URL: `http://<EVENTLISTENER-IP>`
3. Content type: `application/json`
4. Secret: samma som `YOUR_SECRET_TOKEN`
5. Events: "Just the push event"

## Felsökning

```powershell
# Kontrollera Tekton pods
kubectl get pods -n tekton-pipelines

# Se events
kubectl get events --sort-by=.metadata.creationTimestamp

# Se logs för en specifik task
kubectl logs <pod-name> -c step-<step-name>

# Ta bort en PipelineRun
kubectl delete pipelinerun <pipelinerun-name>

# Ta bort alla gamla runs
kubectl delete pipelinerun --all
```

## Nästa Steg

- [ ] Testa manuell pipeline-körning
- [ ] Installera Tekton Dashboard
- [ ] Konfigurera GitHub webhooks för automatiska builds
- [ ] Anpassa pipeline-parametrar i `tekton/pipeline.yaml`
- [ ] Lägg till fler tasks (testing, security scanning, etc.)
