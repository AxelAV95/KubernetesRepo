### Laboratorio: Despliegue de Aplicaciones con estado en GKE

#### Objetivo:
Desplegar una aplicación que requiere almacenamiento persistente, como una base de datos PostgreSQL, utilizando Persistent Volumes y Persistent Volume Claims en un clúster de Kubernetes en GKE.

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

1. **Crear un clúster de GKE:**

    - Para verificar en que zonas se puede desplegar el clúster en el proyecto, ejecutar el comando:
    ```sh
      gcloud org-policies describe constraints/gcp.resourceLocations --project [PROJECT-ID]
    ```

   - Crea un clúster de GKE con 3 nodos:
     ```sh
     gcloud container clusters create my-cluster --num-nodes=3 --zone [ZONE]
     ```
   - Configura `kubectl` para usar el clúster recién creado:
     ```sh
     gcloud container clusters get-credentials my-cluster --zone [ZONE]
     ```

### Paso 3: Crear una aplicación de ejemplo con PostgreSQL

1. **Crear un cirectorio para la aplicación:**
   - Crea un directorio para tu aplicación:
     ```sh
     mkdir my-app
     cd my-app
     ```

2. **Crear un archivo `Dockerfile`:**
   - Crea un archivo `Dockerfile` para la aplicación:
     ```Dockerfile
     FROM python:3.8-slim
     WORKDIR /app
     COPY . /app
     RUN pip install flask psycopg2-binary
     CMD ["python", "app.py"]
     ```

3. **Crear un archivo `app.py`:**
   - Crea un archivo `app.py` para la aplicación:
     ```python
     from flask import Flask, jsonify, request
     import psycopg2
     import os

     app = Flask(__name__)

     def get_db_connection():
         conn = psycopg2.connect(
             host=os.getenv('DB_HOST'),
             database=os.getenv('DB_NAME'),
             user=os.getenv('DB_USER'),
             password=os.getenv('DB_PASSWORD')
         )
         return conn

     @app.route('/')
     def hello():
         return jsonify({'message': 'Hello, World!'})

     @app.route('/data', methods=['GET', 'POST'])
     def data():
         conn = get_db_connection()
         cur = conn.cursor()
         if request.method == 'POST':
             data = request.json
             cur.execute('INSERT INTO my_table (name, value) VALUES (%s, %s)', (data['name'], data['value']))
             conn.commit()
             return jsonify({'message': 'Data inserted!'})
         else:
             cur.execute('SELECT * FROM my_table')
             rows = cur.fetchall()
             return jsonify(rows)
         cur.close()
         conn.close()

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

### Paso 4: Crear un Persistent Volume y Persistent Volume Claim

   Antes de crear estos recursos, es necesario crear un disco para persistencia, en este caso:
    
   ```sh
       gcloud compute disks create postgres-disk --size=10GB --zone=[ZONE]
   ```

1. **Crear un Archivo `pv.yaml`:**
   - Crea un archivo `pv.yaml` para definir un Persistent Volume:
     ```yaml
     apiVersion: v1
     kind: PersistentVolume
     metadata:
       name: postgres-pv
     spec:
       capacity:
         storage: 10Gi
       accessModes:
         - ReadWriteOnce
       gcePersistentDisk:
         pdName: postgres-disk
         fsType: ext4
     ```

2. **Crear un archivo `pvc.yaml`:**
   - Crea un archivo `pvc.yaml` para definir un Persistent Volume Claim:
     ```yaml
     apiVersion: v1
     kind: PersistentVolumeClaim
     metadata:
       name: postgres-pvc
     spec:
       accessModes:
         - ReadWriteOnce
       resources:
         requests:
           storage: 10Gi
     ```

3. **Aplicar los archivos de PV y PVC:**
   - Aplica los archivos de PV y PVC:
     ```sh
     kubectl apply -f pv.yaml
     kubectl apply -f pvc.yaml
     ```

### Paso 5: Desplegar PostgreSQL en GKE

1. **Crear un archivo `postgres-deployment.yaml`:**
   - Crea un archivo `postgres-deployment.yaml` para desplegar PostgreSQL:
     ```yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: postgres
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: postgres
        template:
          metadata:
            labels:
              app: postgres
          spec:
            containers:
            - name: postgres
              image: postgres:13
              ports:
              - containerPort: 5432
              env:
              - name: POSTGRES_DB
                value: mydatabase
              - name: POSTGRES_USER
                value: myuser
              - name: POSTGRES_PASSWORD
                value: mypassword
              - name: PGDATA
                value: /var/lib/postgresql/data/pgdata   # <--- Variable agregada
              volumeMounts:
              - mountPath: /var/lib/postgresql/data
                name: postgres-storage
            volumes:
            - name: postgres-storage
              persistentVolumeClaim:
                claimName: postgres-pvc
      
       ```

2. **Aplicar el archivo de despliegue de PostgreSQL:**
   - Aplica el archivo de despliegue de PostgreSQL:
     ```sh
     kubectl apply -f postgres-deployment.yaml
     ```

3. **Crear un archivo `postgres-service.yaml`:**
   - Crea un archivo `postgres-service.yaml` para exponer PostgreSQL:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: postgres
     spec:
       selector:
         app: postgres
       ports:
         - protocol: TCP
           port: 5432
           targetPort: 5432
       type: ClusterIP
     ```

