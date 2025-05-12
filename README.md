# Configuración de Odoo en Google Cloud con Docker y Nginx

Este documento guía la instalación y configuración de Odoo en Google Cloud Platform utilizando Docker y Nginx como proxy inverso.

## Paso 0: Configuración del Firewall
Antes de comenzar, es crucial configurar el firewall de Google Cloud para permitir el tráfico en el puerto 9069.


## Paso 1: Verificar Espacio en Disco
```bash
df -h
```
Este comando muestra el espacio disponible en el disco. Es importante tener suficiente espacio para la instalación de Odoo y sus datos.

## Paso 2: Actualizar Paquetes del Sistema
```bash
sudo apt update
sudo apt-get update
```
Actualiza la lista de paquetes disponibles y sus versiones. Es importante mantener el sistema actualizado para la seguridad y compatibilidad.

## Paso 3: Instalar Docker
```bash
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
```
- Instala Docker Engine
- Inicia el servicio Docker
- Configura Docker para iniciar automáticamente al arrancar el sistema

## Paso 4: Agregar Usuario al Grupo Docker
```bash
whoami
sudo usermod -aG docker user
exit
```
- Verifica el usuario actual
- Agrega el usuario al grupo docker para ejecutar comandos sin sudo
- Es necesario cerrar sesión y volver a entrar para que los cambios surtan efecto

## Paso 5: Instalar Docker Compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```
Instala Docker Compose, una herramienta para definir y ejecutar aplicaciones Docker multi-contenedor.

## Paso 6: Instalar Git
```bash
sudo apt install -y git
```
Instala Git para clonar el repositorio de Odoo.

## Paso 7: Clonar Repositorio de Odoo
```bash
git clone https://github.com/suarezjoa/odoo-google-cloud.git
cd Odoo
git switch Development/Production
```
Clona el repositorio y cambia a la rama de desarrollo/producción.

## Paso 8: Iniciar Docker Compose
```bash
docker-compose up
sudo docker-compose up -d
```
- Primero comando: inicia los contenedores en primer plano (para ver logs)
- Segundo comando: inicia los contenedores en segundo plano

## Paso 9: Reiniciar Docker Compose
```bash
docker-compose down
docker-compose up
```
Detiene y reinicia los contenedores, útil después de cambios en la configuración.


## Paso 10: Instalar Certbot y Nginx
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo apt install -y nginx
```
Instala Nginx como servidor web y Certbot para certificados SSL.

## Paso 11: Configurar Nginx
```bash
sudo nano /etc/nginx/sites-available/Odoo
```
Configura Nginx como proxy inverso:
```nginx
server {
    listen 80;
    server_name YOUR-DOMAIN-NAME;

    location / {
        proxy_pass http://localhost:9069;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;

        # Headers para soporte WebSocket
        proxy_set_header Connection 'Upgrade';
        proxy_set_header Upgrade $http_upgrade;

        # Headers adicionales
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Paso 12: Habilitar Sitio Nginx
```bash
sudo ln -s /etc/nginx/sites-available/Odoo /etc/nginx/sites-enabled/
sudo nginx -t
```
Crea enlace simbólico y verifica la configuración de Nginx.

## Paso 13: Reiniciar Nginx
```bash
sudo systemctl restart nginx
```
Aplica los cambios en la configuración de Nginx.

## Paso 14: Configurar SSL con Certbot
```bash
sudo certbot --nginx -d YOUR-DOMAIN-NAME
```
Obtiene y configura certificado SSL para el dominio.

## Paso 15: Reiniciar Nginx
```bash
sudo systemctl restart nginx
```
Aplica la configuración SSL.

## Prueba de la Aplicación
Accede a tu aplicación Odoo a través de:
```bash
http://<tu-dominio>:9069
```

## Notas Importantes
- Reemplaza `YOUR-DOMAIN-NAME` con tu dominio real
- Asegúrate de que los puertos necesarios estén abiertos en el firewall
- Mantén las contraseñas seguras y no las compartas
- Realiza copias de seguridad regularmente

