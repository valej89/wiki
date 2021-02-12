# Ambiente Test-Revisión
La función de este ambiente es facilitar el trabajo en conjunto con la Editorial en Alianza para cuestiones referentes a revisión, corrección, validación y certificación de aspectos cualitativos, funcionales o pedagógicos de la plataforma Siembra Digital.

---
## EC2 ELB
El punto de entrada de la aplicación es mediante el Aplication Load Balancer configurado.
La instancia en particular es: **sd-cf-alb**

El APL tiene configurado dos listeners:
- Puerto 80: El cual tiene configurado un *redirect rule* al listener del puerto 443 de manera a evitar conexiones no SSL.
```
HTTPS://#{host}:443/#{path}?#{query}
```
- Puerto 443: Sobre este puerto llegan todas las peticiones y son cifradas mediante el certificado configurado previamente.

---
## EC2
Las instancias en donde se ejecuta la plataforma del ambiente Test-Revisión se ejecutan en dos Avalability Zone:
- us-east-1b
- us-east-1c

| Nombre | Tipo de Instancia | Avalability Zone | Repositorio |
| --------|---------|-------|------|
| sd-cf-frontend | t2.small | us-east-1b | **sd-ui/** PROTOTYPE |
| sd-cf-backend | t2.micro | us-east-1b | **sd-backend/** PROTOTYPE |
| sd-cf-dev-frontend | t2.small | us-east-1b | **sd-ui/** DEV |
| sd-cf-dev-backend | t2.small | us-east-1b | **sd-backend/** DEV |
| sd-cf-drupal | t2.micro | us-east-1c | **sd-docker-drupal8** |

### Configuraciones Generales
Instancias con **Ubuntu 20.04 LTS** utilizando el segmento de red **10.0.33.0/24**, excepto **sd-cf-drupal** que tiene configurada la red **10.0.49.0/24**.

| Tipo de Instancia | vCPU | Memoria |
| --- | --- | --- |
| t2.micro | 1 | 1 GB |
| t2.small | 1 | 2 GB |

### Configuraciones particulares

#### Instancias sd-*-frontend
Las intancias ejecutan **Nginx** para servir el contenido de la Web sobre el puerto **80**.

Las configuraciones de sitios disponibles del Nginx se encuentran en **/etc/nginx/sites-available** y las configuraciones vigentes están en **/etc/nginx/sites-enabled**.

El archivo de configuraciones del Nginx en el ambiente de **Test-Revisión** está en **/etc/nginx/sites-available**
```
server {
	listen 80;
	listen [::]:80;

	root /var/www/html/build;
	index index.html index.htm index.nginx-debian.html;

	location / {
		try_files $uri /index.html;
	}
}
```

El contenido de la web se ubica dentro del directorio **/var/www/html/build** y el se basa en el repositorio **sd-ui**.

El archivo que indica las configuraciones personalizadas para la generación del paquete se encuentran en el archivo **.env-cmdrc**
```
{
    "testrevision": {
        "REACT_APP_API_URL": "https://test-revision.siembradigital.com.py/app/api",
        "REACT_APP_PUBLIC_URL": "/app",
        "PUBLIC_URL": "/app",
        "REACT_APP_S3_FILE_UPLOAD": "https://sd-upload-files.s3-us-west-2.amazonaws.com",
        "REACT_APP_AUTH_REGION": "us-east-1",
        "REACT_APP_AUTH_USER_POOL_ID": "us-east-1_PhC1WvdF8",
        "REACT_APP_AUTH_USER_POOL_WEB_CLIENT_ID": "6rf3hlbsi78p6tecub6gvq8p02"
    },
}
```

#### Instancias sd-*-backend
Estos servidores ejecutan servicios construidos en Java, por lo tanto se realizaron configuraciones para poder desplegar los servicios como demonios en Linux.

Las configuraciones son guardadas en **/etc/systemd/system/sd.service**:
```
[Unit]
Description=SD Server Daemon

[Service]
ExecStart=/usr/bin/java -jar /opt/sd-backend.jar --spring.config.location=file:/etc/sd/application.properties --logging.file.name=/var/log/spring/spring-app.log
User=ubuntu

[Install]
WantedBy=multi-user.target
```

Las configuraciones del servicio se encuentran en **/etc/sd/application.properties**
```
host.port=443
host.url=https://test-revision-drupal.siembradigital.com.py:${host.port}

decouplerouter.url=${host.url}/router/translate-path
drupal.url=${host.url}/jsonapi


server.error.include-stacktrace=never

# DATABASE
#spring.jpa.hibernate.ddl-auto=update
spring.datasource.url=jdbc:mysql://[RDS_endpoint]:[PUERTO]/sd_preprod_prototype
spring.datasource.username=[USUARIO]
spring.datasource.password=[CONTRASEÑA]

server.servlet.context-path=/api
```

El paquete Java **sd-backend.jar** se encuentra ubicado en el directorio **/opt**.

---
## Base de Datos
El ambiente **test-revision** utiliza no utiliza bases de datos sobre RDS, actualmente cuenta con un motor MySQL instalado de manera local en cada servidor sd-*-backend.

Los esquemas de base de datos disponibles son:
- siembradigital **(sd-cf-backend)**
- siembradigital **(sd-cf-dev-backend)**

---
## S3
Para el almacenamiento de datos, así como para todo contenido estático necesario por los servicios. Para esto, se encuentra configurado en el archivo de entorno **.env-cmdrc** utilizado para la construcción del backend de los servicios de Siembra Digital.

En particular para el ambiente test-revisión el bucket configurado tiene el siguiente endpoint: **sd-upload-files**

El contenido estático se encuentra en: **assets-sd**

---
## Cognito
Para la gestión de usuarios de Siembra Digital se encuentra habilitado un **User Pool** dentro de Cognito. La instancia de cognito se encuentra en **us-east** (North Virginia), identificado como **sd-testrevision-userpool**.

Las configuraciones de AWS Cognito se realizan en el archivo **.env-cmdrc** del repositorio **sd-ui**.