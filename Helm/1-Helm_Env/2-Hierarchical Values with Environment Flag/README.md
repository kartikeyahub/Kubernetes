# Managing Multiple Environments with a Single Helm Chart

## Approach 2: Hierarchical Values with Environment Flag

### values.yaml
```yaml
# Base values with environment-specific sections
environments:
  development:
    replicaCount: 1
    resources: 
      requests: {cpu: "100m", memory: "128Mi"}
    ingress: {enabled: false}
  
  staging:
    replicaCount: 2
    resources: 
      requests: {cpu: "250m", memory: "512Mi"}
    ingress: {enabled: true, host: "staging.myapp.com"}
  
  production:
    replicaCount: 3
    resources: 
      requests: {cpu: "500m", memory: "1Gi"}
    ingress: {enabled: true, host: "myapp.com"}

# Default environment
currentEnvironment: "development"
```

### Template Usage
```yaml
{{- $env := .Values.currentEnvironment -}}
{{- $config := index .Values.environments $env -}}

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ $config.replicaCount }}
  template:
    spec:
      containers:
      - name: app
        resources:
          requests:
            cpu: {{ $config.resources.requests.cpu }}
            memory: {{ $config.resources.requests.memory }}
        env:
        - name: ENVIRONMENT
          value: {{ $env }}
---
{{- if $config.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
  {{- with $config.ingress.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
  rules:
  - host: {{ $config.ingress.host }}
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

## How to Control at Execution Time


### Method 2: Using --set to Override Environment
```bash
# Development
helm install my-app ./my-chart --set currentEnvironment=development

# Staging
helm install my-app ./my-chart --set currentEnvironment=staging

# Production
helm install my-app ./my-chart --set currentEnvironment=production

# With additional overrides
helm install my-app ./my-chart \
  --set currentEnvironment=production \
  --set environments.production.replicaCount=5
```

### Method 3: Combined Approach (Most Flexible)
```bash
# Base values from file + runtime overrides
helm install my-app ./my-chart \
  -f values-production.yaml \
  --set replicaCount=5 \
  --set image.tag="v2.1.0"
```

### Method 4: Using Environment Variables
```bash
# In your CI/CD pipeline
export ENVIRONMENT="production"
helm install my-app ./my-chart \
  -f values-${ENVIRONMENT}.yaml \
  --set image.tag="${CI_COMMIT_TAG}"
```

## Advanced: Conditional Templates Based on Environment

```yaml
{{- if eq .Values.environment "production" }}
# Production-only resources
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: {{ .Release.Name }}-monitor
spec:
  endpoints:
  - port: web
    interval: 30s
{{- end }}

{{- if or (eq .Values.environment "staging") (eq .Values.environment "production") }}
# Staging and Production resources
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ .Release.Name }}-hpa
spec:
  minReplicas: {{ .Values.replicaCount }}
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
{{- end }}
```

## Best Practices

1. **Keep defaults safe**: Set development defaults in `values.yaml`
2. **Use descriptive names**: `values-production.yaml`, not `values-prod.yaml`
3. **Test all environments**: Use `helm template` to verify each environment
4. **Version control**: Store all value files in version control
5. **CI/CD integration**: Use environment variables in your pipeline

```bash
# Test rendering before deployment
helm template my-app ./my-chart -f values-production.yaml --debug
```

This approach gives you complete control over which environment configuration is applied at execution time while maintaining a single, maintainable chart codebase.