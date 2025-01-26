**Cheatsheet completo de Kubernetes**, que incluye tanto los comandos básicos.

---

## **Cheatsheet completo de Kubernetes**

---

### **Conceptos básicos**
- **Pod**: La unidad más pequeña en Kubernetes, que puede contener uno o más contenedores.
- **Deployment**: Gestiona la creación y actualización de Pods.
- **Service**: Expone un conjunto de Pods como un servicio de red.
- **Namespace**: Aísla recursos dentro de un clúster.
- **Node**: Máquina física o virtual que ejecuta los Pods.
- **ConfigMap**: Almacena configuración no confidencial en pares clave-valor.
- **Secret**: Almacena información confidencial, como contraseñas o tokens.
- **Volume**: Almacenamiento persistente para los Pods.

---

### **Comandos básicos**

#### Obtener información
```bash
kubectl get pods                     # Listar todos los Pods
kubectl get deployments              # Listar todos los Deployments
kubectl get services                 # Listar todos los Services
kubectl get nodes                    # Listar todos los Nodes
kubectl get namespaces               # Listar todos los Namespaces
kubectl get configmaps               # Listar todos los ConfigMaps
kubectl get secrets                  # Listar todos los Secrets
kubectl get all                      # Listar todos los recursos
```

#### Describir recursos
```bash
kubectl describe pod <pod-name>      # Describir un Pod
kubectl describe node <node-name>    # Describir un Node
kubectl describe svc <service-name>  # Describir un Service
```

#### Crear recursos
```bash
kubectl create deployment <name> --image=<image>  # Crear un Deployment
kubectl create namespace <name>                  # Crear un Namespace
kubectl create configmap <name> --from-literal=<key>=<value>  # Crear un ConfigMap
kubectl create secret generic <name> --from-literal=<key>=<value>  # Crear un Secret
```

#### Eliminar recursos
```bash
kubectl delete pod <pod-name>        # Eliminar un Pod
kubectl delete deployment <name>     # Eliminar un Deployment
kubectl delete service <name>        # Eliminar un Service
kubectl delete namespace <name>      # Eliminar un Namespace
```

#### Ejecutar comandos en un Pod
```bash
kubectl exec -it <pod-name> -- /bin/sh  # Conectar a un Pod (shell interactivo)
kubectl exec <pod-name> -- <command>    # Ejecutar un comando en un Pod
```

#### Logs
```bash
kubectl logs <pod-name>               # Ver logs de un Pod
kubectl logs -f <pod-name>            # Ver logs en tiempo real (follow)
kubectl logs <pod-name> -c <container> # Ver logs de un contenedor específico
```

#### Escalar recursos
```bash
kubectl scale deployment <name> --replicas=<number>  # Escalar un Deployment
```

#### Actualizar recursos
```bash
kubectl apply -f <file.yaml>          # Aplicar un archivo YAML
kubectl set image deployment/<name> <container>=<new-image>  # Actualizar la imagen de un Deployment
```

#### Port-forward
```bash
kubectl port-forward <pod-name> <local-port>:<pod-port>  # Redirigir un puerto local a un Pod
```

---

### **Namespaces**

#### Listar namespaces
```bash
kubectl get namespaces  # o kubectl get ns
```

#### Crear un namespace
```bash
kubectl create namespace <nombre-del-namespace>
```

#### Eliminar un namespace
```bash
kubectl delete namespace <nombre-del-namespace>
```

#### Cambiar el namespace por defecto
```bash
kubectl config set-context --current --namespace=<nombre-del-namespace>
```

#### Ver recursos en un namespace específico
```bash
kubectl get pods -n <nombre-del-namespace>  # Listar Pods en un Namespace
kubectl get all -n <nombre-del-namespace>   # Listar todos los recursos en un Namespace
```

#### Crear un recurso en un namespace específico
```bash
kubectl create deployment <nombre> --image=<imagen> -n <nombre-del-namespace>
```

#### Eliminar todos los recursos en un namespace
```bash
kubectl delete all --all -n <nombre-del-namespace>
```

#### Archivo YAML de un namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: <nombre-del-namespace>
```

---

### **Archivos YAML**

#### Ejemplo de Deployment
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
      - name: my-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

#### Ejemplo de Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

#### Ejemplo de ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  key1: value1
  key2: value2
```

#### Ejemplo de Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: <base64-encoded>
  password: <base64-encoded>
```

---

### **Manejo de contextos y configuración**
```bash
kubectl config view                  # Ver la configuración de kubectl
kubectl config get-contexts          # Listar todos los contextos
kubectl config use-context <name>    # Cambiar de contexto
kubectl config set-context --current --namespace=<name>  # Cambiar el Namespace actual
```

---

### **Manejo de volúmenes**
```bash
kubectl get pv                       # Listar Persistent Volumes
kubectl get pvc                      # Listar Persistent Volume Claims
```

---

### **Autocompletado**
```bash
source <(kubectl completion bash)    # Habilitar autocompletado en Bash
```

---

### **Depuración**
```bash
kubectl get events                   # Ver eventos del clúster
kubectl top nodes                    # Ver uso de recursos en los Nodes
kubectl top pods                     # Ver uso de recursos en los Pods
```

---

### **Herramientas adicionales**
- **kubens**: Cambia fácilmente entre Namespaces.
- **kubectx**: Cambia entre contextos de clúster.

Instalación:
```bash
brew install kubectx
```

Uso:
```bash
kubens <nombre-del-namespace>  # Cambiar al Namespace especificado
kubens                         # Listar Namespaces y seleccionar uno
```

---

### **Consejos**
1. Usa Namespaces para separar entornos (dev, staging, prod).
2. Evita usar el Namespace `default` para aplicaciones críticas.
3. Asegúrate de que los equipos tengan acceso solo a los Namespaces que necesitan.

---

¡Espero que este cheatsheet completo te sea de mucha ayuda! 🚀
