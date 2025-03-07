### **Laboratorio: Implementación de ArgoCD en GKE con GitOps**

#### **Objetivo**
Configurar un cluster en GKE, instalar ArgoCD, conectarlo a un repositorio Git y aplicar el enfoque GitOps para gestionar aplicaciones de manera declarativa.

#### **Requisitos previos**
1. Una cuenta de Google Cloud Platform (GCP) con permisos para crear clusters GKE.
2. Herramientas instaladas localmente:
   - `gcloud` (Google Cloud SDK).
   - `kubectl` (CLI de Kubernetes).
   - `git` (para gestionar el repositorio).
   - Acceso a un terminal.
3. Un repositorio Git (puede ser GitHub, GitLab, etc.) donde almacenarás los manifests.

---

### **Paso 1: Crear un Cluster en GKE**
1. **Inicia sesión en GCP**:
   ```bash
   gcloud auth login
   ```
   Sigue las instrucciones para autenticarte.

2. **Configura tu proyecto en GCP**:
   Reemplaza `TU-PROYECTO-ID` con el ID de tu proyecto en GCP.
   ```bash
   gcloud config set project TU-PROYECTO-ID
   ```

3. **Crea el cluster GKE**:
   Ejecuta este comando para crear un cluster básico en GKE:
   ```bash
   gcloud container clusters create argocd-cluster \
     --region us-central1 \
     --num-nodes 2 \
     --machine-type e2-standard-2
   ```
   - `argocd-cluster`: Nombre del cluster.
   - `us-central1`: Región (ajústala según tu preferencia).
   - `num-nodes`: 2 nodos para este ejemplo.
   - `machine-type`: Tipo de máquina (puedes ajustarlo según necesidades).

4. **Conecta tu `kubectl` al cluster**:
   Una vez creado el cluster, obtén las credenciales:
   ```bash
   gcloud container clusters get-credentials argocd-cluster --region us-central1
   ```

5. **Verifica el acceso**:
   ```bash
   kubectl get nodes
   ```
   Deberías ver los nodos del cluster listados.

---

### **Paso 2: Instalar ArgoCD en el Cluster**
1. **Crea un namespace para ArgoCD**:
   ```bash
   kubectl create namespace argocd
   ```

2. **Instala ArgoCD**:
   Aplica los manifests oficiales de ArgoCD desde su repositorio:
   ```bash
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. **Verifica la instalación**:
   Espera unos minutos y comprueba que los pods estén corriendo:
   ```bash
   kubectl get pods -n argocd
   ```
   Deberías ver varios pods como `argocd-server`, `argocd-repo-server`, etc., en estado `Running`.

---

### **Paso 3: Acceder a la interfaz de ArgoCD**
1. **Expose el servicio de ArgoCD**:
   Por defecto, el servicio `argocd-server` no es accesible externamente. Cámbialo a tipo `LoadBalancer`:
   ```bash
   kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
   ```

2. **Obtén la IP externa**:
   Ejecuta:
   ```bash
   kubectl get svc argocd-server -n argocd
   ```
   Busca la columna `EXTERNAL-IP`. Puede tardar unos minutos en asignarse.

3. **Obtén la contraseña inicial del usuario `admin`**:
   La contraseña predeterminada está en un secreto:
   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
   ```
   Copia esta contraseña (es una cadena larga).

4. **Accede a la UI**:
   Abre tu navegador y ve a `https://<EXTERNAL-IP>`. Acepta el certificado autofirmado si es necesario. Inicia sesión con:
   - Usuario: `admin`
   - Contraseña: La obtenida en el paso anterior.

---

### **Paso 4: Conectar ArgoCD con el Cluster**
ArgoCD ya está instalado en el cluster, por lo que ya está "conectado" a sí mismo. Sin embargo, si deseas gestionar otros clusters, puedes agregarlos más adelante mediante la CLI o la UI.

Para este laboratorio, trabajaremos con el mismo cluster GKE:
1. **Instala la CLI de ArgoCD (opcional)**:
   Descarga la CLI desde [ Releases de ArgoCD](https://github.com/argoproj/argo-cd/releases) o usa este comando en Linux/macOS:
   ```bash
   curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
   chmod +x argocd
   sudo mv argocd /usr/local/bin/
   ```

2. **Inicia sesión en la CLI**:
   Usa la IP externa del paso anterior:
   ```bash
   argocd login <EXTERNAL-IP> --username admin --password <CONTRASEÑA>
   ```

---

### **Paso 5: Conectar un repositorio Git con ArgoCD**
1. **Crea un repositorio Git**:
   - Crea un repositorio en GitHub/GitLab (público o privado).
   - Dentro del repositorio, crea una carpeta como `manifests/` y añade un archivo YAML simple, por ejemplo:
     ```yaml
     # manifests/nginx.yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: nginx-deployment
       namespace: default
     spec:
       replicas: 2
       selector:
         matchLabels:
           app: nginx
       template:
         metadata:
           labels:
             app: nginx
         spec:
           containers:
           - name: nginx
             image: nginx:latest
             ports:
             - containerPort: 80
     ```

2. **Conecta el repositorio a ArgoCD**:
   Desde la UI:
   - Ve a **Settings > Repositories**.
   - Haz clic en **Connect Repo using HTTPS**.
   - Ingresa:
     - URL del repositorio (ej. `https://github.com/tu-usuario/tu-repo.git`).
     - Si es privado, añade el usuario y token/password.
   O con la CLI:
   ```bash
   argocd repo add https://github.com/tu-usuario/tu-repo.git --username <USUARIO> --password <TOKEN>
   ```

---

### **Paso 6: Crear una aplicación con GitOps**
1. **Define la aplicación en ArgoCD**:
   Desde la UI:
   - Ve a **Applications > + New App**.
   - Completa:
     - **Application Name**: `nginx-app`.
     - **Project**: `default`.
     - **Sync Policy**: `Manual` (cámbialo a `Automatic` después si deseas sincronización automática).
     - **Repository URL**: URL de tu repositorio Git.
     - **Path**: `manifests/` (donde está el archivo `nginx.yaml`).
     - **Cluster**: `https://kubernetes.default.svc` (el cluster local).
     - **Namespace**: `default`.
   - Haz clic en **Create**.

   O con la CLI:
   ```bash
   argocd app create nginx-app \
     --repo https://github.com/tu-usuario/tu-repo.git \
     --path manifests \
     --dest-server https://kubernetes.default.svc \
     --dest-namespace default
   ```

2. **Sincroniza la aplicación**:
   Desde la UI:
   - Haz clic en la app y presiona **Sync**.
   O con la CLI:
   ```bash
   argocd app sync nginx-app
   ```

3. **Verifica el despliegue**:
   ```bash
   kubectl get pods -n default
   ```
   Deberías ver los pods de Nginx corriendo.

---

### **Paso 7: Prueba el enfoque GitOps**
1. Modifica el archivo `nginx.yaml` en tu repositorio Git, por ejemplo, cambia `replicas: 2` a `replicas: 3`.
2. Si configuraste sincronización automática, ArgoCD detectará el cambio y actualizará el cluster. Si es manual, ejecuta:
   ```bash
   argocd app sync nginx-app
   ```
3. Verifica que el número de réplicas cambió:
   ```bash
   kubectl get deployments -n default
   ```

---

### **Limpieza (Opcional)**
Si deseas eliminar todo:
1. Elimina la aplicación:
   ```bash
   argocd app delete nginx-app
   ```
2. Desinstala ArgoCD:
   ```bash
   kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```
3. Elimina el cluster GKE:
   ```bash
   gcloud container clusters delete argocd-cluster --region us-central1
   ```

---

