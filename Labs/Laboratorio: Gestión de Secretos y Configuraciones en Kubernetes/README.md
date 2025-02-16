### Laboratorio: Gestión de Secretos y Configuraciones en Kubernetes

#### Objetivo:
Utilizar Kubernetes Secrets y ConfigMaps para gestionar de manera segura las configuraciones y secretos de tu aplicación.

#### Requisitos previos:
1. Cuenta de Google Cloud Platform (GCP).
2. Google Cloud SDK instalado y configurado.
3. Docker instalado en tu máquina local.
4. kubectl instalado y configurado.

### Paso 1: Configurar el entorno de GCP

1. **Crear un proyecto en GCP:**
   - Accede a la consola de GCP y crea un nuevo proyecto.
   - Anota el ID del proyecto.

2. **Habilitar las APIs necesarias:**
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

### Paso 2: Crear un clúster de GKE

1. **Crear un Clúster de GKE:**
   - Crea un clúster de GKE con 3 nodos:
     ```sh
     gcloud container clusters create my-cluster --num-nodes=3 --zone us-central1-a
     ```
   - Configura `kubectl` para usar el clúster recién creado:
     ```sh
     gcloud container clusters get-credentials my-cluster --zone us-central1-a
     ```

### Paso 3: Crear una aplicación de ejemplo

1. **Crear un directorio para la aplicación:**
   - Crea un directorio para tu aplicación:
     ```sh
     mkdir my-app
     cd my-app
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
     from flask import Flask, jsonify, request
     import os

     app = Flask(__name__)

     @app.route('/')
     def hello():
         return jsonify({'message': 'Hello, World!'})

     @app.route('/config')
     def config():
         db_user = os.getenv('DB_USER')
         db_password = os.getenv('DB_PASSWORD')
         return jsonify({'db_user': db_user, 'db_password': db_password})

     if __name__ == '__main__':
         app.run(host='0.0.0.0', port=5000)
     ```

4. **Construir y subir la imagen a Google Container Registry:**
   - Construye la imagen:
     ```sh
     docker build -t gcr.io/[PROJECT_ID]/my-app:v1 .
     ```
   - Sube la imagen:
     ```sh
     docker push gcr.io/[PROJECT_ID]/my-app:v1
     ```

### Paso 4: Crear un ConfigMap

1. **Crear un Archivo `configmap.yaml`:**
   - Crea un archivo `configmap.yaml` para almacenar configuraciones no sensibles:
     ```yaml
     apiVersion: v1
     kind: ConfigMap
     metadata:
       name: my-app-config
     data:
       DB_HOST: "localhost"
       DB_PORT: "5432"
     ```

2. **Aplicar el archivo de ConfigMap:**
   - Aplica el archivo de ConfigMap:
     ```sh
     kubectl apply -f configmap.yaml
     ```

### Paso 5: Crear un Secret

1. **Crear un Archivo `secret.yaml`:**
   - Crea un archivo `secret.yaml` para almacenar secretos sensibles. Asegúrate de codificar los valores en base64:
     ```yaml
     apiVersion: v1
     kind: Secret
     metadata:
       name: my-app-secret
     type: Opaque
     data:
       DB_USER: YWRtaW4=  # 'admin' en base64
       DB_PASSWORD: cGFzc3dvcmQ=  # 'password' en base64
     ```

2. **Aplicar el archivo de Secret:**
   - Aplica el archivo de Secret:
     ```sh
     kubectl apply -f secret.yaml
     ```

### Paso 6: Desplegar la aplicación en GKE

1. **Crear un archivo `deployment.yaml`:**
   - Crea un archivo `deployment.yaml` que utilice el ConfigMap y el Secret:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: my-app
     spec:
       replicas: 3
       selector:
         matchLabels:
           app: my-app
       template:
         metadata:
           labels:
             app: my-app
         spec:
           containers:
           - name: my-app
             image: gcr.io/[PROJECT_ID]/my-app:v1
             ports:
             - containerPort: 5000
             envFrom:
             - configMapRef:
                 name: my-app-config
             env:
             - name: DB_USER
               valueFrom:
                 secretKeyRef:
                   name: my-app-secret
                   key: DB_USER
             - name: DB_PASSWORD
               valueFrom:
                 secretKeyRef:
                   name: my-app-secret
                   key: DB_PASSWORD
     ```

2. **Aplicar el archivo de despliegue:**
   - Aplica el archivo de despliegue:
     ```sh
     kubectl apply -f deployment.yaml
     ```

### Paso 7: Configurar el servicio y el balanceo de Carga

1. **Crear un archivo `service.yaml`:**
   - Crea un archivo `service.yaml`:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: my-app
     spec:
       selector:
         app: my-app
       ports:
         - protocol: TCP
           port: 80
           targetPort: 5000
       type: LoadBalancer
     ```

2. **Aplicar el archivo de servicio:**
   - Aplica el archivo de servicio:
     ```sh
     kubectl apply -f service.yaml
     ```

3. **Obtener la dirección IP del Load Balancer:**
   - Verifica el estado del servicio y obtén la dirección IP externa:
     ```sh
     kubectl get services
     ```
   - Una vez que la columna `EXTERNAL-IP` tenga una dirección IP, accede a ella en tu navegador para ver la aplicación en funcionamiento.

### Paso 8: Verificar la configuración y los secretos

1. **Acceder a la aplicación:**
   - Abre un navegador y navega a la dirección IP externa del servicio.
   - Accede a la ruta `/config` para verificar que la aplicación puede leer las configuraciones y secretos.

### Paso 9: Limpieza

1. **Eliminar los recursos:**
   - Elimina los despliegues, servicios, ConfigMap y Secret:
     ```sh
     kubectl delete -f service.yaml
     kubectl delete -f deployment.yaml
     kubectl delete -f configmap.yaml
     kubectl delete -f secret.yaml
     ```
   - Elimina el clúster de GKE:
     ```sh
     gcloud container clusters delete my-cluster --zone us-central1-a
     ```

### Resumen

Con este laboratorio, has utilizado Kubernetes Secrets y ConfigMaps para gestionar de manera segura las configuraciones y secretos de tu aplicación. Esto demuestra habilidades en la gestión de configuraciones y el manejo de secretos sensibles en Kubernetes.
