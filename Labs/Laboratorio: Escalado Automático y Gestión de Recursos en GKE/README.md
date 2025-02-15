### Laboratorio: Escalado automático y gestión de recursos en GKE

#### Objetivo:
Implementar políticas de escalado automático y gestión de recursos para asegurar que tu aplicación pueda manejar cargas variables de tráfico en un clúster de Kubernetes en GKE.

#### Requisitos previos:
1. Cuenta de Google Cloud Platform (GCP).
2. Google Cloud SDK instalado y configurado.
3. Docker instalado en tu máquina local.
4. kubectl instalado y configurado.

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
     from flask import Flask, jsonify
     import os
     import time

     app = Flask(__name__)

     @app.route('/')
     def hello():
         return jsonify({'message': 'Hello, World!'})

     @app.route('/load')
     def load():
         time.sleep(int(os.getenv('LOAD_TIME', 5)))
         return jsonify({'message': 'Load test!'})

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

### Paso 4: Desplegar la aplicación en GKE

1. **Crear un archivo `deployment.yaml`:**
   - Crea un archivo `deployment.yaml`:
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

2. **Aplicar el archivo de despliegue:**
   - Aplica el archivo de despliegue:
     ```sh
     kubectl apply -f deployment.yaml
     ```

### Paso 5: Configurar el servicio y el balanceo de carga

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

### Paso 6: Implementar el Horizontal Pod Autoscaler

Para ver que versiones de Horizontal Pod Autoscaler están disponibles en el clúster, ejecuta:
```sh
kubectl api-versions | grep autoscaling

```
Saber la versión permite modificarle en la *apiVersion* del hpa.yaml

1. **Crear un archivo `hpa.yaml`:**
   - Crea un archivo `hpa.yaml`:
     ```yaml
      apiVersion: autoscaling/v2
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
        metrics:
        - type: Resource
          resource:
            name: cpu
            target:
              type: Utilization
              averageUtilization: 50

     ```

2. **Aplicar el archivo de HPA:**
   - Aplica el archivo de HPA:
     ```sh
     kubectl apply -f hpa.yaml
     ```

### Paso 7: Verificar el escalado automático

Para instalar en Linux: https://grafana.com/docs/k6/latest/set-up/install-k6/

1. **Generar carga en la aplicación:**
   - Puedes usar herramientas como `k6` o `Apache Benchmark` para generar carga en tu aplicación. Aquí hay un ejemplo usando `k6`, primero hay que crear el script:
   ```sh
        import http from 'k6/http';
         import { sleep } from 'k6';
         
         export let options = {
           vus: 50, // Número de usuarios virtuales
           duration: '2m', // Duración de la prueba
         };
         
         export default function () {
           // Enviar solicitudes GET para simular carga en el servidor
           http.get('http://[IP-EXTERNA]/load');
         
           // Simulación de carga de CPU más costosa (cálculos de factoriales)
           for (let i = 1; i <= 1000; i++) {
             factorial(i); // Realizar cálculo de factorial de números grandes
           }
         
           sleep(0.5); // Reducimos el tiempo de descanso para mantener la carga alta
         }
         
         // Función para calcular el factorial de un número de forma recursiva
         function factorial(n) {
           if (n <= 1) {
             return 1;
           } else {
             return n * factorial(n - 1);
           }
         }

     ```
     En este script se hacen operaciones costosas para observar el comportamiento del HPA, para ejecutar el script:
     ```sh
     k6 run script.js
     ```

1. **Monitorear el escalado:**
   - Observa cómo el número de pods aumenta y disminuye en respuesta a la carga:
     
     Para observar los pods:
     ```sh
        kubectl get pods -w
     ```
     Para observar el HPA:
     ```sh
      kubectl get hpa -w
      ```
### Paso 8: Limpieza

1. **Eliminar los recursos:**
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

Con este laboratorio, has implementado políticas de escalado automático y gestión de recursos en tu clúster de GKE. Esto asegura que tu aplicación pueda manejar cargas variables de tráfico de manera eficiente, optimizando el uso de recursos y la capacidad del clúster.
