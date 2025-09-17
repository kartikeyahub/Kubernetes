# Kubernetes

Yes, it's absolutely possible and actually a very common practice to manage multiple environments (dev, staging, production) with a single Helm chart. This is one of Helm's key strengths. Here's how to control it at execution time:

# Managing Multiple Environments with a Single Helm Chart

## Approach 1: Environment-Specific Value Files (Recommended)

### Directory Structure
```
my-chart/
├── values.yaml          # Default values (usually development)
├── values-staging.yaml   # Staging environment overrides
├── values-production.yaml # Production environment overrides
├── charts/
├── templates/
│   └── deployment.yaml
└── Chart.yaml
```


























### Method 1: Using Different Value Files
```bash
# Development (uses default values.yaml)
helm install my-app ./my-chart

# Staging
helm install my-app ./my-chart -f values-staging.yaml

# Production
helm install my-app ./my-chart -f values-production.yaml

# Upgrade with different environment
helm upgrade my-app ./my-chart -f values-production.yaml
```
