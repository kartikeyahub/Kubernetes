# Managing Multiple Environments with a Single Helm Chart

## Approach 1: Environment-Specific Value Files (Recommended)

## Directory Structure
```
my-chart/
├── Chart.yaml
├── values.yaml
├── values-staging.yaml
├── values-production.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    └── ingress.yaml
```

## 1. Chart.yaml
```yaml
apiVersion: v2
name: my-app
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.0.0"
```

## 2. values.yaml (Development defaults)
```yaml
# Default values - typically development environment
environment: "development"
replicaCount: 1
image:
  repository: nginx
  tag: "latest"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "200m"
    memory: "256Mi"
ingress:
  enabled: false
  host: "dev.myapp.com"
  annotations: {}
```

## 3. values-staging.yaml
```yaml
# Staging environment overrides
environment: "staging"
replicaCount: 2
image:
  repository: nginx
  tag: "staging"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
resources:
  requests:
    cpu: "250m"
    memory: "512Mi"
  limits:
    cpu: "500m"
    memory: "1Gi"
ingress:
  enabled: true
  host: "staging.myapp.com"
  annotations:
    kubernetes.io/ingress.class: "nginx"
```

## 4. values-production.yaml
```yaml
# Production environment overrides
environment: "production"
replicaCount: 3
image:
  repository: nginx
  tag: "stable"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
resources:
  requests:
    cpu: "500m"
    memory: "1Gi"
  limits:
    cpu: "1"
    memory: "2Gi"
ingress:
  enabled: true
  host: "myapp.com"
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
```

## 5. templates/deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
  labels:
    app: {{ .Release.Name }}
    environment: {{ .Values.environment }}
    chart: {{ .Chart.Name }}-{{ .Chart.Version }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
        environment: {{ .Values.environment }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: 80
        env:
        - name: ENVIRONMENT
          value: {{ .Values.environment | quote }}
        resources:
          {{- with .Values.resources.requests }}
          requests:
            {{- if .cpu }}
            cpu: {{ .cpu }}
            {{- end }}
            {{- if .memory }}
            memory: {{ .memory }}
            {{- end }}
          {{- end }}
          {{- with .Values.resources.limits }}
          limits:
            {{- if .cpu }}
            cpu: {{ .cpu }}
            {{- end }}
            {{- if .memory }}
            memory: {{ .memory }}
            {{- end }}
          {{- end }}
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

## 6. templates/service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-service
  labels:
    app: {{ .Release.Name }}
    environment: {{ .Values.environment }}
spec:
  selector:
    app: {{ .Release.Name }}
  ports:
    - protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: 80
  type: {{ .Values.service.type }}
```

## 7. templates/ingress.yaml
```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
  annotations:
    {{- with .Values.ingress.annotations }}
    {{- toYaml . | nindent 4 }}
    {{- end }}
spec:
  rules:
  - host: {{ .Values.ingress.host }}
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: {{ .Release.Name }}-service
            port:
              number: 80
{{- end }}
```

## 8. templates/_helpers.tpl
```yaml
{{/*
Common labels
*/}}
{{- define "my-app.labels" -}}
app: {{ .Release.Name }}
chart: {{ .Chart.Name }}-{{ .Chart.Version }}
release: {{ .Release.Name }}
heritage: {{ .Release.Service }}
{{- end -}}

{{/*
Environment label
*/}}
{{- define "my-app.environment" -}}
environment: {{ .Values.environment }}
{{- end -}}
```

## Deployment Commands

### For Development Environment:
```bash
# Install development environment
helm install my-app-dev ./my-chart \
  --namespace development \
  --create-namespace

# Or with explicit values file (though defaults are for dev)
helm install my-app-dev ./my-chart \
  -f values.yaml \
  --namespace development \
  --create-namespace

# Upgrade development environment
helm upgrade my-app-dev ./my-chart \
  --namespace development
```

### For Staging Environment:
```bash
# Install staging environment
helm install my-app-staging ./my-chart \
  -f values-staging.yaml \
  --namespace staging \
  --create-namespace

# Upgrade staging environment
helm upgrade my-app-staging ./my-chart \
  -f values-staging.yaml \
  --namespace staging
```

### For Production Environment:
```bash
# Install production environment
helm install my-app-production ./my-chart \
  -f values-production.yaml \
  --namespace production \
  --create-namespace

# Upgrade production environment (with dry-run first)
helm upgrade my-app-production ./my-chart \
  -f values-production.yaml \
  --namespace production \
  --dry-run

# Then actually upgrade
helm upgrade my-app-production ./my-chart \
  -f values-production.yaml \
  --namespace production
```

### Testing and Verification Commands:
```bash
# Test template rendering for different environments
helm template my-app ./my-chart -f values-staging.yaml --debug

# Check what would be installed (dry-run)
helm install my-app-staging ./my-chart \
  -f values-staging.yaml \
  --namespace staging \
  --dry-run \
  --debug

# List releases in all namespaces
helm list --all-namespaces

# Get values for a specific release
helm get values my-app-production --namespace production

# Uninstall a release
helm uninstall my-app-staging --namespace staging
```

This setup allows you to manage all three environments (dev, staging, production) with a single Helm chart, while maintaining environment-specific configurations through separate values files. The deployment commands show how to control which environment gets deployed at execution time.