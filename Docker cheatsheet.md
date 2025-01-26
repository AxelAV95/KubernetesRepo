Aquí tienes un **cheatsheet completo de Docker** con los comandos y conceptos más importantes para trabajar con contenedores. Este resumen cubre desde la instalación hasta la gestión avanzada de contenedores, imágenes, redes y volúmenes.

<img src="https://static-00.iconduck.com/assets.00/docker-icon-2048x1753-uguk29a7.png" width="350" height="300"/>

---

## **Instalación de Docker**
1. **Linux (Ubuntu/Debian)**:
   ```bash
   sudo apt update
   sudo apt install docker.io
   sudo systemctl start docker
   sudo systemctl enable docker
   ```
2. **Windows/macOS**:
   - Descarga Docker Desktop desde [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop).

---

## **Comandos básicos de Docker**

### **Gestión de contenedores**
- **Ejecutar un contenedor**:
  ```bash
  docker run <imagen>
  ```
  Ejemplo:
  ```bash
  docker run hello-world
  ```

- **Ejecutar un contenedor en segundo plano**:
  ```bash
  docker run -d <imagen>
  ```

- **Listar contenedores en ejecución**:
  ```bash
  docker ps
  ```

- **Listar todos los contenedores (activos e inactivos)**:
  ```bash
  docker ps -a
  ```

- **Detener un contenedor**:
  ```bash
  docker stop <container_id>
  ```

- **Eliminar un contenedor**:
  ```bash
  docker rm <container_id>
  ```

- **Eliminar todos los contenedores detenidos**:
  ```bash
  docker container prune
  ```

- **Ver logs de un contenedor**:
  ```bash
  docker logs <container_id>
  ```

- **Ejecutar comandos dentro de un contenedor**:
  ```bash
  docker exec -it <container_id> <comando>
  ```
  Ejemplo:
  ```bash
  docker exec -it <container_id> bash
  ```

---

### **Gestión de imágenes**
- **Descargar una imagen**:
  ```bash
  docker pull <imagen>
  ```

- **Listar imágenes locales**:
  ```bash
  docker images
  ```

- **Eliminar una imagen**:
  ```bash
  docker rmi <imagen_id>
  ```

- **Eliminar imágenes no utilizadas**:
  ```bash
  docker image prune
  ```

- **Construir una imagen desde un Dockerfile**:
  ```bash
  docker build -t <nombre_imagen> .
  ```

- **Etiquetar una imagen**:
  ```bash
  docker tag <imagen_id> <nombre_imagen>:<tag>
  ```

---

### **Redes en Docker**
- **Listar redes**:
  ```bash
  docker network ls
  ```

- **Crear una red**:
  ```bash
  docker network create <nombre_red>
  ```

- **Conectar un contenedor a una red**:
  ```bash
  docker network connect <nombre_red> <container_id>
  ```

- **Desconectar un contenedor de una red**:
  ```bash
  docker network disconnect <nombre_red> <container_id>
  ```

- **Inspeccionar una red**:
  ```bash
  docker network inspect <nombre_red>
  ```

---

### **Volúmenes en docker**
- **Listar volúmenes**:
  ```bash
  docker volume ls
  ```

- **Crear un volumen**:
  ```bash
  docker volume create <nombre_volumen>
  ```

- **Eliminar un volumen**:
  ```bash
  docker volume rm <nombre_volumen>
  ```

- **Montar un volumen en un contenedor**:
  ```bash
  docker run -v <nombre_volumen>:<ruta_en_contenedor> <imagen>
  ```

- **Eliminar volúmenes no utilizados**:
  ```bash
  docker volume prune
  ```

---

### **Docker Compose**
- **Iniciar servicios**:
  ```bash
  docker-compose up
  ```

- **Iniciar servicios en segundo plano**:
  ```bash
  docker-compose up -d
  ```

- **Detener servicios**:
  ```bash
  docker-compose down
  ```

- **Listar servicios**:
  ```bash
  docker-compose ps
  ```

- **Ver logs de servicios**:
  ```bash
  docker-compose logs
  ```

---

### **Configuración y mantenimiento**
- **Ver información del sistema Docker**:
  ```bash
  docker info
  ```

- **Ver versión de Docker**:
  ```bash
  docker --version
  ```

- **Limpiar recursos no utilizados**:
  ```bash
  docker system prune
  ```

- **Limpiar todo (imágenes, contenedores, volúmenes, redes)**:
  ```bash
  docker system prune -a --volumes
  ```

---

### **Dockerfile (Sintaxis Básica)**
- **FROM**: Especifica la imagen base.
  ```dockerfile
  FROM ubuntu:20.04
  ```

- **RUN**: Ejecuta comandos durante la construcción.
  ```dockerfile
  RUN apt update && apt install -y curl
  ```

- **COPY**: Copia archivos al contenedor.
  ```dockerfile
  COPY app.py /app/
  ```

- **WORKDIR**: Establece el directorio de trabajo.
  ```dockerfile
  WORKDIR /app
  ```

- **EXPOSE**: Expone un puerto.
  ```dockerfile
  EXPOSE 80
  ```

- **CMD**: Define el comando por defecto al ejecutar el contenedor.
  ```dockerfile
  CMD ["python", "app.py"]
  ```

- **ENTRYPOINT**: Similar a CMD, pero permite pasar argumentos.
  ```dockerfile
  ENTRYPOINT ["python"]
  ```

---

### **Consejos y buenas prácticas**
1. Usa `.dockerignore` para excluir archivos innecesarios en la construcción de imágenes.
2. Minimiza el número de capas en las imágenes para reducir su tamaño.
3. Usa volúmenes para persistir datos importantes.
4. Evita ejecutar contenedores como root siempre que sea posible.
5. Usa Docker Compose para gestionar aplicaciones multi-contenedor.

---

Este cheatsheet cubre los aspectos más importantes de Docker. ¡Espero que te sea útil! Si necesitas más detalles sobre algún comando o concepto, no dudes en preguntar. 🐳
