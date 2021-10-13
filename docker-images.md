## Docker Image Layers
Las imagenes de docker estan formadas de una lista de solo lectura de capas que representan las diferencias del sistema de archivos. Las capas imagenes (image layers) estan apiladas formando una base para el filesystem del contenedor. Cada imagen consiste en multiples capas y cada capa es otra imagen, una  imagen que se encuentra debajo de otra se reconoce como la imagen padre. La imagen que esta en la base del stack es llamada imagen base (base image). Docker trae las imagenes capa por capa

para ver las capas  que conforman una imagen podemos utilizar el comando `history`

    $ docker history busybox:1.24


#### Construyendo Docker Images
 tenemos 2 maneras de construir una imagen de docker 

1. Commit de los cambios hechos en un contenedor de docker 
2. Escribir un dockerfile.

##### Escribiendo una imagen  haciendo un commit 
para escribir una imagenen haciendo un commit seguimos los siguientes pasos
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



##### Escribiendo una imagen  mediante un dockerfile

 

