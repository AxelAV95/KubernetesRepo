### Laboratorio: Uso de Namespaces para Testing, Desarrollo y Producción en Kubernetes

#### Objetivo:
Practicar el uso de namespaces en Kubernetes para separar entornos de testing, desarrollo y producción, y desplegar distintas aplicaciones en cada uno de estos namespaces.

#### Requisitos previos:
1. Cuenta de Google Cloud Platform (GCP).
2. Google Cloud SDK instalado y configurado.
3. kubectl instalado y configurado.
4. Clúster de Kubernetes en GKE.

### Paso 1: Configurar el entorno de GCP

1. **Crear un proyecto en GCP:**
   - Accede a la consola de GCP y crea un nuevo proyecto.
   - Anota el ID del proyecto.

2. **Habilitar las APIs Necesarias:**
   - Habilita las siguientes APIs en tu proyecto:
     - Kubernetes Engine API
     - Compute Engine API
     - Container Registry API

3. **Configurar Google Cloud SDK:**
   - Abre una terminal y autentícate con tu cuenta de GCP:
     ```sh
     gcloud auth login
     ```
   - Establece el proyecto predeterminado:
     ```sh
     gcloud config set project [PROJECT_ID]
     ```

### Paso 2: Crear un Clúster de GKE

1. **Crear un clúster de GKE:**
   - Crea un clúster de GKE con 3 nodos:
     ```sh
     gcloud container clusters create my-cluster --num-nodes=3 --zone us-central1-a
     ```
   - Configura `kubectl` para usar el clúster recién creado:
     ```sh
     gcloud container clusters get-credentials my-cluster --zone us-central1-a
     ```

### Paso 3: Crear Namespaces para Testing, Desarrollo y Producción

1. **Crear Namespaces:**
   - Crea archivos YAML para definir los namespaces:
     ```yaml
     # namespace-testing.yaml
     apiVersion: v1
     kind: Namespace
     metadata:
       name: testing
     ```

     ```yaml
     # namespace-development.yaml
     apiVersion: v1
     kind: Namespace
     metadata:
       name: development
     ```

     ```yaml
     # namespace-production.yaml
     apiVersion: v1
     kind: Namespace
     metadata:
       name: production
     ```

2. **Aplicar los archivos de Namespaces:**
   - Aplica los archivos de namespaces:
     ```sh
     kubectl apply -f namespace-testing.yaml
     kubectl apply -f namespace-development.yaml
     kubectl apply -f namespace-production.yaml
     ```

### Paso 4: Desplegar aplicaciones en los Namespaces

#### Aplicación 1: Frontend

1. **Crear un directorio para la aplicación frontend:**
   - Crea un directorio para la aplicación frontend:
     ```sh
     mkdir frontend
     cd frontend
     ```

2. **Crear un archivo `Dockerfile`:**
   - Crea un archivo `Dockerfile`:
     ```Dockerfile
     FROM nginx:alpine
     COPY . /usr/share/nginx/html
     ```

3. **Crear un archivo `index.html`:**
   - Crea un archivo `index.html`:
     ```html
     <!DOCTYPE html>
     <html>
     <head>
         <title>Frontend</title>
     </head>
     <body>
         <h1>Hello from Frontend!</h1>
     </body>
     </html>
     ```

4. **Construir y subir la imagen a Google Container Registry:**
   - Construye la imagen:
     ```sh
     docker build -t gcr.io/[PROJECT_ID]/frontend:v1 .
     ```
   - Sube la imagen:
     ```sh
     docker push gcr.io/[PROJECT_ID]/frontend:v1
     ```

