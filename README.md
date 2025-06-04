# MLOps-k8s-puj

# Despliegue de API

Este proyecto implementa un flujo CI/CD automatizado para una API de inferencia en FastAPI. El despliegue se realiza en un clúster Kubernetes distribuido, donde la **máquina de Jeison** actúa como nodo con capacidad de construcción de imágenes y despliegue.

## 🔧 Estructura del Proyecto

```
Taller_CI_CD/
├── Niveles/
│   └── 4/
│       ├── api/
│       │   ├── app/
│       │   │   ├── main.py
│       │   │   └── model.pkl
│       │   ├── train_model.py
│       │   ├── requirements.txt
│       │   └── Dockerfile
│       └── manifests/
│           ├── api-deployment.yaml
│           └── api-service.yaml
├── .github/
│   └── workflows/
│       └── ci-cd.yml
├── .dockerignore
└── README.md
```

## 🧪 Entrenamiento del modelo

El modelo se entrena automáticamente desde el script `train_model.py`, que genera un archivo `model.pkl` en la carpeta `app/`.

## 🐳 Docker

El `Dockerfile` está optimizado para reducir tiempos de build utilizando cache de dependencias:

```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "80"]
```

## 📦 Archivo `.dockerignore`

Asegura que no se copien archivos innecesarios al contenedor:

```
__pycache__/
*.pyc
.git
.github
```

## 🚀 CI/CD Workflow (`.github/workflows/ci-cd.yml`)

### Proceso automatizado al hacer push en `main`:

1. **Checkout del código**
2. **Instalación de dependencias**
3. **Entrenamiento del modelo**
4. **Construcción y subida de imagen Docker a DockerHub**
5. **Despliegue en Kubernetes con `kubectl`**

### Autenticación

- Se utiliza `DOCKER_USERNAME`, `DOCKER_PASSWORD` como secrets para Docker Hub.
- Se configura el acceso a Kubernetes usando el contenido base64 de `~/.kube/config` como `KUBECONFIG`.

### Despliegue en Kubernetes

El archivo `api-deployment.yaml` define el `Deployment` y `Service`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: <DOCKER_USERNAME>/api:latest
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: api
spec:
  selector:
    app: api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

## ⚙️ Configuración del clúster Kubernetes

- El clúster tiene un **nodo maestro (10.43.101.184)** y otros nodos como el de Jeison.
- El despliegue puede redirigir a cualquier nodo según disponibilidad.
- El puerto `NodePort` expone el servicio para acceso externo.


## ✅ Acceso a la API

Una vez desplegado correctamente, accede al endpoint Swagger:

```
http://<IP_DEL_NODE>:<PORT>/docs
```

Por ejemplo:

```
http://10.43.101.184:30080/docs
```
