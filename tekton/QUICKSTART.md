# Tekton Quickstart Guide

Denna guide visar hur du snabbt kommer igång med Tekton för AKS WebApp.

## Förutsättningar

- AKS-kluster konfigurerat (`kubectl` fungerar)
- Azure Container Registry (aksgbo.azurecr.io)
- Helm installerat

## Steg 1: Installera Tekton i AKS

```powershell
# Installera Tekton Pipelines
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml

# Vänta tills alla pods är ready
kubectl wait --for=condition=ready pod --all -n tekton-pipelines --timeout=300s

# Installera Tekton Dashboard (rekommenderat)
kubectl apply -f https://storage.googleapis.com/tekton-releases/dashboard/latest/release.yaml
```

## Steg 2: Skapa ACR Credentials Secret

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

## Steg 3: Installera Pipeline-resurser

```powershell
# Från projekt-roten
cd C:\Users\gboger\source\repos\WebAppAKS

# Installera ServiceAccount med RBAC (krävs för Helm)
kubectl apply -f tekton/serviceaccount.yaml

# Installera den fungerande pipelinen
kubectl apply -f tekton/pipeline-simple.yaml

# Verifiera
kubectl get pipelines
kubectl get serviceaccounts tekton-build-sa
```

## Steg 4: Kör Pipeline

```powershell
# Skapa en PipelineRun
kubectl create -f tekton/test-pipelinerun.yaml

# Följ loggar (kräver tkn CLI)
tkn pipelinerun logs -f --last

# Eller se status
kubectl get pipelineruns
```

## Steg 5: Öppna Dashboard (valfritt)

```powershell
# Port-forward till dashboard
kubectl port-forward svc/tekton-dashboard -n tekton-pipelines 9097:9097
```

Öppna http://localhost:9097 i webbläsaren.

## Manuell PipelineRun (alternativ)

Om du inte vill använda `test-pipelinerun.yaml`, skapa en manuellt:

```powershell
@"
apiVersion: tekton.dev/v1
kind: PipelineRun
metadata:
  generateName: manual-run-
  namespace: default
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

## Verifiera Deployment

```powershell
# Kontrollera att poddar kör
kubectl get pods -l app.kubernetes.io/name=aks-webapp

# Kontrollera service
kubectl get svc aks-webapp

# Testa appen
curl http://20.123.122.15
```

## Felsökning

### Problem: Permission denied
```powershell
# Kontrollera att ServiceAccount har rätt RBAC
kubectl get rolebindings
kubectl describe rolebinding tekton-build-sa-binding
```

### Problem: Kan inte klona repo
```powershell
# Verifiera att repo är public
git ls-remote https://github.com/gusbogSogeti/WebAppAKS.git
```

### Problem: Kan inte pusha till ACR
```powershell
# Verifiera ACR credentials
kubectl get secret acr-credentials -o jsonpath='{.data.\.dockerconfigjson}' | base64 -d
```

## Nästa steg

- Konfigurera GitHub webhooks för automatisk triggning
- Sätt upp olika miljöer (dev/staging/prod)
- Lägg till tester i pipelinen
