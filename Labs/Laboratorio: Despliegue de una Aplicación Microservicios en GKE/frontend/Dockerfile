FROM nginx:alpine

# Eliminar el archivo de configuración predeterminado de Nginx
RUN rm /etc/nginx/nginx.conf

# Copiar tu archivo nginx.conf
COPY nginx.conf /etc/nginx/nginx.conf

# Copiar los archivos estáticos (si es necesario)
COPY . /usr/share/nginx/html
