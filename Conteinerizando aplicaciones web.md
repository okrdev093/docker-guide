# Creando web apps containerized

En esta seccion veremos como contenerizar una app web

Tenemos el siguiente Dockerfile para nuestra web app

     FROM python:3.5
     RUN pip install Flask=0.11.1
     RUN useradd -ms /bin/bash admin
     USER admin
     WORKDIR /app
     COPY app /app
     CMD ["python", "app.py"]


 -  Primero tenemos la instruccion `FROM` que define la imagen base de python 3.5
 - Luego instalamos Flask ( web framework minimalista de python) .
 - Luego con `useradd` creamos un usuario agreagando la bandera `-ms` para que se cree un directorio home para el usuario y con las lineas `/bin/bash` estamos seteando el shell por default como `bash` y por ultimo ponemos el nombre de usuario como `admin`
 - Despues con `USER admin` aseguramos que la app va estar corriendo logueada con el usuario admin, en general esto es algo que siempre debemos hacer en nuestro Dockerfile, correr nuestros procesos como usuario no privilegiado dentro de nuestro contenedor. *De otra manera el proceso va a correr como root dentro del contenedor, esto trae algunos problemas de seguridad, se dice que si un atacante toma control del contenedor tendria acceso root a la maquina host porque los UID's son iguales.
 - En `WORKDIR` esta instruccion setea el directorio de trabajo principal en nuestro caso `/app` para cualquier  instruccion RUN, CMD, ENTRYPOINT, COPY y ADD que sean escritas a continuacion en el  Dockerfile
 - Luego con `COPY` Copiamos el directorio app dentro del contenedor
 - Por ultimo con `CMD` corremos nuestra aplicacion Flask

ahora usamos docker bulild para correr nuestra imagen

    $docker build -t dockerapp:v0.1 .


Una vez terminado el proceso corremos el contenedor, usamos `-d` para correr el contenedor en el background y `-p` para exponer el puerto por defecto el puerto de flask server es `5000` dentro del contenedor , mapeamos el puerto `5000` del contenedor al puerto `5000` del contenedor.

    $ docker run -d -p 5000:5000 <image_id>  

si vamos al navegador de la maquina host y tipeamos 

    localhost:5000
debemos ver el mensaje "hello mundo" desde la app de flask

##### Docker Exec
`docker exec` nos permite correr un comando dentro de un contenedor que este corriendo, la bandera -it corre el comando en modo interactivo

    $ docker exec -it <container_id>  <comando>
    $ docker exec -it b8a02342asc  bash

 al correr este comando ingresaremos a la consola dentro del contedor y nos indicara que estamos usando el  user `admin` que definimos en nuestro `Dockefile` 

podemos entrar a la carpeta home del usuario admin

    cd /home/admin

y si corremos `$ ps axu` podemos ver todos los procesos corriendo dentro del contenedor para comprobar que nuestra python app este corriendo bajo el mando del user admin.
