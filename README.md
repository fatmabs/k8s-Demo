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

