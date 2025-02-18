### Laboratorio: Implementación de Helm en Kubernetes con una Aplicación Propia

#### Objetivo:
Implementar Helm en Kubernetes para gestionar una aplicación propia de manera eficiente.

#### Requisitos Previos:
1. Cuenta de Google Cloud Platform (GCP).
2. Google Cloud SDK instalado y configurado.
3. kubectl instalado y configurado.
4. Helm instalado.
5. Clúster de Kubernetes en GKE.
6. Sistema operativo Linux.

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

### Paso 3: Instalar Helm

1. **Instalar Helm:**
   - Si no tienes Helm instalado, puedes instalarlo siguiendo las instrucciones oficiales de Helm. Aquí tienes un ejemplo para instalar Helm en Linux:
     ```sh
     curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
     ```

2. **Inicializar Helm:**
   - Inicializa Helm en tu clúster de Kubernetes:
     ```sh
     helm repo add stable https://charts.helm.sh/stable
     helm repo update
     ```

### Paso 4: Crear una aplicación propia

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

     app = Flask(__name__)

     @app.route('/')
     def hello():
         return jsonify({'message': 'Hello from my custom app!'})

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

### Paso 5: Crear un Chart de Helm para la aplicación

1. **Crear la estructura del Chart de Helm:**
   - Crea la estructura básica del chart de Helm:
     ```sh
     helm create my-app-chart
     ```
*Para facilidad de editar estos archivos usando nano, presiona Alt+A, te posicionas al inicio del documento y vas seleccionando hasta    el final, posteriormente, presionar Ctrl+K. Luego copiar lo siguiente:*


2. **Modificar el archivo `values.yaml`:**
   
   - Edita el archivo `my-app-chart/values.yaml` para incluir la imagen de tu aplicación:
     ```yaml
     replicaCount: 3

     image:
       repository: gcr.io/[PROJECT_ID]/my-app
       tag: "v1"
       pullPolicy: IfNotPresent

     service:
       type: LoadBalancer
       port: 80
     ```

4. **Modificar el archivo `deployment.yaml`:**

   
   - Edita el archivo `my-app-chart/templates/deployment.yaml` para asegurarte de que utiliza las variables definidas en `values.yaml`:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: {{ include "my-app-chart.fullname" . }}
       labels:
         app: {{ include "my-app-chart.name" . }}
     spec:
       replicas: {{ .Values.replicaCount }}
       selector:
         matchLabels:
           app: {{ include "my-app-chart.name" . }}
       template:
         metadata:
           labels:
             app: {{ include "my-app-chart.name" . }}
         spec:
           containers:
             - name: {{ .Chart.Name }}
               image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
               ports:
                 - containerPort: 5000
     ```

5. **Modificar el archivo `service.yaml`:**
   - Edita el archivo `my-app-chart/templates/service.yaml` para asegurarte de que utiliza las variables definidas en `values.yaml`:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: {{ include "my-app-chart.fullname" . }}
       labels:
         app: {{ include "my-app-chart.name" . }}
     spec:
       type: {{ .Values.service.type }}
       ports:
         - port: {{ .Values.service.port }}
           targetPort: 5000
       selector:
         app: {{ include "my-app-chart.name" . }}
     ```

### Paso 6: Instalar la aplicación con Helm

1. **Instalar la aplicación:**

   *Nota: Antes de instalar la aplicación el chart de Helm, asegurate de eliminar los archivos de la carpeta template: hpa.yaml, ingress.yaml, serviceaccount.yaml, NOTES.txt*
   
   - Instala la aplicación utilizando el chart de Helm:
     ```sh
     helm install my-app ./my-app-chart
     ```

3. **Verificar la instalación:**
   - Verifica que la aplicación se haya instalado correctamente:
     ```sh
     helm list
     ```
   - Deberías ver `my-app` en la lista de aplicaciones instaladas.

### Paso 7: Acceder a la aplicación

1. **Obtener la dirección IP del servicio:**
   - Obtén la dirección IP externa del servicio de `my-app`:
     ```sh
     kubectl get svc
     ```
   - Anota la dirección IP externa del servicio `my-app`.

2. **Acceder a la aplicación:**
   - Abre un navegador y navega a la dirección IP externa del servicio. Deberías ver el mensaje "Hello from my custom app!".

### Paso 8: Actualizar la aplicación con Helm

1. **Actualizar la aplicación:**
   - Actualiza la aplicación `my-app` para cambiar la imagen a una nueva versión:
     ```sh
     helm upgrade my-app ./my-app-chart --set image.tag=v2
     ```

2. **Verificar la actualización:**
   - Verifica que la aplicación se haya actualizado correctamente:
     ```sh
     helm list
     ```

### Paso 9: Eliminar la aplicación con Helm

1. **Eliminar la aplicación:**
   - Elimina la aplicación `my-app` instalada con Helm:
     ```sh
     helm uninstall my-app
     ```

2. **Verificar la eliminación:**
   - Verifica que la aplicación se haya eliminado correctamente:
     ```sh
     helm list
     ```

### Paso 10: Limpieza

1. **Eliminar los recursos:**
   - Elimina el clúster de GKE:
     ```sh
     gcloud container clusters delete my-cluster --zone us-central1-a
     ```

### Resumen

Con este laboratorio, has implementado Helm en Kubernetes para gestionar una aplicación propia de manera eficiente. Esto demuestra habilidades en la instalación, actualización y gestión de aplicaciones en Kubernetes utilizando Helm.