5. **Crear archivos de despliegue para cada Namespace:**
   - Crea un archivo `frontend-deployment-testing.yaml`:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: frontend
       namespace: testing
     spec:
       replicas: 1
       selector:
         matchLabels:
           app: frontend
       template:
         metadata:
           labels:
             app: frontend
         spec:
           containers:
           - name: frontend
             image: gcr.io/[PROJECT_ID]/frontend:v1
             ports:
             - containerPort: 80
     ```

   - Crea un archivo `frontend-deployment-development.yaml`:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: frontend
       namespace: development
     spec:
       replicas: 2
       selector:
         matchLabels:
           app: frontend
       template:
         metadata:
           labels:
             app: frontend
         spec:
           containers:
           - name: frontend
             image: gcr.io/[PROJECT_ID]/frontend:v1
             ports:
             - containerPort: 80
     ```

   - Crea un archivo `frontend-deployment-production.yaml`:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: frontend
       namespace: production
     spec:
       replicas: 3
       selector:
         matchLabels:
           app: frontend
       template:
         metadata:
           labels:
             app: frontend
         spec:
           containers:
           - name: frontend
             image: gcr.io/[PROJECT_ID]/frontend:v1
             ports:
             - containerPort: 80
     ```

6. **Aplicar los archivos de despliegue:**
   - Aplica los archivos de despliegue:
     ```sh
     kubectl apply -f frontend-deployment-testing.yaml
     kubectl apply -f frontend-deployment-development.yaml
     kubectl apply -f frontend-deployment-production.yaml
     ```

7. **Crear archivos de servicio para cada Namespace:**
   - Crea un archivo `frontend-service-testing.yaml`:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: frontend
       namespace: testing
     spec:
       selector:
         app: frontend
       ports:
         - protocol: TCP
           port: 80
           targetPort: 80
       type: ClusterIP
     ```

   - Crea un archivo `frontend-service-development.yaml`:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: frontend
       namespace: development
     spec:
       selector:
         app: frontend
       ports:
         - protocol: TCP
           port: 80
           targetPort: 80
       type: ClusterIP
     ```

   - Crea un archivo `frontend-service-production.yaml`:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: frontend
       namespace: production
     spec:
       selector:
         app: frontend
       ports:
         - protocol: TCP
           port: 80
           targetPort: 80
       type: LoadBalancer
     ```

8. **Aplicar los archivos de servicio:**
   - Aplica los archivos de servicio:
     ```sh
     kubectl apply -f frontend-service-testing.yaml
     kubectl apply -f frontend-service-development.yaml
     kubectl apply -f frontend-service-production.yaml
     ```

#### Aplicación 2: Backend

1. **Crear un directorio para la aplicación Backend:**
   - Crea un directorio para la aplicación backend:
     ```sh
     mkdir backend
     cd backend
     ```

2. **Crear un archivo `Dockerfile`:**
   - Crea un archivo `Dockerfile`:
     ```Dockerfile
     FROM python:3.8-slim
     WORKDIR /app
     COPY . /app
     RUN pip install flask
     CMD ["python", "app.py"]
     ```

3. **Crear un archivo `app.py`:**
   - Crea un archivo `app.py`:
     ```python
     from flask import Flask, jsonify

     app = Flask(__name__)

     @app.route('/')
     def hello():
         return jsonify({'message': 'Hello from Backend!'})

     if __name__ == '__main__':
         app.run(host='0.0.0.0', port=5000)
     ```

4. **Construir y subir la imagen a Google Container Registry:**
   - Construye la imagen:
     ```sh
     docker build -t gcr.io/[PROJECT_ID]/backend:v1 .
     ```
   - Sube la imagen:
     ```sh
     docker push gcr.io/[PROJECT_ID]/backend:v1
     ```

