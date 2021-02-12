# Arquitectura del Proyecto
En el presente apartado se describe la arquitectura de componentes del proyecto.

![Diagrama General](https://bitbucket.org/softwarenatura/sd-wiki/src/master/images/diagrama-arquitectura.jpeg)

## EC2
### EC2 LBs
El punto de entrada de las peticiones web externas se encuentran habilitadas mediante un Balanceador de Carga utilizando **EC2 LBs**. El mismo cuenta con configuraciones para la recepción de peticiones del protocolo HTTP y HTTPS.

Las peticiones HTTP son redirigidas de manera automática al puerto HTTPS.

Por detrás, los servicios se encuentran distribuidos en dos apartados diferenciados marcados por ramas distintas de desarrollo: Dev y Prototype.

Las peticiones dentro de la rama Prototype van dirigidas a:
*  **[nombreDelAmbiente]-drupal.siembradigital.com.py/** para solicitudes al Drupal.
*  **[nombreDelAmbiente].siembradigital.com.py/** para lo que refiere a consultas al backend.

### EC2
#### Web de Siembra Digital
La web de Siembra Digital o Frontend se encuentra alojada en una instancia de EC2 sobre Ubuntu Linux 20.04. La misma utiliza un Nginx para servir el contenido.

#### Rama Prototype
Para el despligue de los servicios de la rama Prototype, se han montado dos servidores Linux sobre EC2. Los mismos se encuentran basados en Ubuntu 20.04 LTS.

Las instancias se separan en dos funciones:
*  **Drupal:** Sobre esta instancia corre un Docker container sobre Docker Community Engine (Docker-CE). Este Drupal tiene como función, desplegar el contenido tanto de la Web de Siembra Digital como las funciones básicas de demo.
*  **Backend:** Esta instancia ejecuta el backend para los servicios necesarios de la demo que son desplegados mediante el Drupal.

#### Rama Dev
Para el despligue de los servicios de la rama Dev, también se han montado dos servidores Linux sobre EC2, ambos igualmente configurados con Ubuntu Linux 20.04 LTS.

Las instancias se separan en dos funciones:
- **Frontend:** Sobre este servidor se encuentra configurado un Nginx y éste se encarga de servir las peticiones relacionadas a la aplicación de Siembra Digital.
- **Backend:** En esta instancia se ejecutan los servicios de backend, correspondientes con un proceso Java que se encuentra funcionando sobre el puerto 8080.

## RDS
Para ambas ramas, se encuentra configurada además una instancia de **RDS** que ejecuta un motor de base de datos **MySQL**.
Sobre este motor se despliegan dos esquemas actualmente:
*  **sd_[nombreDelAmbiente]_prototype**
*  **sd_[nombreDelAmbiente]_dev**

## S3
Finalmente para servirse de contenido estático como imágenes, o para la carga de archivos a la plataforma se encuentra configurada actualmente una bucket de S3.

## Cognito
En cuestiones relacionadas a ABM de usuarios, así como el proceso de autenticación de los mismos se encuentra disponible el servicio de AWS Cognito.
Mediante el mismo se realiza la gestión de tokens de autenticación para la gestión de accesos de la plataforma de Siembra Digital.