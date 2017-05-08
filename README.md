# Distribuidos-Parcial2

# Balanceador de cargas
Balanceador de cargas automatizado 

Daniel Steven Ocampo

A00053980

https://github.com/DanielOcampo23/Distribuidos-Parcial2

# Objetivos

-Realizar de forma autónoma el aprovisionamiento automático de infraestructura
-Diagnosticar y ejecutar de forma autónoma las acciones necesarias para lograr infraestructuras estables
-Integrar servicios ejecutandose en nodos distintos

# Descripción

Deberá realizar el aprovisionamiento de un ambiente compuesto por los siguientes elementos: un servidor encargado de realizar balanceo de carga, tres servidores web con páginas estáticas. Se debe probar el funcionamiento del balanceador realizando peticiones y mostrando servidores distintos atendiendo las peticiones.

![Github Logo0](Imagenes/01_diagrama_despliegue.png)

#Herramientas
 -Docker
 -Docker-compose
 -Nginx
 -html (para probar las páginas web)
 
#Procedimiento

Primero configuramos el Dockerfile del nginx para su respectiva instalación, el cual va a ser la herramienta encargada de hacer el balaceo de las web, por eso tenemos que asegurarnos de su funcionamiento antes de configurar las web:

```
FROM nginx

# Remove the default Nginx configuration file
RUN rm /etc/nginx/conf.d/default.conf && rm -r /etc/nginx/conf.d

# Copy a configuration file from the current directory
ADD nginx.conf /etc/nginx/nginx.conf

# Append "daemon off;" to the beginning of the configuration
RUN echo "daemon off;" >> /etc/nginx/nginx.conf

# Set the default command to execute
# when creating a new container
CMD service nginx start
                                 
```

En este Dockerfile se hace los respectivos pasos para poder garantizar su instalación, uno de los pasos importantes es remover el archivo de configuración que viene por defecto del ngnix y replazarlo por el nuevo que mostraré a continuación, el cual lo llamaremos nginx.conf:

```
http {
    sendfile on;
    upstream app_servers {
        server app1:80;
        server app2:80;
        server app3:80;
    } 
    server {
        listen 80;
        location / {
            proxy_pass         http://app_servers;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
        }
    }
}
```

En este archivo configuramos cuales son las web que el ngnix balanceará y los puertos en los que va a escuchar el nginx.

#Docker-compose.yml

Después de configurar el dockerfile del nginx, pasamos a configurar el docker-compose.yml el cual es el encargado de montar toda las infraestructura mostrada en el diagrama de deployment anteriormente, este docker-compose debería quedar así con sus respectivas web:

```
version: '2'
services:
  web1:
    build: ./web1
    expose: 
      - "5000"
  web2:
    build: ./web2
    expose:
      - "5000"
  web3:
    build: ./web3
    expose:
      - "5000" 
  proxy:
    build: ./nginx 
    ports: 
      - "8080:80"
    links:
      - web1
      - web2
      - web3
```           

En este docker-compose configuramos los servicios, osea las web que va a balancear el nginx con su respectivo puerto que expone cada web, después se configura el proxy (nginx) en el cual se le indica la carpeta en donde esta su respectivo Dockerfile y nginx.config que configuramos en anteriormente, después de esto se describe los puertos del nginx y después se lista las web que balaceará 

#Configuración de las carpetas con sus archivos de las web

Después de mencionar las carpetas de las web en el docker-compose.yml, procedemos a crearlas y configurarlas con sus respectivos archivos:
  -Dockerfile
  -web1.html

Para el Dockerfile lo configuraremos de esta manera:

``` 
FROM httpd

#Ubicacion de la web1
ADD web1.html /usr/local/apache2/htdocs/web1.html
```                       
               
En este archivo simplemente mencionamos donde se encuentra el html de la web1

Para el web1.hmtl

Simplemente adentro de este archivo escribimos el nombre de la web para diferenciarlo de las demas algo como:
```
web1
``` 
Después realizamos lo mismo con las demas web (web2 y web3) para poder tener las tres web requeridas para el balaceo

#Ejecución

Finalmente después de haber configurado correctamente todos los archivos antes mencionados, procedemos a ejecutar el docker-compose el cual es el encargado de ejecutar toda la infraestructura como lo mencione anteriormente, para esto ejecutamos el siguiente comando:

```
docker-compose up
```
    
#Problematica

Durante la realización de este ejecicio se presentaron una serie de problemas que con el tiempo se fueron solucionando, algunas fueron:
-Problema con los volumenes de Docker: primero tenia pensado abordar el problema con volumenes, pero después de presentar una serie de problemas con esto, investigé si de verdad eran necesario estos volumenes, y como era de esperarse no fue necesario utilizarlos, por lo que la solución de este problema no aborda volumenes.