4. **Aplicar el archivo de servicio de PostgreSQL:**
   - Aplica el archivo de servicio de PostgreSQL:
     ```sh
     kubectl apply -f postgres-service.yaml
     ```

### Paso 6: Desplegar la aplicación en GKE

1. **Crear un archivo `app-deployment.yaml`:**
   - Crea un archivo `app-deployment.yaml` para desplegar la aplicación:
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
             env:
             - name: DB_HOST
               value: postgres
             - name: DB_NAME
               value: mydatabase
             - name: DB_USER
               value: myuser
             - name: DB_PASSWORD
               value: mypassword
     ```

2. **Aplicar el Archivo de despliegue de la aplicación:**
   - Aplica el archivo de despliegue de la aplicación:
     ```sh
     kubectl apply -f app-deployment.yaml
     ```

### Paso 7: Configurar el servicio y el balanceo de carga

1. **Crear un archivo `app-service.yaml`:**
   - Crea un archivo `app-service.yaml` para exponer la aplicación:
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

2. **Aplicar el archivo de servicio de la aplicación:**
   - Aplica el archivo de servicio de la aplicación:
     ```sh
     kubectl apply -f app-service.yaml
     ```

3. **Obtener la dirección IP del Load Balancer:**
   - Verifica el estado del servicio y obtén la dirección IP externa:
     ```sh
     kubectl get services
     ```
   - Una vez que la columna `EXTERNAL-IP` tenga una dirección IP, accede a ella en tu navegador para ver la aplicación en funcionamiento.

4. **Crear table en el pod de POSTGRES:**
   - Verificar y obtener el nombre del pod:
    ```sh
      kubectl get pods
    ```
   -  Ingresar al pod: 
    ```sh
      kubectl exec -it <nombre-del-pod-postgres> -- psql -U myuser -d mydatabase
    ```
   - Ejecutar lo siguiente:
     ```sh
      \dt    
     ```
    ```sh
        
        CREATE TABLE my_table (
          id SERIAL PRIMARY KEY,
          name VARCHAR(255),
          value VARCHAR(255)
        );
  
      ```
### Paso 8: Verificar el despliegue

1. **Acceder a la aplicación:**
   - Abre un navegador y navega a la dirección IP externa del servicio.
   - Accede a la ruta `/data` para verificar que la aplicación puede leer y escribir datos en la base de datos PostgreSQL.

### Paso 9: Limpieza

1. **Eliminar los Recursos:**
   - Elimina los despliegues, servicios, PV y PVC:
     ```sh
     kubectl delete -f app-service.yaml
     kubectl delete -f app-deployment.yaml
     kubectl delete -f postgres-service.yaml
     kubectl delete -f postgres-deployment.yaml
     kubectl delete -f pvc.yaml
     kubectl delete -f pv.yaml
     ```
   - Elimina el clúster de GKE:
     ```sh
     gcloud container clusters delete my-cluster --zone [ZONE]
     ```

### Resumen

Con este laboratorio, has desplegado una aplicación que requiere almacenamiento persistente utilizando Persistent Volumes y Persistent Volume Claims en un clúster de Kubernetes en GKE. Esto demuestra habilidades en la gestión de almacenamiento persistente, configuración de volúmenes, respaldo y recuperación de datos.
