# WebAppAKS - Guide för Helm och Tekton

Detta projekt innehåller nu stöd för både **Helm Charts** och **Tekton Pipelines**.

## 📦 Helm Charts

Helm är ett pakethanteringsverktyg för Kubernetes som gör det enkelt att deploya och hantera applikationer.

### Vad har skapats?
- `helm/aks-webapp/` - Komplett Helm chart för applikationen
- `helm/aks-webapp/values.yaml` - Konfigurationsfil
- `helm/aks-webapp/templates/` - Kubernetes manifests som templates

### Fördelar med Helm:
- ✅ Återanvändbar konfiguration
- ✅ Enkel hantering av olika miljöer (dev/staging/prod)
- ✅ Versionering och rollback-möjligheter
- ✅ Templating för dynamiska värden
- ✅ Paketerad distribution

### Snabbstart med Helm:
```bash
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
- `tekton/pipeline.yaml` - Huvudpipeline för build och deploy
- `tekton/task-helm-upgrade.yaml` - Custom task för Helm deployment
- `tekton/triggers.yaml` - GitHub webhook integration
- `tekton/rbac.yaml` - Säkerhetsinställningar

### Pipeline-steg:
1. **Fetch Repository** - Klonar Git-repo
2. **Build & Push Image** - Bygger Docker-image med Kaniko
3. **Deploy with Helm** - Deployar med Helm chart
4. **Verify Deployment** - Kontrollerar att deployment lyckades

### Fördelar med Tekton:
- ✅ Körs native i Kubernetes
- ✅ Ingen extern CI/CD-server behövs
- ✅ Skalbar och molnagnostisk
- ✅ Reusable tasks från Tekton Catalog
- ✅ GitOps-friendly

### Snabbstart med Tekton:
```bash
# Installera Tekton
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml

# Installera projekt-resurser
kubectl apply -f tekton/

# Kör pipeline
tkn pipeline start aks-webapp-pipeline --showlog
```

📖 **Läs mer:** [tekton/README.md](tekton/README.md)

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
- Du inte har komplicerata deployment-behov
- Du vill minimal overhead

### Använd Helm när:
- Du behöver deploya till flera miljöer (dev/staging/prod)
- Du vill ha versionering och rollback
- Du behöver återanvändbara konfigurationer
- Du vill separera deployment från CI

### Använd Tekton när:
- Du vill ha hela CI/CD direkt i Kubernetes
- Du behöver cloud-native lösning
- Du vill undvika externa CI/CD-tjänster
- Du arbetar med multi-cloud eller on-premise

## 🚀 Kombinera lösningarna

Du kan också kombinera dessa verktyg:

### Option 1: GitHub Actions + Helm
```yaml
# .github/workflows/deploy-helm.yml (redan skapad!)
- name: Deploy with Helm
  run: |
    helm upgrade --install aks-webapp ./helm/aks-webapp \
      --set image.tag=${{ github.sha }}
```

### Option 2: Tekton + Helm
Tekton pipeline använder redan Helm för deployment (se `tekton/pipeline.yaml`).

### Option 3: GitHub Actions triggar Tekton
```yaml
- name: Trigger Tekton Pipeline
  run: |
    tkn pipeline start aks-webapp-pipeline \
      --param git-revision=${{ github.sha }}
```

## 📁 Projektstruktur

```
WebAppAKS/
├── AksWebApp/                    # .NET applikation
│   ├── Dockerfile
│   ├── aks-deployment.yaml       # Original K8s manifest
│   └── network-policy.yaml
├── helm/                         # 📦 Helm Charts (NYT!)
│   ├── README.md
│   └── aks-webapp/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── deployment.yaml
│           ├── service.yaml
│           ├── networkpolicy.yaml
│           └── hpa.yaml
├── tekton/                       # 🔄 Tekton Pipelines (NYT!)
│   ├── README.md
│   ├── pipeline.yaml
│   ├── task-helm-upgrade.yaml
│   ├── triggers.yaml
│   └── rbac.yaml
└── .github/workflows/
    ├── deploy.yml                # Original workflow
    └── deploy-helm.yml           # Helm-baserad workflow (NYT!)
```

## 🔧 Nästa steg

1. **Testa Helm lokalt:**
   ```bash
   helm install aks-webapp ./helm/aks-webapp --dry-run --debug
   ```

2. **Installera Tekton i ditt AKS-kluster:**
   ```bash
   kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
   ```

3. **Välj din deployment-strategi:**
   - Fortsätt med GitHub Actions (enklast)
   - Byt till GitHub Actions + Helm (rekommenderat)
   - Gå full cloud-native med Tekton

4. **Konfigurera secrets:**
   - ACR credentials
   - GitHub webhook tokens (för Tekton)

## 📚 Resurser

- [Helm Documentation](https://helm.sh/docs/)
- [Tekton Documentation](https://tekton.dev/docs/)
- [Tekton Catalog](https://hub.tekton.dev/)
- [AKS Best Practices](https://learn.microsoft.com/azure/aks/)

## 🤝 Bidra

Har du förbättringsförslag? Skapa en PR eller öppna en issue!
