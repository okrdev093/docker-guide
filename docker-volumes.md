# Docker Volumes
Los docker volumes son usados para persistencia de datos en docker, nos permiten compartir información entre el host y los contenedores y también entre contenedores, estos pueden ser archivos o carpetas.
En nuestro Host tenemos un file-system físico (donde persistimos nuestra información), la manera en que los volúmenes funcionan, es que nosotros conectamos nuestro  ruta de file-system físico  ( puede ser se una carpeta, un directorio etc) con una ruta de file-system del contenedor. En simples palabra un directorio o un archivo quedará montado en el file-system del contendor, de esta manera lo que sucede es cuando el contenedor escribe en el directorio del file-system que esta montado, la información se replica en nuestro file-system físico  y viceversa. Así que cuando el contenedor se reinicie automáticamente va a obtener la información que esta en el file-system del host que fue montado en el contenedor.
Esta es la manera en que poblamos la información cada vez que el contenedor se inicia.

#### Crear un volumen 

##### Host Volumes
La manera usual de crear un docker volume es mediante el comando `docker run` con la bandera `-v` donde indicamos el directorio del host (`host-directory`) y el directorio del contenedor (`container-directory`) , esta manera de definir un volumen la llamamos *Host Volumes* y su característica principal es que nosotros tenemos decisión de donde  cual va a ser el directorio en el host que vamos a compartir 

    $ docker run -v host-directory:container-directory

##### Anonymous volumes
En esta manera creamos un contenedor solo definiendo el el directorio del contenedor, es decir no especificamos que directorio del host va a estar vinculado, esto es controlado automáticamente por docker, si el directorio en el contenedor  es creado por docker en `/var/lib/mysql/data`  entonces tendremos una carpeta en nuestro host , por ejemplo en `/var/lib/docker/volumes/random-hash/_data` que sera montada automáticamente en el contenedor, este tipo de volumes son llamados  *anonymous volumes*  porque no tenemos referencia a este directorio 

    $ docker run -v var/lib/mysql/data

También podemos especificar un nombre  a la ruta del contenedor para referenciar el directorio usando un nombre sin conocer el path como hacíamos en host volumes

    $ docker run -v name:var/lib/mysql/data

esta tercera forma es quizás la mas usada en producción


### Crear un volumen entre contenedor y host ejemplo  

En este ejemplo se crea un volumen entre el container y nuestro host
1. Dentro de nuestro host creamos una carpeta *website* y dentro de ella guardamos un *index.html*, entonces el contenido de la carpeta `website` sera compartido dentro del contenedor en el directorio `usr/share/nginx/html`
2. iniciamos un contenedor y montamos nuestro directorio `website` 	 dentro del contenedor

        $ docker run --name website -v $(pwd):/usr/share/nginx/html:ro -p 8080:80 -d  nginx:latest

> nota: `$ (pwd)`  toma el directorio de trabajo actual, en nuestro caso estamos dentro del directorio website que creamos.
> :ro indica que es read-only.

3. Ahora al abrir nuestro navegador <ip>:8080  debemos ver la pagina  `index.html` que guardamos en nuestro host en el directorio website. Si hacemos algún cambio a nuestro archivo index.html entonces al refrescar el navegador se reflejara tal cambio sin tener que reiniciar el contenedor.

#### Guardando archivos en el contenedor para verlos en el host
Si guardamos un archivo dentro de nuestro contenedor en la carpeta que montamos `usr/share/nginx/html`, también sera reflejado en la carpeta `website` de nuestro host

usamos

    $ docker exec -it website bash
esta linea indica que queremos iniciar el modo interactivo `-it` el contenedor website y ademas queremos correr el comando `bash` el ejecutarlo veremos que ingresamos a una terminal del contenedor `root@5a9c11fdff34:/#`

navegamos a la carpeta `/usr/share/nginx/html` si hacemos `ls -al` veremos el archivo que creamos en nuestro directorio `website` en el host 

creamos el archivo about.html

    $ touch about.html
y obtendremos el siguiente mensaje 

    touch: cannot touch 'about.html': Read-only file system

Esto se da debido a que en primera instancia definimos nuestro volumen como solo lectura, si hacemos 

    docker run --name website -v $(pwd):/usr/share/nginx/html -p 8080:80 -d nginx:latest

> notar que eliminamos :ro

Ahora todo debería funcionar correctamente,y ahora debería aparecer nuestro archivo `about.html` en el carpeta `website` del host probarlo!

Esta es la manera en que compartimos archivos entre el host y el container usando volumes.


### Crear un volumenes entre contenedores

En este caso tenemos un contenedor A y un contenedor B y queremos compartir un volumen entre A y B, la manera en  que logramos esto es usando el comando `--volumes-from`

    $ docker run --name website-copy -d  --volumes-from  website  -p 8081:80 nginx:latest 

levantamos un nuevo contenedor con nombre `website-copy`
especificamos el volumen con --volumes-from y ponemos el nombre del otro contenedor `website` que levantamos en la sección anterior ,  definimos con `-p` el puerto 8081 para la conexión con el nuevo contenedor
 ahora si entramos en el  navegador `<ip>:8081` veremos una copia exacta del contenedor que esta levantado en el puerto `8080` , de esta manera estamos compartiendo datos entre 2 contenedores.