5. **Crear archivos de despliegue para cada Namespace:**
   - Crea un archivo `backend-deployment-testing.yaml`:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: backend
       namespace: testing
     spec:
       replicas: 1
       selector:
         matchLabels:
           app: backend
       template:
         metadata:
           labels:
             app: backend
         spec:
           containers:
           - name: backend
             image: gcr.io/[PROJECT_ID]/backend:v1
             ports:
             - containerPort: 5000
     ```

   - Crea un archivo `backend-deployment-development.yaml`:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: backend
       namespace: development
     spec:
       replicas: 2
       selector:
         matchLabels:
           app: backend
       template:
         metadata:
           labels:
             app: backend
         spec:
           containers:
           - name: backend
             image: gcr.io/[PROJECT_ID]/backend:v1
             ports:
             - containerPort: 5000
     ```

   - Crea un archivo `backend-deployment-production.yaml`:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: backend
       namespace: production
     spec:
       replicas: 3
       selector:
         matchLabels:
           app: backend
       template:
         metadata:
           labels:
             app: backend
         spec:
           containers:
           - name: backend
             image: gcr.io/[PROJECT_ID]/backend:v1
             ports:
             - containerPort: 5000
     ```

6. **Aplicar los archivos de despliegue:**
   - Aplica los archivos de despliegue:
     ```sh
     kubectl apply -f backend-deployment-testing.yaml
     kubectl apply -f backend-deployment-development.yaml
     kubectl apply -f backend-deployment-production.yaml
     ```

7. **Crear archivos de servicio para cada Namespace:**
   - Crea un archivo `backend-service-testing.yaml`:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: backend
       namespace: testing
     spec:
       selector:
         app: backend
       ports:
         - protocol: TCP
           port: 80
           targetPort: 5000
       type: ClusterIP
     ```

   - Crea un archivo `backend-service-development.yaml`:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: backend
       namespace: development
     spec:
       selector:
         app: backend
       ports:
         - protocol: TCP
           port: 80
           targetPort: 5000
       type: ClusterIP
     ```

   - Crea un archivo `backend-service-production.yaml`:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: backend
       namespace: production
     spec:
       selector:
         app: backend
       ports:
         - protocol: TCP
           port: 80
           targetPort: 5000
       type: LoadBalancer
     ```

8. **Aplicar los archivos de servicio:**
   - Aplica los archivos de servicio:
     ```sh
     kubectl apply -f backend-service-testing.yaml
     kubectl apply -f backend-service-development.yaml
     kubectl apply -f backend-service-production.yaml
     ```

### Paso 5: Verificar los despliegues

1. **Acceder a las aplicaciones:**
    - Para ver los servicios y pods de un namespaces específico usar:
     ```sh
         kubectl get services -n NAMESPACE
         kubectl get pods -n NAMESPACE
     ```
   - Para acceder a un pod específico de manera local para propósitos de desarrollo y testing (necesario tener SDK instalado en la máquina), usar:
     ```sh
        kubectl port-forward -n NAMESPACE pod/NOMBRE_DEL_POD 8080:8080
     ```
   - Abre un navegador y navega a las direcciones IP externas de los servicios de  `production`.
   - Verifica que las aplicaciones estén funcionando correctamente en cada entorno.

### Paso 6: Limpieza

1. **Eliminar los recursos:**
   - Elimina los despliegues y servicios:
     ```sh
     kubectl delete -f backend-service-testing.yaml
     kubectl delete -f backend-deployment-testing.yaml
     kubectl delete -f backend-service-development.yaml
     kubectl delete -f backend-deployment-development.yaml
     kubectl delete -f backend-service-production.yaml
     kubectl delete -f backend-deployment-production.yaml
     kubectl delete -f frontend-service-testing.yaml
     kubectl delete -f frontend-deployment-testing.yaml
     kubectl delete -f frontend-service-development.yaml
     kubectl delete -f frontend-deployment-development.yaml
     kubectl delete -f frontend-service-production.yaml
     kubectl delete -f frontend-deployment-production.yaml
     ```
   - Elimina los namespaces:
     ```sh
     kubectl delete namespace testing
     kubectl delete namespace development
     kubectl delete namespace production
     ```
   - Elimina el clúster de GKE:
     ```sh
     gcloud container clusters delete my-cluster --zone us-central1-a
     ```

### Resumen

Con este laboratorio, has practicado el uso de namespaces en Kubernetes para separar entornos de testing, desarrollo y producción, y has desplegado distintas aplicaciones en cada uno de estos namespaces. Esto demuestra habilidades en la gestión de entornos y la organización de aplicaciones en Kubernetes.
