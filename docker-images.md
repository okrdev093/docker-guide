## Docker Image Layers
Las imágenes de docker están formadas de una lista de solo lectura de capas que representan las diferencias del sistema de archivos. Las capas imágenes (image layers) están apiladas formando una base para el filesystem del contenedor. Cada imagen consiste en múltiples capas y cada capa es otra imagen, una  imagen que se encuentra debajo de otra se reconoce como la imagen padre. La imagen que esta en la base del stack es llamada imagen base (base image). Docker trae las imágenes capa por capa

para ver las capas  que conforman una imagen podemos utilizar el comando `history`

    $ docker history busybox:1.24


#### Construyendo Docker Images
 tenemos 2 maneras de construir una imagen de docker 

1. Commit de los cambios hechos en un contenedor de docker 
2. Escribir un dockerfile.

##### Escribiendo una imagen  haciendo un commit 
Para escribir una imagen  haciendo un commit seguimos los siguientes pasos

1. Creamos un contenedor a partir de una imagen base.
2. Instalamos Git Package en el contenedor.
3. Comiteamos los cambios hechos en el contenedor.
  
 para este ejemplo usamos la imagen de debian (1)

    $docker run -it debian:jessie

instalamos git (2), corriendo el comando dentro del contenedor

    $ apt-get update && apt-get install -y git

salimos del contenedor y hacemos un commit de nuestra imagen usando el comando `docker commit`, lo que hace este comando es guardar los cambios que hicimos dentro del contenedor en una nueva imagen

    $ docker commit <container_ID> <repository_name:tag>
para nuestro caso usamos

     $ docker commit 0daef331231as osskr/debian:1.00
     
estos nos retornara el ID de una nueva imagen   

si hacemos docker images podemos ver nuestra imagen creada

    $ docker images
si observamos el tamaño veremos que es mucho mas grande que la imagen oficial contiene debian porque nosotros agregamos git a nuestro filesystem.

ahora podemos levantar un contenedor basado en esta nueva imagen 

    $ docker run -it osskr/debian:1.00
y si corremos el comando git una vez abierto veremos que se encuentra instalado correctamente.

#### Eliminando una imagen
Para eliminar las imagenes que tenemos en nuestro registro usamos el siguiente comando

    $  docker rmi <your-image-id>

