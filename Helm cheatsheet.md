Aquí tienes un **cheatsheet detallado de Helm**, la herramienta de gestión de paquetes para Kubernetes. Este resumen cubre los comandos y conceptos más comunes que necesitarás al trabajar con Helm.

<img src="https://icons.veryicon.com/png/o/business/vscode-program-item-icon/helm-1.png" width="300" height="300"/>

---


## **Conceptos básicos de Helm**
- **Chart**: Un paquete de Helm que contiene la información necesaria para desplegar una aplicación en Kubernetes (archivos YAML, plantillas, etc.).
- **Release**: Una instancia de un Chart desplegado en un cluster de Kubernetes.
- **Repository**: Un repositorio donde se almacenan y comparten Charts.
- **Values**: Variables personalizables que se pasan a un Chart para configurar el despliegue.

---

## **Comandos básicos de Helm**

### **Inicialización y Configuración**
- Inicializar Helm en el cluster:
  ```bash
  helm init
  ```
- Ver la versión de Helm:
  ```bash
  helm version
  ```
- Agregar un repositorio de Charts:
  ```bash
  helm repo add <nombre_repo> <url_repo>
  ```
- Actualizar la lista de repositorios:
  ```bash
  helm repo update
  ```
- Listar repositorios agregados:
  ```bash
  helm repo list
  ```

---

### **Instalación y despliegue**
- Instalar un Chart desde un repositorio:
  ```bash
  helm install <nombre_release> <nombre_repo>/<nombre_chart>
  ```
- Instalar un Chart desde un archivo local:
  ```bash
  helm install <nombre_release> <ruta_al_chart>
  ```
- Instalar un Chart con valores personalizados (usando un archivo `values.yaml`):
  ```bash
  helm install <nombre_release> <nombre_chart> -f valores.yaml
  ```
- Instalar un Chart con valores personalizados (usando parámetros en línea):
  ```bash
  helm install <nombre_release> <nombre_chart> --set key=value
  ```
- Verificar la instalación de un Release:
  ```bash
  helm status <nombre_release>
  ```

---

### **Actualización y rollback**
- Actualizar un Release con un Chart nuevo o valores actualizados:
  ```bash
  helm upgrade <nombre_release> <nombre_chart>
  ```
- Realizar un rollback a una versión anterior de un Release:
  ```bash
  helm rollback <nombre_release> <revision_number>
  ```
- Ver el historial de revisiones de un Release:
  ```bash
  helm history <nombre_release>
  ```

---

### **Listado y búsqueda**
- Listar todos los Releases desplegados:
  ```bash
  helm list
  ```
- Buscar Charts en repositorios agregados:
  ```bash
  helm search repo <nombre_chart>
  ```
- Buscar Charts en el Hub de Helm (Artifact Hub):
  ```bash
  helm search hub <nombre_chart>
  ```

---

### **Eliminación**
- Eliminar un Release:
  ```bash
  helm uninstall <nombre_release>
  ```
- Eliminar todos los Releases:
  ```bash
  helm uninstall $(helm list -q)
  ```

---

### **Creación y dersonalización de Charts**
- Crear un nuevo Chart:
  ```bash
  helm create <nombre_chart>
  ```
- Validar la sintaxis de un Chart:
  ```bash
  helm lint <ruta_al_chart>
  ```
- Empaquetar un Chart para compartirlo:
  ```bash
  helm package <ruta_al_chart>
  ```
- Probar la plantilla de un Chart (sin instalarlo):
  ```bash
  helm template <nombre_release> <ruta_al_chart>
  ```

---

### **Gestión de dependencias**
- Actualizar las dependencias de un Chart:
  ```bash
  helm dependency update <ruta_al_chart>
  ```
- Listar las dependencias de un Chart:
  ```bash
  helm dependency list <ruta_al_chart>
  ```

---

### **Plugins**
- Listar plugins instalados:
  ```bash
  helm plugin list
  ```
- Instalar un plugin:
  ```bash
  helm plugin install <url_plugin>
  ```
- Desinstalar un plugin:
  ```bash
  helm plugin uninstall <nombre_plugin>
  ```

---

### **Comandos avanzados**
- Ver los valores por defecto de un Chart:
  ```bash
  helm show values <nombre_chart>
  ```
- Descargar un Chart sin instalarlo:
  ```bash
  helm pull <nombre_repo>/<nombre_chart>
  ```
- Exportar un Release a un archivo YAML:
  ```bash
  helm get manifest <nombre_release> > release.yaml
  ```

---

### **Variables y plantillas**
- Usar valores personalizados en un Chart:
  ```yaml
  # values.yaml
  replicaCount: 3
  image:
    repository: nginx
    tag: latest
  ```
- Acceder a valores en una plantilla:
  ```yaml
  # deployment.yaml
  replicas: {{ .Values.replicaCount }}
  image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
  ```

---

### **Consejos y buenas prácticas**
- Usa `helm lint` para validar tus Charts antes de desplegarlos.
- Utiliza `helm template` para depurar y probar tus plantillas.
- Mantén tus Charts en un repositorio Git para facilitar la colaboración.
- Usa `helm dependency update` para gestionar dependencias externas.

---

Este cheatsheet cubre la mayoría de los comandos y conceptos esenciales de Helm. ¡Espero que te sea útil! Si necesitas más detalles, consulta la [documentación oficial de Helm](https://helm.sh/docs/).
