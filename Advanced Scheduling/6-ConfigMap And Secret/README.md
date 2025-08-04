# Kubernetes ConfigMap and Secret with YAML and Real-Time Use Cases

This document provides an overview of Kubernetes ConfigMaps and Secrets, their YAML examples, and real-time use cases. It also includes sample Pod and Deployment manifests to demonstrate their usage.

---

## **Table of Contents**
1. [ConfigMap](#configmap)
   - [YAML Example](#configmap-yaml)
   - [Real-Time Use Cases](#configmap-use-cases)
2. [Secret](#secret)
   - [YAML Example](#secret-yaml)
   - [Real-Time Use Cases](#secret-use-cases)
3. [Sample Pod Manifest](#pod-manifest)
4. [Sample Deployment Manifest](#deployment-manifest)

---

## **ConfigMap**
A **ConfigMap** is used to store non-sensitive configuration data in key-value pairs. It decouples configuration from application code, making it easier to manage and update.

### **YAML Example** {#configmap-yaml}
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: "blue"
  LOG_LEVEL: "info"
  DATABASE_URL: "postgres://user:password@db:5432/mydb"
```

### **Real-Time Use Cases** {#configmap-use-cases}
- **Environment Variables**: Inject environment variables into pods.
- **Configuration Files**: Mount ConfigMap data as files in a pod.
- **Application Settings**: Store application-specific settings like feature flags or API endpoints.

---

## **Secret**
A **Secret** is used to store sensitive information such as passwords, tokens, or keys. Secrets are encoded in base64 and can be encrypted at rest.

### **YAML Example** {#secret-yaml}
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  # Base64 encoded values
  username: dXNlcm5hbWU=  # username
  password: cGFzc3dvcmQ=  # password
```

### **Real-Time Use Cases** {#secret-use-cases}
- **Database Credentials**: Store database usernames and passwords.
- **TLS Certificates**: Store SSL/TLS certificates for secure communication.
- **API Tokens**: Store API keys or tokens for external services.

---

## **Sample Pod Manifest** {#pod-manifest}
This Pod manifest demonstrates how to use ConfigMap and Secret in a Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
spec:
  containers:
    - name: app-container
      image: nginx:latest
      env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_COLOR
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

---

## **Sample Deployment Manifest** {#deployment-manifest}
This Deployment manifest demonstrates how to use ConfigMap and Secret in a Deployment.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: app-container
          image: nginx:latest
          env:
            - name: APP_COLOR
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: APP_COLOR
            - name: DB_USERNAME
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: username
          volumeMounts:
            - name: config-volume
              mountPath: /etc/config
      volumes:
        - name: config-volume
          configMap:
            name: app-config
```

---

## **Best Practices**
1. **Avoid Hardcoding**: Use ConfigMaps and Secrets to avoid hardcoding configuration data in application code.
2. **Access Control**: Restrict access to Secrets using Kubernetes RBAC.
3. **Immutable Updates**: Use immutable ConfigMaps and Secrets for critical configurations to prevent accidental changes.
4. **Environment-Specific Configs**: Create separate ConfigMaps and Secrets for different environments (e.g., dev, prod).

---

This README.md provides a comprehensive guide to using ConfigMaps and Secrets in Kubernetes, along with practical examples. Feel free to adapt the YAML manifests to your specific use case.