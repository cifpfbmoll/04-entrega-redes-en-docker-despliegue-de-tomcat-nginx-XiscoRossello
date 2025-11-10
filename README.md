# Tutorial: Despliegue de Tomcat con Nginx usando Docker

## Descripción del Proyecto

Este proyecto demuestra cómo configurar un servidor de aplicaciones Java (Tomcat) detrás de un proxy inverso Nginx utilizando Docker Compose. La configuración permite servir una aplicación Java WAR a través de Nginx, que actúa como proxy y punto de entrada.

## Arquitectura

El proyecto consta de dos contenedores Docker:
- **Tomcat 9.0**: Servidor de aplicaciones Java que ejecuta la aplicación `sample.war`
- **Nginx**: Servidor web que actúa como proxy inverso hacia Tomcat

## Estructura del Proyecto

```
tomcat-nginx/
├── docker-compose.yaml      # Configuración de los servicios Docker
├── default.conf             # Configuración del proxy Nginx
├── sample.war               # Aplicación Java de ejemplo
└── README.md                # Este documento
```

## Archivos de Configuración

### docker-compose.yaml

```yaml
version: '3.1'
services:
  aplicacionjava:
    container_name: tomcat
    image: tomcat:9.0
    restart: always
    volumes:
      - ./sample.war:/usr/local/tomcat/webapps/sample.war:ro
  proxy:
    container_name: nginx
    image: nginx
    ports:
      - 80:80
    volumes:
      - ./default.conf:/etc/nginx/conf.d/default.conf:ro
```

**Explicación:**
- **aplicacionjava (Tomcat)**: 
  - Utiliza la imagen oficial de Tomcat 9.0
  - Monta el archivo `sample.war` en el directorio de webapps
  - El volumen es de solo lectura (`:ro`)
  - Se reinicia automáticamente en caso de fallo

- **proxy (Nginx)**:
  - Utiliza la imagen oficial de Nginx
  - Expone el puerto 80 al host
  - Monta la configuración personalizada de Nginx
  - Actúa como proxy hacia el contenedor de Tomcat

### default.conf

```nginx
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        proxy_pass http://aplicacionjava:8080/sample/;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

**Explicación:**
- Nginx escucha en el puerto 80
- Todas las peticiones a `/` se redirigen a `http://aplicacionjava:8080/sample/`
- `aplicacionjava` es el nombre del servicio Docker que resuelve internamente
- Se configuran páginas de error personalizadas

## Pasos del Tutorial

### 1. Preparación de los archivos

Primero, asegúrate de tener todos los archivos necesarios en el directorio del proyecto:

![Estructura inicial del proyecto](imagenes/Captura%20de%20pantalla%202025-11-10%20a%20las%2017.23.12.png)

### 2. Crear el archivo docker-compose.yaml

Crea el archivo de configuración de Docker Compose con los servicios necesarios:

![Configuración de docker-compose.yaml](imagenes/Captura%20de%20pantalla%202025-11-10%20a%20las%2017.23.45.png)

### 3. Configurar Nginx (default.conf)

Configura el archivo de Nginx para que actúe como proxy inverso hacia Tomcat:

![Configuración de Nginx](imagenes/Captura%20de%20pantalla%202025-11-10%20a%20las%2017.25.33.png)

### 4. Levantar los contenedores

Ejecuta el siguiente comando para iniciar los servicios:

```bash
docker-compose up -d
```

![Ejecutando docker-compose up](imagenes/Captura%20de%20pantalla%202025-11-10%20a%20las%2017.25.40.png)

### 5. Verificar los contenedores

Comprueba que los contenedores están funcionando correctamente:

```bash
docker ps
```

![Verificación de contenedores](imagenes/Captura%20de%20pantalla%202025-11-10%20a%20las%2017.27.45.png)

Deberías ver dos contenedores en ejecución:
- `nginx` (proxy)
- `tomcat` (aplicacionjava)

### 6. Acceder a la aplicación

Abre un navegador y accede a `http://localhost`

![Aplicación funcionando](imagenes/Captura%20de%20pantalla%202025-11-10%20a%20las%2017.28.02.png)

### 7. Verificar el funcionamiento

La aplicación Java debería mostrarse correctamente a través del proxy Nginx:

![Aplicación Sample en funcionamiento](imagenes/Captura%20de%20pantalla%202025-11-10%20a%20las%2017.29.29.png)

![Detalles de la aplicación](imagenes/Captura%20de%20pantalla%202025-11-10%20a%20las%2017.29.48.png)

### 8. Monitoreo y logs

Para ver los logs de los contenedores:

```bash
# Ver logs de todos los servicios
docker-compose logs

# Ver logs de un servicio específico
docker-compose logs nginx
docker-compose logs tomcat
```

![Logs y estado final](imagenes/Captura%20de%20pantalla%202025-11-10%20a%20las%2017.32.36.png)

## Comandos Útiles

### Gestión de contenedores

```bash
# Iniciar los servicios
docker-compose up -d

# Detener los servicios
docker-compose down

# Ver el estado de los contenedores
docker-compose ps

# Ver logs en tiempo real
docker-compose logs -f

# Reiniciar los servicios
docker-compose restart
```

### Acceso a los contenedores

```bash
# Acceder al contenedor de Tomcat
docker exec -it tomcat bash

# Acceder al contenedor de Nginx
docker exec -it nginx bash
```

## Solución de Problemas

### El contenedor no inicia
- Verifica que los puertos no estén en uso: `lsof -i :80`
- Revisa los logs: `docker-compose logs`

### Error 502 Bad Gateway
- Asegúrate de que el contenedor de Tomcat está funcionando
- Verifica que la aplicación WAR se ha desplegado correctamente
- Comprueba la configuración del proxy en `default.conf`

### La aplicación no se carga
- Espera unos segundos, Tomcat puede tardar en desplegar la aplicación
- Verifica que el archivo `sample.war` existe y tiene permisos correctos
- Accede directamente a Tomcat: `http://localhost:8080/sample/` (si expones el puerto)

## Notas Importantes

1. **Volúmenes de solo lectura**: Los archivos montados usan `:ro` para evitar modificaciones accidentales
2. **Red Docker**: Los contenedores se comunican a través de la red interna de Docker
3. **Nombres de servicio**: Nginx usa el nombre del servicio `aplicacionjava` para resolver la dirección IP
4. **Puerto 8080**: Tomcat escucha en el puerto 8080 internamente, pero no se expone al host

## Mejoras Futuras

- Añadir volumen persistente para logs
- Configurar HTTPS con certificados SSL
- Implementar balanceo de carga con múltiples instancias de Tomcat
- Añadir healthchecks para los contenedores
- Configurar límites de recursos (CPU, memoria)

## Conclusión

Este tutorial demuestra cómo configurar un entorno de desarrollo con Tomcat y Nginx usando Docker Compose. Esta arquitectura es escalable y fácil de mantener, siendo ideal para aplicaciones Java en producción.

---

**Fecha**: 10 de noviembre de 2025  
**Tecnologías**: Docker, Docker Compose, Nginx, Tomcat 9.0
