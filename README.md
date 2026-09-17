# Kubernetes Web Application with MongoDB
 
## 📌 Project Overview

This project is a hands-on Kubernetes deployment demo 

The objective is to deploy a containerized web application on a Kubernetes cluster and connect it to a MongoDB database.

The WebApp is provided as a Docker image. Kubernetes pulls the image from the container registry and deploys it inside a WebApp Pod. The application is exposed through a Kubernetes Service, which provides a stable endpoint for accessing the WebApp.

The WebApp communicates with MongoDB through a dedicated MongoDB Service. Kubernetes Service discovery allows the WebApp to reach the MongoDB Pod using the Service name rather than relying on the Pod's IP address.

Application configuration is externalized using a ConfigMap, while sensitive database credentials are stored in a Secret and injected into the WebApp Pod.

## 🏗️ Architecture

```text
                         User
                           │
                           ▼
                 ┌──────────────────┐
                 │   WebApp Service │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │    WebApp Pod    │
                 │                  │
                 │  ┌────────────┐  │
                 │  │ WebApp     │  │
                 │  │ Container  │  │
                 │  └────────────┘  │
                 └────────┬─────────┘
                          │
                          │ MongoDB connection
                          │
                          ▼
                 ┌──────────────────┐
                 │ MongoDB Service  │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │   MongoDB Pod    │
                 │                  │
                 │ ┌──────────────┐ │
                 │ │   MongoDB    │ │
                 │ │  Container   │ │
                 │ └──────────────┘ │
                 └──────────────────┘


                            WebApp configuration
                         ─────────────────────────
                                ↙           ↘
              ┌──────────────────┐          ┌──────────────────┐
              │    ConfigMap     │          │    secret        │
              └──────────────────┘          └──────────────────┘
```
## 🛠️ Technologies
- Kubernetes
- Docker
- kubectl
- Minikube
- YAML
- Git / GitHub

##  📚 Kubernetes Concepts & Notes

This section summarizes the main Kubernetes concepts used in this project and the relationships between them.

### 1. Pod

A **Pod** is the smallest deployable unit in Kubernetes. It runs one or more containers that share the same network and storage context.

In this project, the application and MongoDB are each deployed in their own Pod.

```text
WebApp Pod
└── WebApp Container

MongoDB Pod
└── MongoDB Container
```

A Pod is not created from an image. Instead, the **container inside the Pod** is created from a container image.

---

### 2. Container

A **container** is a running instance of a container image.

The image contains the application and its dependencies, while the container is the running instance created from that image.

```text
Container Image
      ↓
   Container
      ↓
     Pod
```

In this project, the WebApp container runs the application image, while the MongoDB container runs the MongoDB image.

---

### 3. Deployment

A **Deployment** manages the desired state of application Pods.

It allows Kubernetes to:

* Create Pods
* Maintain the desired number of replicas
* Replace failed Pods
* Perform rolling updates
* Manage ReplicaSets

For example:

```yaml
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
```

The Deployment specifies that one WebApp Pod should be running.

---

### 4. ReplicaSet

A **ReplicaSet** ensures that the specified number of Pod replicas are running.

A Deployment normally creates and manages a ReplicaSet, which in turn creates and maintains the Pods.

```text
Deployment
     ↓
 ReplicaSet
     ↓
   Pod(s)
     ↓
 Container(s)
```

The ReplicaSet is usually not managed directly when using Deployments.

---

### 5. Labels and Selectors

**Labels** are key-value pairs attached to Kubernetes resources.

A **selector** is used to identify resources based on their labels.

For example:

```yaml
selector:
  matchLabels:
    app: webapp

template:
  metadata:
    labels:
      app: webapp
```

The Deployment uses `matchLabels` to identify the Pods it manages.

Labels also allow Pods to be queried:

```bash
kubectl get pods -l app=webapp
```

---

### 6. Services

A **Service** provides a stable network endpoint for accessing a group of Pods.

Pods are ephemeral, and their IP addresses can change. A Service provides a stable way for other components to communicate with them.

This project uses two Services:

#### WebApp Service

The **WebApp Service** exposes the WebApp and provides an entry point to access the application.

```text
User
  ↓
WebApp Service
  ↓
WebApp Pod
  ↓
WebApp Container
```

Depending on the Service type, the WebApp can be accessed from outside the Kubernetes cluster.

For this local Minikube project, the WebApp Service can be accessed using:

```bash
minikube service <webapp-service-name>
```

#### MongoDB Service

The **MongoDB Service** provides a stable internal endpoint that allows the WebApp Pod to communicate with the MongoDB Pod.

```text
WebApp Pod
    ↓
MongoDB Service
    ↓
MongoDB Pod
    ↓
MongoDB Container
```

The WebApp does not need to connect directly to the MongoDB Pod IP. Instead, it uses the MongoDB Service name as the stable network endpoint.

### Service and Pod Selection

Services use **label selectors** to determine which Pods receive traffic.

For example:

```yaml
selector:
  app: webapp
```

The WebApp Service routes traffic to Pods with the matching `app: webapp` label.

Similarly, the MongoDB Service can use:

```yaml
selector:
  app: mongodb
```

This routes traffic to the MongoDB Pod with the matching label.


---

### 7. ConfigMap

A **ConfigMap** stores non-sensitive configuration data separately from the application definition.

For example:

```yaml
data:
  MONGO_DB: mydatabase
  MONGO_HOST: mongodb-service
```

The application can consume these values as environment variables.

ConfigMaps are appropriate for configuration that does not contain sensitive information.

---

### 8. Secret

A **Secret** is used to store sensitive configuration data such as usernames, passwords, tokens, or keys.

For example:

```yaml
data:
  MONGO_USERNAME: <base64-value>
  MONGO_PASSWORD: <base64-value>
```

Kubernetes Secret values in the `data` field are Base64-encoded.

Base64 encoding is **not encryption**. It only represents the value in an encoded format.

Secrets can be injected into containers as environment variables or mounted as files.

---

### 9. ConfigMap vs Secret

|               | ConfigMap                   | Secret                   |
| ------------- | --------------------------- | ------------------------ |
| Purpose       | Non-sensitive configuration | Sensitive configuration  |
| Example       | Database name, host         | Username, password       |
| `data` values | Plain text                  | Base64-encoded           |
| Encryption    | Not intended for secrets    | Base64 is not encryption |

In this project, the MongoDB configuration is separated between a ConfigMap and a Secret.

---

### 10. Resource Relationships

The main relationships in this project can be summarized as:

```text
                 Deployment
                     │
                     ▼
                ReplicaSet
                     │
                     ▼
                  Pod
                     │
                     ▼
                Container
                     │
                     ▼
              Container Image


WebApp Pod ──────► MongoDB Service ──────► MongoDB Pod
    │                                         │
    │                                         │
    └────── ConfigMap                         └────── Secret
```

The Deployment manages the WebApp Pods, while Services provide stable network access to the Pods. ConfigMaps and Secrets provide external configuration to the containers.

---
 
### 11. What happens when the Pod goes down?
```text
Pod fails
   ↓
ReplicaSet detects fewer than 1 replica
   ↓
New Pod is created
   ↓
New Pod gets label app=webapp
   ↓
Service selector finds the new Pod
   ↓
Service endpoint is updated
   ↓
Traffic goes to the new Pod
```