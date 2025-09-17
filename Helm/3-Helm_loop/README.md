Here are comprehensive examples of loops in Helm templates:

## 1. Basic `range` Loop Examples

### Looping through a list
```yaml
# values.yaml
services:
  - web
  - api
  - database
```

```yaml
# templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-services
data:
  {{- range .Values.services }}
  {{ . }}: "enabled"
  {{- end }}

# Output:
# web: "enabled"
# api: "enabled"  
# database: "enabled"
```

### Looping with index
```yaml
# templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-services
data:
  {{- range $index, $service := .Values.services }}
  service-{{ $index }}: {{ $service }}
  {{- end }}

# Output:
# service-0: web
# service-1: api
# service-2: database
```

## 2. Looping through Key-Value Pairs (Maps)

### values.yaml
```yaml
config:
  LOG_LEVEL: "DEBUG"
  DATABASE_URL: "postgres://localhost:5432"
  MAX_CONNECTIONS: "100"
  TIMEOUT: "30s"
```

### templates/configmap.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
  {{- range $key, $value := .Values.config }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}

# Output:
# DATABASE_URL: "postgres://localhost:5432"
# LOG_LEVEL: "DEBUG"
# MAX_CONNECTIONS: "100"
# TIMEOUT: "30s"
```

## 3. Complex Object Looping

### values.yaml
```yaml
containers:
  - name: web
    image: nginx:latest
    port: 80
    resources:
      memory: "256Mi"
      cpu: "250m"
  - name: app
    image: node:16
    port: 3000
    resources:
      memory: "512Mi"
      cpu: "500m"
  - name: cache
    image: redis:alpine
    port: 6379
    resources:
      memory: "128Mi"
      cpu: "100m"
```

### templates/deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  template:
    spec:
      containers:
      {{- range .Values.containers }}
      - name: {{ .name }}
        image: {{ .image }}
        ports:
        - containerPort: {{ .port }}
        resources:
          requests:
            memory: {{ .resources.memory }}
            cpu: {{ .resources.cpu }}
        env:
        - name: CONTAINER_NAME
          value: {{ .name }}
      {{- end }}
```

## 4. Nested Loops

### values.yaml
```yaml
environments:
  - name: production
    replicas: 3
    services:
      - name: frontend
        version: v1.2.0
      - name: backend
        version: v1.1.0
  - name: staging
    replicas: 2
    services:
      - name: frontend
        version: v1.1.0
      - name: backend
        version: v1.0.0
```

### templates/configmap.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-environments
data:
  {{- range .Values.environments }}
  # Environment: {{ .name }}
  {{- range .services }}
  {{ $.Release.Name }}-{{ .name }}-{{ $.Values.environment }}: {{ .version }}
  {{- end }}
  {{- end }}
```

## 5. Looping with Conditions

### values.yaml
```yaml
features:
  - name: logging
    enabled: true
    level: "DEBUG"
  - name: monitoring
    enabled: false
    interval: "30s"
  - name: caching
    enabled: true
    size: "100MB"
```

### templates/configmap.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-features
data:
  {{- range .Values.features }}
  {{- if .enabled }}
  {{ .name }}.enabled: "true"
  {{- if .level }}
  {{ .name }}.level: {{ .level | quote }}
  {{- end }}
  {{- if .size }}
  {{ .name }}.size: {{ .size | quote }}
  {{- end }}
  {{- end }}
  {{- end }}
```

## 6. Creating Multiple Kubernetes Resources with Loops

### values.yaml
```yaml
services:
  - name: web-service
    type: ClusterIP
    port: 80
    targetPort: 80
  - name: api-service
    type: NodePort
    port: 3000
    targetPort: 3000
  - name: db-service
    type: ClusterIP
    port: 5432
    targetPort: 5432
```

### templates/services.yaml
```yaml
{{- range .Values.services }}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .name }}
  labels:
    app: {{ .name }}
    release: {{ $.Release.Name }}
spec:
  type: {{ .type }}
  ports:
  - port: {{ .port }}
    targetPort: {{ .targetPort }}
  selector:
    app: {{ .name }}
{{- end }}
```

## 7. Advanced: Looping with Functions and Pipelines

### values.yaml
```yaml
users:
  - alice
  - bob
  - charlie
  - diana
```

### templates/configmap.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-users
data:
  # Convert to uppercase and add prefix
  {{- range .Values.users }}
  user-{{ . | upper }}: "active"
  {{- end }}

  # Join list into a single string
  all-users: {{ .Values.users | join "," }}

  # First and last user
  first-user: {{ first .Values.users }}
  last-user: {{ last .Values.users }}
```

## 8. Real-World Example: Multiple Ingress Routes

### values.yaml
```yaml
ingress:
  enabled: true
  hosts:
    - host: app.example.com
      paths:
        - path: /
          service: web
        - path: /api
          service: api
    - host: admin.example.com
      paths:
        - path: /
          service: admin
        - path: /metrics
          service: metrics
```

### templates/ingress.yaml
```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
spec:
  rules:
  {{- range .Values.ingress.hosts }}
  - host: {{ .host }}
    http:
      paths:
      {{- range .paths }}
      - path: {{ .path }}
        pathType: Prefix
        backend:
          service:
            name: {{ .service }}-service
            port:
              number: 80
      {{- end }}
  {{- end }}
{{- end }}
```

## 9. Using `range` with `include` for Reusable Templates

### templates/_helpers.tpl
```yaml
{{- define "labels" -}}
{{- range $key, $value := . }}
{{ $key }}: {{ $value | quote }}
{{- end }}
{{- end -}}
```

### templates/deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  labels:
    {{- include "labels" (dict "app" .Release.Name "env" .Values.environment "version" .Chart.Version) | nindent 4 }}
spec:
  template:
    metadata:
      labels:
        {{- include "labels" (dict "app" .Release.Name "env" .Values.environment) | nindent 8 }}
```

## 10. Error Handling in Loops

### templates/configmap.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
  {{- if .Values.config }}
  {{- range $key, $value := .Values.config }}
  {{- if $value }}
  {{ $key }}: {{ $value | quote }}
  {{- else }}
  {{ $key }}: "default-value"
  {{- end }}
  {{- end }}
  {{- else }}
  default-config: "enabled"
  {{- end }}
```

## Deployment Commands with Loop Examples

```bash
# Install with list values
helm install my-app . \
  --set services='{web,api,database}'

# Install with map values
helm install my-app . \
  --set config.LOG_LEVEL=DEBUG \
  --set config.DATABASE_URL=postgres://db:5432

# Install with complex objects
helm install my-app . \
  --set containers[0].name=web \
  --set containers[0].image=nginx \
  --set containers[0].port=80

# Test template rendering with loops
helm template my-app . --debug

# Dry-run to see what would be created
helm install my-app . --dry-run --debug
```

## Key Points to Remember:

1. **`range`** is used for iterating over lists and maps
2. **`$index`** gives the current index (0-based) in lists
3. **`$key`** and **`$value`** for map iterations
4. **`$.`** (dollar sign) accesses the root context inside loops
5. Use **`{{- `** and **` -}}`** to control whitespace in loop output
6. Always check if the value exists before looping: `{{- if .Values.list }}`
7. Combine loops with conditions for complex logic

Loops are extremely powerful in Helm for generating dynamic configuration based on your values, allowing you to create flexible and reusable charts.