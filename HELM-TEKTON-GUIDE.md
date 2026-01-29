# WebAppAKS - Guide för Helm och Tekton

Detta projekt innehåller nu stöd för både **Helm Charts** och **Tekton Pipelines**.

**Status:** ✅ Allt fungerar  
**App URL:** http://20.123.122.15

## 📦 Helm Charts

Helm är ett pakethanteringsverktyg för Kubernetes som gör det enkelt att deploya och hantera applikationer.

### Vad har skapats?
- `helm/aks-webapp/` - Komplett Helm chart för applikationen
- `helm/aks-webapp/values.yaml` - Konfigurationsfil
- `helm/aks-webapp/templates/` - Kubernetes manifests som templates

### Status
- ✅ Helm 4.1.0 installerat
- ✅ Chart deployat (revision 4+)
- ✅ App kör på http://20.123.122.15

### Snabbstart med Helm:
```powershell
# Installera chartet
helm install aks-webapp ./helm/aks-webapp

# Uppgradera med ny version
helm upgrade aks-webapp ./helm/aks-webapp --set image.tag=v2.0.0

# Rollback om något går fel
helm rollback aks-webapp
```

📖 **Läs mer:** [helm/README.md](helm/README.md)

## 🔄 Tekton Pipelines

Tekton är en cloud-native CI/CD-lösning som körs direkt i Kubernetes-klustret.

### Vad har skapats?
- `tekton/pipeline-simple.yaml` - ✅ Fungerande pipeline (inline tasks)
- `tekton/serviceaccount.yaml` - ServiceAccount med RBAC för Helm
- `tekton/test-pipelinerun.yaml` - Färdig PipelineRun för test

### Status
- ✅ Tekton Pipelines installerat
- ✅ Tekton Dashboard installerat
- ✅ Pipeline fungerar (build + deploy)
- ✅ ACR-push fungerar
- ✅ Helm deploy fungerar

### Pipeline-steg:
1. **build-and-push** - Klonar Git-repo, bygger Docker-image med Kaniko, pushar till ACR
2. **deploy** - Klonar repo, kör Helm upgrade

### Snabbstart med Tekton:
```powershell
# Installera resurser
kubectl apply -f tekton/serviceaccount.yaml
kubectl apply -f tekton/pipeline-simple.yaml

# Kör pipeline
kubectl create -f tekton/test-pipelinerun.yaml

# Följ loggar
tkn pipelinerun logs -f --last
```

📖 **Läs mer:** [tekton/README.md](tekton/README.md)  
📖 **Quickstart:** [tekton/QUICKSTART.md](tekton/QUICKSTART.md)  
📖 **Status:** [tekton/STATUS.md](tekton/STATUS.md)

## 🔀 Jämförelse: Helm vs Tekton vs GitHub Actions

| Aspekt | GitHub Actions | Helm | Tekton |
|--------|---------------|------|--------|
| **Var körs det?** | GitHub's servrar | Lokalt/CI | I Kubernetes |
| **Användning** | CI/CD workflow | Deployment | CI/CD pipeline |
| **Kostnad** | Gratis tier sedan betalt | Gratis | Gratis (betalar för K8s) |
| **Komplexitet** | Låg | Medel | Hög |
| **Kubernetes-native** | Nej | Ja | Ja |
| **Best for** | Enkel CI/CD | App deployment | Cloud-native CI/CD |

## 🎯 Rekommenderade användningsfall

### Använd GitHub Actions när:
- Du vill ha enkel setup och komma igång snabbt
- Du redan använder GitHub
- Du inte har komplicerade deployment-behov

### Använd Helm när:
- Du behöver deploya till flera miljöer (dev/staging/prod)
- Du vill ha versionering och rollback
- Du behöver återanvändbara konfigurationer

### Använd Tekton när:
- Du vill ha hela CI/CD direkt i Kubernetes
- Du behöver cloud-native lösning
- Du vill undvika externa CI/CD-tjänster

## 📁 Projektstruktur

```
WebAppAKS/
├── AksWebApp/                    # .NET applikation
│   ├── Dockerfile
│   ├── aks-deployment.yaml       # Original K8s manifest
│   └── network-policy.yaml
├── helm/                         # 📦 Helm Charts
│   ├── README.md
│   └── aks-webapp/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── deployment.yaml
│           ├── service.yaml
│           ├── networkpolicy.yaml
│           └── hpa.yaml
├── tekton/                       # 🔄 Tekton Pipelines
│   ├── README.md
│   ├── QUICKSTART.md
│   ├── STATUS.md
│   ├── pipeline-simple.yaml      # ✅ Fungerande pipeline
│   ├── serviceaccount.yaml       # RBAC för Helm
│   ├── test-pipelinerun.yaml     # Test PipelineRun
│   ├── pipeline.yaml             # Original (workspace-problem)
│   ├── task-*.yaml               # Separata tasks
│   └── triggers.yaml
└── .github/workflows/
    ├── deploy.yml                # Original workflow
    └── deploy-helm.yml           # Helm-baserad workflow
```

## 🔧 Konfiguration

### ACR
- **Registry:** aksgbo.azurecr.io
- **Image:** aksgbo.azurecr.io/aks-webapp:latest
- **Secret:** acr-credentials

### AKS
- **Cluster:** akscluster
- **Resource Group:** aksrg

### GitHub
- **Repo:** https://github.com/gusbogSogeti/WebAppAKS.git
- **Synlighet:** Public (krävs för Tekton utan credentials)

## 🚀 Deployment Options

### Option 1: GitHub Actions + Helm (Enklast)
```powershell
# Push till main-branch triggar automatisk deploy
git push origin main
```

### Option 2: Manuell Helm
```powershell
helm upgrade --install aks-webapp ./helm/aks-webapp
```

### Option 3: Tekton Pipeline (Cloud-native)
```powershell
kubectl create -f tekton/test-pipelinerun.yaml
```

## 📚 Resurser

- [Helm Documentation](https://helm.sh/docs/)
- [Tekton Documentation](https://tekton.dev/docs/)
- [Tekton Hub](https://hub.tekton.dev/)
- [AKS Best Practices](https://learn.microsoft.com/azure/aks/)
