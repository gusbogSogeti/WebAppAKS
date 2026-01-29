# Helm Chart för AKS WebApp

Detta är ett Helm chart för att deploya AKS WebApp till Kubernetes.

## Installation

### Förutsättningar
- Helm 3.x installerat
- kubectl konfigurerad mot ditt AKS cluster
- Tillgång till Azure Container Registry (ACR)

### Installera Helm (om inte redan installerat)
```bash
# Windows (PowerShell)
choco install kubernetes-helm

# Eller via winget
winget install Helm.Helm
```

### Deploya med Helm

#### 1. Validera chartet
```bash
# Från root-katalogen av projektet
helm lint helm/aks-webapp
```

#### 2. Dry-run för att se vad som kommer skapas
```bash
helm install aks-webapp helm/aks-webapp --dry-run --debug
```

#### 3. Installera chartet
```bash
# Standard installation
helm install aks-webapp helm/aks-webapp

# Med custom namespace
helm install aks-webapp helm/aks-webapp --namespace production --create-namespace

# Med custom values
helm install aks-webapp helm/aks-webapp -f custom-values.yaml
```

#### 4. Uppgradera en befintlig release
```bash
helm upgrade aks-webapp helm/aks-webapp

# Med specifik image tag
helm upgrade aks-webapp helm/aks-webapp --set image.tag=v1.2.3

# Eller om du vill installera om helt
helm upgrade --install aks-webapp helm/aks-webapp
```

## Konfiguration

Alla konfigurationsvärden finns i `values.yaml`. Här är de viktigaste:

### Image-konfiguration
```yaml
image:
  registry: aksgbo.azurecr.io
  repository: aks-webapp
  tag: "latest"  # Ändra till specifik version i produktion
```

### Replicas och autoscaling
```yaml
replicaCount: 2

autoscaling:
  enabled: false  # Sätt till true för att aktivera HPA
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

### Resources
```yaml
resources:
  limits:
    cpu: 500m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

### Service-typ
```yaml
service:
  type: LoadBalancer  # Eller ClusterIP, NodePort
  port: 80
  targetPort: 8080
```

## Använda custom values

Skapa en `custom-values.yaml` fil:

```yaml
replicaCount: 3

image:
  tag: "v1.0.0"

resources:
  limits:
    cpu: 1000m
    memory: 512Mi
  requests:
    cpu: 200m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 15
```

Använd sedan:
```bash
helm install aks-webapp helm/aks-webapp -f custom-values.yaml
```

## Miljöspecifika deployer

### Development
```bash
helm install aks-webapp-dev helm/aks-webapp \
  --namespace dev \
  --create-namespace \
  --set replicaCount=1 \
  --set image.tag=dev-latest
```

### Staging
```bash
helm install aks-webapp-staging helm/aks-webapp \
  --namespace staging \
  --create-namespace \
  --set replicaCount=2 \
  --set image.tag=staging-v1.0.0
```

### Production
```bash
helm install aks-webapp-prod helm/aks-webapp \
  --namespace production \
  --create-namespace \
  --set replicaCount=3 \
  --set autoscaling.enabled=true \
  --set image.tag=v1.0.0
```

## Hantera releases

```bash
# Lista installerade releases
helm list

# Se status för en release
helm status aks-webapp

# Se historik
helm history aks-webapp

# Rollback till föregående version
helm rollback aks-webapp

# Rollback till specifik revision
helm rollback aks-webapp 2

# Ta bort en release
helm uninstall aks-webapp

# Ta bort men behåll historiken
helm uninstall aks-webapp --keep-history
```

## Testa chartet

```bash
# Kör alla tester
helm test aks-webapp

# Template ut filerna för inspektion
helm template aks-webapp helm/aks-webapp > output.yaml
```

## Paketeera chartet

```bash
# Skapa ett chart package
helm package helm/aks-webapp

# Detta skapar: aks-webapp-0.1.0.tgz
```

## Integration med GitHub Actions

För att integrera med din befintliga GitHub Actions workflow, uppdatera `.github/workflows/deploy.yml`:

```yaml
- name: Deploy with Helm
  run: |
    helm upgrade --install aks-webapp ./helm/aks-webapp \
      --namespace production \
      --create-namespace \
      --set image.tag=${{ github.sha }} \
      --wait \
      --timeout 5m
```

## Felsökning

```bash
# Se alla resurser som skapats av Helm
kubectl get all -l app.kubernetes.io/instance=aks-webapp

# Se events
kubectl get events --sort-by=.metadata.creationTimestamp

# Debugga en release
helm get all aks-webapp

# Se vilka values som används
helm get values aks-webapp

# Se alla values (inklusive defaults)
helm get values aks-webapp --all
```

## Best Practices

1. **Använd specifika image tags** i produktion, inte `latest`
2. **Aktivera autoscaling** för produktion
3. **Sätt resource limits** för att undvika resource exhaustion
4. **Använd namespaces** för att separera miljöer
5. **Versionshantera dina custom values-filer**
6. **Använd `--dry-run`** innan du deployar till produktion
