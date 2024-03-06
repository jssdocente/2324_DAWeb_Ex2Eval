# Examen 2ª Evaluación - Despliegue de Aplicaciones Web

Este examen consta de dos ejercicios:

- Alumnos Grupo A: Deben realizar los ejercicios 1 y 2
- Alumnos Grupo B: Deben realizar los ejercicios 2

> 🔥 Existe 2 preguntas Extra que se puede realizar por cualquier grupo, y que suma 1,5 puntos cada una a la nota final (No más de 10 en total), y puede ser realizada por cualquier grupo.

<hr>


### Datos del alumno

- Nombre alumno: 
- Curso: 
- Fecha: 
- Evaluación: 


## Ejercicio 1  (5 Ptos)

Este ejercicio consiste en preparar para despliegue una Aplicación Web realizada en PHP. La aplicación es un Blog que se conecta a una base de datos MySQL.

Pero antes de poder desplegar, es necesario poder probar nuestra aplicación en local. Por ello los pasos a seguir son los siguientes:

- Crear una configuración de carpetas adecuada para el proyecto, que contenga el código fuente de la aplicación, así como los ficheros de configuración necesarios.
- Probar la aplicación en local con Docker. (A través de docker-compose)

<br>
<hr>

<details>
  <summary><p style="display:inline;font-size:14px">Previsualización página</p></summary>
  <br>
    <img src="res/01.AppWeb.working.gif">
</details>
<hr>

### Partes del ejercicio


#### Aplicación Web PHP

Se está desarrollando una aplicación web PHP, para un blog. Esta aplicación almacenará los datos en una base de datos MySQL.