### Dockerfile
En resumen un [dockerfile](https://docs.docker.com/engine/reference/builder/) contiene una lista de pasos de como crear una imagen.

##### Escribiendo una imagen  mediante un dockerfile
En esta sección veremos como construir una imagen a partir de un `docker-file`.  Es es un documento de texto que contiene todas la instrucciones necesarias para ensamblar una imagen, una **instrucción** puede ser por ejemplo instalar un programa, agregar algo de código fuente, o especificar comando que se van a correr cuando el contenedor inicie.
Docker construirá la imagen automáticamente, leyendo las instrucciones que se encuentran en el `dockerfile`, cada instrucción creará una nueva image-layer a la imagen. En resumen las instrucciones especifican que hacer cuando se construye la imagen.

creamos un dockerfile, estos no tienen extensión y deben ser nombrados `Dockerfile` con 'D' en mayúsculas

    $ touch Dockerfile

estructura de un **DockerFile**

    FROM debian:jessie
    RUN apt-get update
    RUN apt-get install  -y git
    RUN apt-get install  -y vim


Docker lee las instrucciones en orden. La orden `FROM` siempre debe ser la primera y en ella especificamos la **imagen base** a partir de la cual estamos construyendo nuestra nueva imagen, en nuestro caso usamos debian como en la sección anterior.

La instrucción RUN especifica un comando que se ejecutará, puede ser cualquier comando que ejecuta en una terminal de Linux

#### Docker build 

Una vez **terminado** nuestro archivo `Dockerfile` usamos el comando `docker build` para construir nuestra imagen usando las instrucciones que le damos en nuestro Dockerfile

    $ docker build -t osskr/debian .
	# con nombre de tag
	$ docker build --tag <name>:v.01 osskr/debian .

> la opción **-t** nos permite crear un tag para la imagen que estamos
> construyendo
> también podemos usar --tag
> adicionalmente necesitamos un path que especifica el build context, es donde esta el DockerFile, si estamos en esa carpeta podemos usar un punto .
> Por defecto docker buscará este archivo donde lanzamos el build command
> Con el .  definimos donde esta el Dockerfile




Cuando el build comienza, docker client empaquetara todos los archivos en el build context en un tarball y luego transferirá el tarball al daemon.

cuando corramos el comando tendremos un output donde veremos como docker va construyendo la imagen.

Podemos ver los STEPS de cada comando que especificamos en nuestro `Dockerfile`

como no especificamos el tag (aunque si pusimos la bandera `-t`) cuando corremos `$ docker images` veremos que la versión de la imagen que creamos sera `latest`

una vez terminado el build podemos checkear con `$ docker images` para comprobar que se generó la imagen 

y podemos correr el contenedor con la imagen generada haciendo por ejemplo:
   

     $ docker run  --name <nombre-del-contenedor> -p 8080:80 -d <imagen-q-generamos>:<tag>

     $ docker run  --name website -p 8080:80 -d website:latest




   
#### Dockerfile en profundidad
Veremos en profundidad las instrucciones que podemos dar a nuestros Dockerfiles y la manera mas eficiente de escribir nuestras imágenes de docker,

##### Encadenando instrucciones RUN

Cada comando `RUN` se ejecutará en la capa escribible que este mas arriba del contenedor, luego hará un commit del contenedor como una nueva imagen.
Esa nueva imagen generada se utilizará en el próximo paso definido en el `Dockerfile`. Así que cada instrucción `RUN`  creará una nueva capa de imagen.
Es recomendado encadenar las instrucciones `RUN` en el `Dockerfile` para reducir el numero de imagenes de capa que se crean.

podemos hacer esto escribiendo en el  Dockerfile de la siguiente manera
 
 Antes teníamos

    FROM debian:jessie
    RUN apt-get update
    RUN apt-get install  -y git
    RUN apt-get install  -y vim

Lo cambiamos por 

    FROM debian:jessie
    RUN apt-get update && apt-get install -y \ 
    git\
    vim
ahora tenemos todas la instrucciones en un solo `RUN`
y como vemos al correr el comando `docker build` se reduce la cantidad de STEPS en este solo tenemos 2 

##### Ordenar argumentos multi línea alfanuméricamente

si por ejemplo queremos instalar python lo especificamos de la siguiente manera en el `Dockerfile` para tener todo ordenado alfanuméricamente

    FROM debian:jessie
    RUN apt-get update && apt-get install -y \ 
    git\
    python\
    vim

##### CMD Instructions 
Las `CMD instructions` especifican que comandos queremos correr cuando el contenedor se inicia, si no especificamos la `CMD instruction` en el `Dockerfile`, docker usara el comando por defecto definido en la imagen base. En nuestro caso de estos ejemplos el comando por defecto seria `bash` que esta definido en `debian:jessie`. Los comandos cmd no corren cuando estamos construyendo la imagen con `docker build` sino  recién cuando el contenedor se inicia.
Podemos especificar los comando en la forma `exec` ( la cual es recomendable) o en la forma `shell`

Lo hacemos de la siguiente manera en nuestro Dockerfile

    FROM debian:jessie
    RUN apt-get update && apt-get install -y \ 
    git\
    python\
    vim
    CMD ["echo", "hello worldo"]
         
   Al reconstruir la imagen con `docker build` y luego iniciar le contenedor `con docker run` veremos que se despliega el mensaje `"hello worldo"`

podemos sobreescribir la instrucción `CMD` en tiempo de ejecución si al momento de ejecutar el comando `docker run` le enviamos la instrucción que queremos que se ejecute

    $ docker run <container_id> echo "hola en tiempo real"

ahora veremos el mensaje "hola en tiempo real" cuando se ejecuta el contenedor

##### Docker Caché
Como vimos anteriormente cada vez que docker corre una instrucción del `Dockerfile` este crea una nueva image-layer, la siguiente vez que corramos el comando `docker build`, si la instrucción no cambia docker sabrá que la image-layer para esa instrucción existe y en vez de crearla de nuevo usará la existe almacenada en el cache, al usar `docker build`si docker toma la imagen del cache veremos un mensaje como el siguiente 

    -->  Using cache
    --> <4b4e4f4g4e124a>
Esto nos ayuda a que nuestros builds se ejecuten mas rápido, de todas maneras si usamos en exceso el docker cache nos puede traer algunos problemas 

por ejemplo si tenemos el siguiente dockerfile 

      FROM debian:jessie
      RUN apt-get update
      RUN  apt-get install -y git
y posteriormente deseamos agregar el programa curl entonces modificaremos nuestro `Dockerfile` de la siguiente manera

    FROM debian:jessie
    RUN apt-get update
    RUN  apt-get install -y git curl

Esto haría que docker tome las 2 primeras instrucciones desde el cache porque no cambiaron (solo cambiamos la tercera) , lo cual podría llevarnos a obtener versiones no actualizadas de git y curl

la solución es encadenar las instrucciones como vimos anteriormente

    FROM debian:jessie
    RUN apt-get update && apt-get instal l -y git curl

también podemos especificar a docker que invalide el cache  usando la bandera `--no-cache=true` a la hora de construir la imagen.

     $ docker build -t osskr/debian . --no-cache=true

#####  Instrucción COPY

La instrucción `COPY`  como su nombre lo dice copia nuevos archivos o directorios desde el `build context` y los agrega al file-system del contenedor.

Agregamos un archivo de texto cualquiera a nuestro directorio donde se encuentra el Dockerfile

    $ touch archivo-prueba.txt

Modificamos el Dockerfile de la siguiente manera

    FROM debian:jessie
    RUN apt-get update && apt-get install -y \ 
    git\
    vim
    COPY archivo-prueba.txt /src/archivo-prueba.txt

Reconstruimos la imagen y corremos el contenedor

    $ docker build -t osskr/debian .
    $ docker run -it <container_id>

se nos abre el `bash` del contenedor y si buscamos en la carpeta `src` veremos el archivo que enviamos `archivo-prueba.txt`

#####  Instrucción ADD
Es muy similar a `COPY` con la diferencia es que `ADD` es mas potente, esta por ejemplo nos permite descargar un archivo desde internet y copiarlo al contenedor, `ADD` también tiene el poder de desempaquetar archivos comprimidos automáticamente, si el archivo tiene una extensión conocida de archivo comprimido (.rar, .tar, .zip, etc) entonces se descomprimirá en una ruta especificada del file-system del contenedor

Generalmente usaremos `COPY` porque es mas transparente aunque solo soporte el copiado básico de archivos, y en caso de que se requiera mas capacidad usamos el `ADD`

#### Docker ignore File
esto funciona muy parecido a .gitignore, si queremos evitar agregar un directorio o un archivo cuando estamos agregando a nuestro working directory con `ADD` en el `Dockerfile` debemos crear el archivo `.dockerignore` y en el especificar los archivos o directorios que queremos ignorar, en este caso haremos un ejemplo con el directorio `node_modules`

    .dockerignore
    		node_modules
    		Dockerfile
    		.git


en este caso estamos ignorando el directorio `node_modules`, el archivo `DockerFile`, y la carpeta `.git` 

#### Dockerizando una API de node- express
Una vez creada nuestra api como es usual con node y express vamos a crear  dockerizarla así que lo primero que hacemos es crear un Dockerfile para luego crear una imagen

Creamos el dockerfile dentro de nuestro directorio donde esta la app de node y agregamos lo siguiente.
	
    FROM  node:latest
    WORKDIR /app
    ADD . .
    RUN npm install
    CMD node index.js
   
 

   Construimos la imagen con el comando docker build

    $ docker build -t user-service:latest .
y por ultimo levantamos el contenedor 

    $ docker run  --name user-api -p 5500:5500 -d user-service:latest
y ahora si ingresamos al navegador en <ip-server>:5500 veremos nuestra api respondiendo

//falta completar un poco esta seccion

#### Subir imagenes a DockerHub

Primero necesitamos una cuenta de DockerHub.
Para enviar la imagen que construimos al repositorio correcto en DockerHub , tenemos que asociar la imagen con nuestro usuario de DockerHub (docker ID), esto lo hacemos renombrando la imagen como:

*`docker hub ID / nombre_del_repositorio`*

para renombrar la imagen usamos docker tag

    $  docker images -> para ver nuestras imagenes
    $ docker tag <image_name>   <repository_name>
     ej:
     $ docker tag 3h423hf1ads  osskr/debian:1.01
     
y también especificamos un tag para esta imagen, si no lo especificamos docker usara **latest** pero tratemos de no usarlo a menos que no haya otra opción,se hace mas difícil de mantener

para hacer el push de nuestra imagen a DockerHub , usamos el comando

     $ docker login username=<nombre_usuario>
luego un prompt nos pedirá nuestro **password** y si sale todo bien veremos el mensaje *Login Succed*

por ultimo una vez logueado usamos el comando docker push

    $ docker push osskr/debian:1.01

este comando enviara la imagen a DockerHub que luego podremos ver en el sitio en la sección de repositorios.
