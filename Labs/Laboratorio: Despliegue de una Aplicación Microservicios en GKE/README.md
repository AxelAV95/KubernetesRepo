
### Laboratorio: Despliegue de una aplicación microservicios en GKE

#### Objetivo:
Desplegar una aplicación basada en microservicios en un clúster de Kubernetes en GKE.

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
   - Verificar que regiones se permite usar en un proyecto
    ```sh
      gcloud org-policies describe constraints/gcp.resourceLocations --project  [PROJECT_ID]
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

### Paso 3: Crear microservicios

1. **Crear un microservicio de Backend:**
   - Crea un directorio para tu microservicio de backend:
     ```sh
     mkdir backend
     cd backend
     ```
   - Crea un archivo `Dockerfile`:
     ```Dockerfile
     FROM python:3.8-slim
     WORKDIR /app
     COPY . /app
     RUN pip install flask
     CMD ["python", "app.py"]
     ```
   - Crea un archivo `app.py`:
     ```python
     from flask import Flask, jsonify
     app = Flask(__name__)
     @app.route('/api', methods=['GET'])
     def api():
         return jsonify({'message': 'Hello from backend!'})
     if __name__ == '__main__':
         app.run(host='0.0.0.0', port=5000)
     ```
   - Construye y sube la imagen a Google Container Registry:
     ```sh
     docker build -t gcr.io/[PROJECT_ID]/backend:v1 .
     docker push gcr.io/[PROJECT_ID]/backend:v1
     ```

2. **Crear un microservicio de Frontend:**
   - Crea un directorio para tu microservicio de frontend:
     ```sh
     cd ..
     mkdir frontend
     cd frontend
     ```
   - Crea un archivo `Dockerfile`:
     ```Dockerfile
     FROM nginx:alpine
     COPY . /usr/share/nginx/html
     COPY nginx.conf /etc/nginx/nginx.conf
     ```
   - Crea un archivo `index.html`:
     ```html
     <!DOCTYPE html>
     <html>
     <head>
         <title>Frontend</title>
     </head>
     <body>
         <h1>Hello from frontend!</h1>
         <button onclick="fetchData()">Fetch Data from Backend</button>
         <p id="data"></p>
         <script>
             function fetchData() {
                 fetch('/api')
                     .then(response => response.json())
                     .then(data => document.getElementById('data').innerText = data.message);
             }
         </script>
     </body>
     </html>
     ```
   - Crea un archivo `nginx.conf`:
     ```nginx
     events {
          worker_connections 1024;
     }

     http {
       server {
           listen 80;

           location / {
               root /usr/share/nginx/html;
               index index.html index.htm;
               try_files $uri $uri/ /index.html;
           }

           location /api {
               proxy_pass http://backend;
               proxy_set_header Host $host;
               proxy_set_header X-Real-IP $remote_addr;
               proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_set_header X-Forwarded-Proto $scheme;
           }
       }
      }
     ```
   - Construye y sube la imagen a Google Container Registry:
     ```sh
     docker build -t gcr.io/[PROJECT_ID]/frontend:v1 .
     docker push gcr.io/[PROJECT_ID]/frontend:v1
     ```

### Paso 4: Desplegar microservicios en GKE

1. **Desplegar el microservicio de Backend:**
   - Crea un archivo `backend-deployment.yaml`:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: backend
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
   - Aplica el archivo de despliegue:
     ```sh
     kubectl apply -f backend-deployment.yaml
     ```

2. **Desplegar el microservicio de Frontend:**
   - Crea un archivo `frontend-deployment.yaml`:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: frontend
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
   - Aplica el archivo de despliegue:
     ```sh
     kubectl apply -f frontend-deployment.yaml
     ```

### Paso 5: Configurar el servicio y el balanceo de Carga

1. **Crear un Servicio para el Backend:**
   - Crea un archivo `backend-service.yaml`:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: backend
     spec:
       selector:
         app: backend
       ports:
         - protocol: TCP
           port: 80
           targetPort: 5000
       type: ClusterIP
     ```
   - Aplica el archivo de servicio:
     ```sh
     kubectl apply -f backend-service.yaml
     ```

