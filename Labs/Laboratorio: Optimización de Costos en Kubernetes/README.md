### Laboratorio: Optimización de Costos en Kubernetes

#### Objetivo:
Implementar estrategias para optimizar los costos de ejecución de aplicaciones en Kubernetes, como el uso de nodos spot instances, autoscaling de nodos, y análisis de uso de recursos.

#### Requisitos previos:
1. Cuenta de Google Cloud Platform (GCP).
2. Google Cloud SDK instalado y configurado.
3. kubectl instalado y configurado.
4. Clúster de Kubernetes en GKE.

### Paso 1: Configurar el entorno de GCP

1. **Crear un Proyecto en GCP:**
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

### Paso 2: Crear un clúster de GKE con nodos Spot Instances

1. **Crear un clúster de GKE con Nodos Spot Instances:**
   - Crea un clúster de GKE con nodos spot instances para reducir costos:
     ```sh
     gcloud container clusters create my-cluster --num-nodes=3 --zone us-central1-a --spot
     ```
   - Configura `kubectl` para usar el clúster recién creado:
     ```sh
     gcloud container clusters get-credentials my-cluster --zone us-central1-a
     ```

### Paso 3: Configurar autoscaling de nodos

1. **Habilitar Autoscaling de Nodos:**
   - Habilita el autoscaling de nodos en tu clúster de GKE:
     ```sh
     gcloud container clusters update my-cluster --enable-autoscaling --min-nodes=1 --max-nodes=10 --zone us-central1-a
     ```

### Paso 4: Desplegar una aplicación de ejemplo

1. **Crear un Directorio para la Aplicación:**
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
     from flask import Flask, jsonify
     app = Flask(__name__)

     @app.route('/')
     def hello():
         return jsonify({'message': 'Hello, World!'})

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

5. **Crear un archivo `deployment.yaml`:**
   - Crea un archivo `deployment.yaml` para desplegar la aplicación:
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
             resources:
               requests:
                 cpu: "250m"
                 memory: "64Mi"
               limits:
                 cpu: "500m"
                 memory: "128Mi"
     ```

6. **Aplicar el archivo de despliegue:**
   - Aplica el archivo de despliegue:
     ```sh
     kubectl apply -f deployment.yaml
     ```

### Paso 5: Configurar el servicio y el balanceo de carga

1. **Crear un archivo `service.yaml`:**
   - Crea un archivo `service.yaml` para exponer la aplicación:
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

### Paso 6: Implementar Horizontal Pod Autoscaler (HPA)

1. **Crear un archivo `hpa.yaml`:**
   - Crea un archivo `hpa.yaml` para configurar el Horizontal Pod Autoscaler:
     ```yaml
     apiVersion: autoscaling/v1
     kind: HorizontalPodAutoscaler
     metadata:
       name: my-app-hpa
     spec:
       scaleTargetRef:
         apiVersion: apps/v1
         kind: Deployment
         name: my-app
       minReplicas: 3
       maxReplicas: 10
       targetCPUUtilizationPercentage: 50
     ```

2. **Aplicar el archivo de HPA:**
   - Aplica el archivo de HPA:
     ```sh
     kubectl apply -f hpa.yaml
     ```

### Paso 7: Análisis de uso de recursos

1. **Instalar Metrics Server:**
   - Instala Metrics Server para obtener métricas de uso de recursos:
     ```sh
     kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
     ```

2. **Verificar el uso de recursos:**
   - Usa `kubectl top` para verificar el uso de recursos de los pods y nodos:
     ```sh
     kubectl top pods
     kubectl top nodes
     ```

### Paso 8: Limpieza

1. **Eliminar los Recursos:**
   - Elimina los despliegues, servicios y HPA:
     ```sh
     kubectl delete -f hpa.yaml
     kubectl delete -f service.yaml
     kubectl delete -f deployment.yaml
     ```
   - Elimina el clúster de GKE:
     ```sh
     gcloud container clusters delete my-cluster --zone us-central1-a
     ```

### Resumen

Con este laboratorio, has implementado estrategias para optimizar los costos de ejecución de aplicaciones en Kubernetes, como el uso de nodos spot instances, autoscaling de nodos, y análisis de uso de recursos. Esto demuestra habilidades en la gestión de costos, optimización de recursos y análisis de uso.