El código fuente de la aplicación se encuentra disponible en el siguiente [repositorio de GitHub](https://github.com/mesinkasir/cuteblog-php)

La aplicación requiere de una BD MySQL, cuya configuración de acceso se configure en el archivo `widget/cutes.php`, **se requiere cambiar esta configuración, tanto `nombre-servidor`, `usuario`, `contraseña` y `nombre-base-datos`**, según los datos de acceso a la base de datos que se configuren.

#### Nginx-PHP

Para poder probar la aplicación en local, se va a utilizar una imagen de Docker que ya tiene configurado un servidor web Nginx y el servidor de aplicaciones PHP-FPM.

La confgiuración de Nginx por defecto (solo existe este servidor) es la que se indica en el siguiente fichero:

```nginx
# pass https on for Laravel isSecure/asset
map $http_x_forwarded_proto $fastcgi_param_https_variable {
    default '';
    https 'on';
}

server {
    listen       80; #ipv4
    server_name   _; #catch-all

    root   /var/www/html/public;

    if ($request_method = POST) {
        set $skip_cache 1;
    }    

    client_max_body_size 5M;

    location / {
        try_files $uri $uri/ /index.php?$query_string;

        # Client IP Handling for AWS ELB
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    location ~ \.php$ {
        root /var/www/html/public;
        
        fastcgi_cache dwchiang;
        fastcgi_cache_valid 200 204 60m;
        fastcgi_ignore_headers Cache-Control;
        fastcgi_no_cache $skip_cache $http_authorization $cookie_laravel_session;
        fastcgi_cache_lock on;
        fastcgi_cache_lock_timeout 10s;
        fastcgi_buffer_size 6144;

        add_header X-Proxy-Cache $upstream_cache_status;

        fastcgi_pass            127.0.0.1:9000;
        fastcgi_index           index.php;
        fastcgi_param           SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param           HTTPS $fastcgi_param_https_variable;
        fastcgi_read_timeout    900s;
        include                 fastcgi_params;
    }

    location ~* \.(jpg|jpeg|png|gif|ico|css|js|eot|ttf|woff|woff2)$ {
        expires max;
        add_header Cache-Control public;
        add_header Access-Control-Allow-Origin *;
        access_log off;
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

#### Docker

Para poder probar la aplicación en local, se va a utilizar un fichero docker-compose que levante dos contenedores, uno con la aplicación web y otro con la base de datos MySQL.

#### Docker-compose

Se dispone de un fichero `docker-compose.yml` base que hay que configurar con la configuración específica para este caso.

1. Dispone de 2 servicios, uno para la aplicación web y otro para la base de datos MySQL.
   1. Servcio `www`:
      - Se basa en la imagen `jssdocente/nginx-php-fpm:8.2`, que ya tiene configurado un servidor web Nginx y el servidor de aplicaciones PHP-FPM.
      - El servicio web esucha por el puerto 80, y se debe acceder a través de `http://localhost`.
      - El servicio web debe montar un directorio `app` que contiene el código fuente de la Web, en la ruta adecuada según se indica en la configuración de Nginx (indicada anteriormente).
      - La configuración de Nginx indicada se debe montar también para que se aplique, en la ubicación/nombre-archivo adecuada.	

   2. Servicio `db`:
      -  Se basa en la imagen `mysql:8.0`.
      -  El puerto interno de la base de datos es el 3306, pero para evitar conflictos con una posible base de datos MySQL que se esté ejecutando en el equipo, se debe utilizar de forma externa el puerto `3390`.
      -  El nombre de la BD, así como el usuario y contraseña, se indican en el fichero `docker-compose.yml` como variables de entorno.
      -  El nombre del servidor de BD es el nombre del servicio `db`.
      -  La variables de entorno `MYSQL_ALLOW_EMPTY_PASSWORD`:`yes` indica que el usuario `root` no tiene contraseña. (*Para conocer el significada de cada variable de entorno, consulta la documentación de la imagen de MySQL en DockerHub*)

      - Para almacenar los datos de la base de datos, se ha creado un volumen `ex2eval_dbdata`. Se debe mapear el volumen a la ruta `/var/lib/mysql` del contenedor.
      - Para inicializar la base de datos, se debe montar un directorio `init` que contiene los scripts de inicialización de la base de datos, en la ruta `/docker-entrypoint-initdb.d` del contenedor.
      Los datos de inicialización deben estar ubicados en el directorio `conf/db/init/` de la estructura de carpetas del proyecto. El script de inicialización se debe llamar `init.sql`. (Este script está disponible en la carpeta `database` del código fuente de la App)

```yaml
version: "3.8"
name: ex2eval
services:
    www:
        image: jssdocente/nginx-php-fpm:8.2
        container_name: 2ev-nginx
        # ... completa la configuración del servicio www
        
    db:
        image: mysql:8.0
        container_name: 2ev-mysql-8.0
         # ... completa la configuración del servicio www

        command: --default-authentication-plugin=mysql_native_password
        environment:
            MYSQL_DATABASE: cuteblog
            MYSQL_USER: cuteblog
            MYSQL_PASSWORD: cuteblog
            MYSQL_ALLOW_EMPTY_PASSWORD: "yes" 
        volumes:
            - ./conf/db/init:/docker-entrypoint-initdb.d

volumes:
    ex2eval_dbdata:
       driver: local
```

### Pasos de la tarea

- [X] 1.1 Crear la estructura de carpetas para probar la aplicación en local
- [X] 1.2.1a Crear el fichero docker-compose y explicación del mismo, para probar la aplicación en local. 
- [X] 1.2.1b Conexión a la BD desde MySQL Workbench
- [X] 1.2.2 Aplicación Web funcionando
- [X] 1.2.3 Modificación configuración Nginx solucionar problema 403 Forbidden


### Partes a entregar

#### 1.1 Estructura de carpetas

Se debe crear una estructura de carpetas adecuada para el proyecto, que contenga el código fuente de la aplicación, así como los ficheros de configuración necesarios, tanto para probar probar la configuración, como para empaquetar para el despliegue.

> 📄 Explica cada una de las carpetas y archivos, indicando que funcionalidad tiene, qué ficheros se van a alojar en ella, explicando para cada uno de ellos su función/utilidad.


> 🧲 Captura de la estructura de carpetas, donde se visualize claramente el nombre de las carpetas y archivos.


#### 1.2.1 Entrega de la configuración de docker-compose

Comenta las líneas del fichero `docker-compose.yml` que has incluiudos, indicando qué hace cada línea, a través de un comentario en el propio fichero.

> 🧲 Incluye aquí una captura de pantalla del fichero docker-compose.yml con todas las partes necesarias rellenas.



> 🧲 Incluye un GIF con la ejecución del comando `docker compose up`



> 🧲 Incluye un GIF donde se visulize la conexión desde *MySQL Workbench* a la BD. Muestra la estructura de la BD, y los datos de la tabla `blog`.



#### 1.2.2 Entrega Aplicación Web funcionando

> 🧲 Incluye un GIF con la ejecución de la aplicación web en local. Accede a la misma a través del Navegador, y navega a algunas `entradas de blog`.




#### 1.2.3 Modifica configuración Nginx

Si accedes a `localhost` verás que obtienes una página `403 Forbidden`. Corrige la configuración de Nginx para que se pueda acceder a la aplicación sin especificar una página. (Hazlo a través del fichero de configuración de Nginx que se monta en el contenedor).

> 🧲 Captura pantalla configuración Nginx donde se resalte el cambio realizado



> 📄 Explica qué has configuración has incluido y explica el motivo.



> 🧲 Incluye un GIF donde se visualize que se puede acceder por `localhost`.

<br>
<hr>
<br>

## Ejercicio 2  (5 ptos)

En base a la tarea [TE6.2](https://github.com/jssfpciclos/DAW_daweb/blob/main/UT6/TE6.2/te62_tarea.md), se pide realizar el siguiente ejercicio.

### Página Web Estática

Codigo fuente: [relax.website.zip](https://drive.google.com/file/d/1OPi2iQAQ-M8kFhYW5IvJi4fKC0T7i4Oi/view?usp=sharing)

Condiciones:

1. Dominio: relax.local / www.relax.local
2. Escuchar por el puerto 80
3. Alojar la web en la carpeta /var/www/html/relax.local
4. La página índice principal debe ser index.html
5. Configura para que si se accede a `relax.local/images` se pueda lista el contenido de la carpeta `images`
6. Configura para los errores 500 502 503 504 se muestre una página llamada 50x.html (si no existe crealá)
7. Crear una página 404.hml personalizada al producirse un error 404.
8. Esta página 404.html,y la página 50x.html no pueden ser accedidas desde el exterior, es decir, si se accede a `relax.local/404.html` o `relax.local/50x.html` se debe mostrar un error 403 (forbidden)
9. Configura los logs de acceso y error para que se guarden en la carpeta `/var/log/nginx/relax` con el nombre `relax.local.access.log` y `relax.local.error.log` respectivamente.
10. Utiliza la imagen de nginx:1.25.3-alpine

> 🔥 Para aplicar esta configuración es requerido crear una configuración personalizada para este dominio, que se debe alojar en la carpeta adecuada según la configuración del fichero /etc/nginx/nginx.conf.

Imagen de Docker: nombreusuario/relax:1.0

<hr>
<details>
  <summary><p style="display:inline;font-size:14px">Previsualización página</p></summary>
    <br>
    <img src="res/02.AppWeb.working.gif">
</details>

<hr>

### Pasos de la tarea

- [X] 1. Crear la estructura de carpetas para probar la aplicación en local / empaquetar para el despliegue.
- [X] 2. Crear el fichero docker-compose para probar la aplicación en local. 
- [X] 3. Levantar los contenedores y probar la aplicación en local.
- [X] 4. Una vez todo OK, eliminar los contenedores, a través de comando.
- [X] 5. Crear imagen docker a partir de DockerFile 
- [X] 6. Crear contenedor en base al DockerFile y probar la apliación en local.
- [X] 7. Comprobación de funcionalidad (todo OK) a través de dominio `relax.local` y el contenedor en base a la imagen creada.
- [X] 8. Subir contenedor en base a DockerHub

<hr>

#### Documentación de la tarea

> 📄 0. Indica con un comentario dentro del fichero de configuración , para qué se usa cada directiva.
> 🧲 Adjunta captura de pantalla donde se visualize la configuración del fichero de configuración personaliado para alojar el dominio realizado.<br>


> 📄 1. Explica cada una de las carpetas y archivos, indicando que funcionalidad tiene, qué ficheros se van a alojar en ella, explicando para cada uno de ellos su función/utilidad.


> 🧲 1. Adjunta captura de pantalla donde se visualize la estrucutra de carpetas


> 🧲 2. Adjunta captura del fichero docker-compose que incluya los comentarios en línea de qué hace cada línea.


> 🧲 3. Adjunta GIF levantar escenario con docker-compose.


> 🧲 3. Adjunta GIF docker-desktop con el escenario creado y los contenedores creados y OKs (en verde).


> 🧲 4. Adjunta GIF/Imagen donde se eliminen el escenario y todos los recursos asociados.


> 🧲 5. Adjunta captura del fichero DockerFile creado. Pon un comentario en las líneas principales explicando su funcionalidad


> 🧲 5. Adjunta GIF con la ejecución de la imagen a través del DockerFile.  


> 🧲 6. Adjunta GIF con la creación del contenedor en base a la imagen creada.


> 🧲 7.1 Adjunta GIF accediendo a la página web, y se visualize correctamente a través de dominio `relax.local`. También accede a `relax.local/noexiste.html` para comprobar que se muestra la página 404.<br>


> 🧲 7.2 Adjunta GIF visualize error al acceder desde fuera a `relax.local/404.html` o `relax.local/50x.html`


> 🧲 8.1 Adjunta GIF con la subida de la imagen de docker a DockerHub


> 🧲 8.2 Adjunta captura desde DockerHub, donde se visualize tu repositorio y la imagen subida al mismo


<br>
<hr>
<br>

## Ejercicio Extra 1  (Extra 1,5 ptos)

En base al ejercicio 2, la empresa `relaxx` propietaria de la web `relax.local`, dispone del dominio `buendescanso.local`, pero quiere que los usuarios que accedan a `buendescanso.com` sean redirigidos a `relax.local`, ya que la página antigua `buendescanso.local` ya no se utiliza.

Para la página `buendescanso.local`, crea una página `index.html` muy básica que muestre un cartel de `UN BUEN DESCANSO NO TIENE PRECIO`.

Para ello, crea un fichero de configuración para el dominio `buendescanso.local`, dentro del docker-compose, que aplique esta configuración.

> **Auto-aprendizaje**<br>
> De este tema no se ha realizado ninguna práctica, pero la documentación está disponible en los [apuntes](https://github.com/jssfpciclos/DAW_daweb/blob/main/UT4/README.md#redirecciones) y también existe mucha documentación en la web.


### Pasos de la tarea

- [X] 0. Breve explicación de cómo conseguir esto.
- [X] 1. Crea una configuración para este nuevo dominio.
- [X] 2. Prueba la nueva configuración para que al acceder a ella, se redirija a `relax.local`.


#### Documentación de la tarea

> 📄 0. Explica brevemente qué pasos debes dar para conseguir esto


> 🧲 1. Adjunta captura de pantalla donde se visualize la configuración del fichero configuración dominio `buendescanso.local`. Explica las líneas con comentarios internos en el fichero.



> 🧲 2. Adjunta GIF accediendo a la página web `buendescanso.local` y como se redirige al nuevo dominio.<br>

<br>
<hr>
<br>

## Ejercicio Extra 2  (Extra 1,5 ptos)

En base al ejercicio extra 1, se requiere configurar un certificado `autofirmado` para que Google no penaliza al sitio web de esta empresa, ya que actualmente solo se puede acceder por `http`. 

Es decir que si un usuario accede a `http://relax.local` sea redirigido a `https://relax.local`.

Para conseguir esto, es necesario que el bloque de configuración para el dominio `relax.local` incluya la configuración necesaria para que se pueda acceder por `https`, a través de un certificado autofirmado.

También se requiere que las personas que accedan por `http://relax.local` sean redirigidas a `https://relax.local`, por lo que será necesario crear una configuración para HTTP y otra para HTTPS.

➕ *Resumiendo:*

- Habría que crear 2 configuraciones, una para HTTPS y otra para HTTP.
  - La configuración original habrá que cambiar para que escuche por el puerto 443, en lugar del 80.
  - Crear otra nueva, para que escuche por el puerto 80 y deberá redirigir a `https://relax.local`.


> **Auto-aprendizaje**<br>
> De este tema no se ha realizado ninguna práctica, pero la documentación está disponible en los [apuntes](https://github.com/jssfpciclos/DAW_daweb/blob/main/UT4/README.md#certificados-autofirmados) y también existe mucha documentación en la web.


> 🔥 Recuerda, que debes redirigir el puerto 443, tanto a nivel del contenedor, como a nivel de DNS con Awesome Manager.
>    - localhost:443 https://www.relax.local https://relax.local
>    - localhost www.relax.local relax.local


### Pasos de la tarea

- [X] 0. Breve explicación de cómo conseguir esto.
- [X] 1. Crea una configuración para este nuevo dominio.
- [X] 2. Prueba la nueva configuración mostrando su funcionamiento.


#### Documentación de la tarea

> 📄 0. Explica brevemente qué pasos debes dar para conseguir esto


> 🧲 1. Adjunta captura de pantalla donde se visualize la configuración de los 2 ficheros de configuración para el dominio `relax.local`, uno para HTTP y otro para HTTPS.



> 🧲 2. Adjunta GIF accediendo a la página web `http://relax.local` sean redirigidas a `https://relax.local`.