2. **Crear un servicio para el Frontend:**
   - Crea un archivo `frontend-service.yaml`:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: frontend
     spec:
       selector:
         app: frontend
       ports:
         - protocol: TCP
           port: 80
           targetPort: 80
       type: LoadBalancer
     ```
   - Aplica el archivo de servicio:
     ```sh
     kubectl apply -f frontend-service.yaml
     ```

3. **Obtener la dirección IP del Load Balancer:**
   - Verifica el estado del servicio de frontend y obtén la dirección IP externa:
     ```sh
     kubectl get services
     ```
   - Una vez que la columna `EXTERNAL-IP` tenga una dirección IP, accede a ella en tu navegador para ver la aplicación en funcionamiento.

### Paso 6: Verificar el despliegue

1. **Acceder a la aplicación:**
   - Abre un navegador y navega a la dirección IP externa del servicio de frontend.
   - Haz clic en el botón "Fetch Data from Backend" para verificar que el frontend puede comunicarse con el backend.

### Paso 7: Limpieza

**Para verificar Logs de un pod, usar:**
  ```sh
    kubectl logs [POD-ID] --previous
   ```

1. **Eliminar los recursos:**
   - Elimina los despliegues y servicios:
     ```sh
     kubectl delete -f frontend-service.yaml
     kubectl delete -f backend-service.yaml
     kubectl delete -f frontend-deployment.yaml
     kubectl delete -f backend-deployment.yaml
     ```
   - Elimina el clúster de GKE:
     ```sh
     gcloud container clusters delete my-cluster --zone us-central1-a
     ```
# Extras

### Habilitar Horizontal Pod Autoscaler (HPA)
Antes de hacer una prueba de carga, asegúrate de que tu aplicación está configurada para escalar automáticamente. Kubernetes tiene un recurso llamado Horizontal Pod Autoscaler (HPA) que ajusta el número de réplicas de un pod en función de métricas como la CPU y la memoria.

Pasos para configurar el HPA:
- Instalar el HPA: Si ya tienes un deployment de tu aplicación, puedes habilitar el autoscaling por CPU con el siguiente comando:
```sh
   kubectl autoscale deployment frontend --cpu-percent=50 --min=1 --max=10
```
Esto significa que Kubernetes ajustará el número de réplicas entre 1 y 10 dependiendo de la carga de CPU.

- Verificar el HPA: Puedes verificar si el HPA está funcionando correctamente con:

     ```sh
     kubectl get hpa
     ```
- Monitorear el escalado: A medida que generas carga en tu aplicación, Kubernetes debería aumentar o disminuir el número de réplicas según la carga.

### Pruebas de carga con Artillery

Artillery es una herramienta ligera y fácil de usar para realizar pruebas de carga y de rendimiento en aplicaciones web. Puedes instalarla y ejecutarla con un archivo de configuración sencillo:

  ```sh
npm install -g artillery
artillery quick --count 100 -n 10 http://your-service-url
  ```
Esto enviará 100 solicitudes concurrentes a tu URL durante 10 segundos.

### Monitorear recursos en GKE
Es importante monitorear el uso de recursos como CPU, memoria y red mientras realizas las pruebas de carga.

- Google Cloud Monitoring: GKE se integra con Google Cloud Monitoring (antes conocido como Stackdriver) para ofrecer métricas de rendimiento.

- Kubectl Metrics: Usa kubectl top para obtener métricas de uso de recursos de tus pods:

```bash
kubectl top pods
```
- Prometheus + Grafana: Si necesitas un monitoreo más avanzado, puedes configurar Prometheus y Grafana en tu clúster de Kubernetes. Estas herramientas te proporcionan métricas detalladas sobre el estado de tu aplicación y clúster.

### Escalabilidad de red y latencia
Además de la CPU y la memoria, es importante monitorear la latencia y el tráfico de red mientras se prueba la carga. Si tu servicio depende de otros microservicios (por ejemplo, backend), monitorear el tráfico y la latencia entre servicios es crucial.

- Istio: Si estás utilizando Istio para la gestión del tráfico, puedes configurarlo para realizar pruebas de latencia y resiliencia.
- Pingdom o New Relic: Puedes usar servicios externos como Pingdom o New Relic para monitorear la latencia de tus servicios y obtener información sobre la disponibilidad.

### Realizar pruebas de fallos (Chaos Engineering)
Para evaluar la resiliencia de tu sistema bajo carga, puedes realizar pruebas de caos para simular fallos en la infraestructura.

- Chaos Mesh: Es una herramienta que te permite introducir fallos de red, fallos de nodos, etc., en tu clúster de Kubernetes para probar cómo responde tu aplicación.
- Gremlin: Otra herramienta para realizar pruebas de caos, que permite inyectar fallos y ver cómo tu sistema maneja situaciones inesperadas.
  
### Prueba de escalabilidad manual
Si prefieres realizar una prueba más sencilla sin herramientas complejas, puedes simplemente escalar manualmente los pods y observar cómo la aplicación maneja el aumento del tráfico. Para hacer esto, puedes usar los siguientes comandos:

Escalar manualmente los pods:

```sh
kubectl scale deployment frontend --replicas=10
```
Para simular más tráfico, aumenta el número de réplicas y verifica si Kubernetes maneja correctamente el aumento de carga.

Luego, revisa el comportamiento de la aplicación y si los pods están siendo gestionados correctamente.



