
# Docker

### Introducción a las tecnologías de virtualización
Docker es una implementación de las tecnologías de virtualización basadas en contenedores, nos ayuda a correr aplicaciones en un ambiente aislado, es similar a una maquina virtual. Nos facilita muchísimo el trabajo de desplegar una aplicación.

#### Mundo Pre- Virtualización
Antes de que exista la virtualización utilizabamos servidores físicos, en la base teniamos los servidores, luego le instalabamos  el sistema operativo y corriamos nuestra aplicación en el servidor. Cada Servidor corria una aplicación. Esto llevaba a grandes costos debido a los precios de los servidores y muchas  veces solo se aprovechaba una fracción de la capacidad de procesamiento del servidor. Otro gran problema era el tiempo de despliegue de una aplicación y lo difícil de migrar de un servidor a otro en caso de que fuera necesario.

#### Virtualizacion basada en  Hypervisor
En esta tecnologia tambien tenemos el servidor físico en la base y luego le instalamos nuestro sistema operativo y encima de nuestro sistema operativo instalamos una capa hipervisor, la cual nos permite instalar múltiples maquinas virtuales en una sola maquina, cada una de estas maquinas puede correr un sistema operativo distinto y así podemos correr distintas aplicaciones en una sola maquina. Los provedores mas populares de estas tecnologia basada en hipervisor son VMware, VirtualBox. En los inicios de esta tecnologías los usuarios desplegaban sus maquinas virtuales en sus propios servidores físicos pero hoy en día hay una migración a utilizar servicios de virtualización en la nube utilizando servicios como AWS, o  Microsoft Azure. Donde no necesitamos comprar nuestro servidores físicos.
Hay múltiples beneficios que esta tecnología nos ofrece (cloud) , por ejemplo es mas eficiente en términos de costo ya que cada maquina virtual esta divida en múltiples VMs( virtual machines)   y cada una solo utiliza su propia CPU memoria y recursos de almacenamiento, solo pagamos por lo que utilizamos. También es fácil de escalar ya que solo cliqueamos y desplegamos mas VMs en la nube.
Dentro de las desventajas encontramos que cada maquina virtual necesita su sistema operativo, lo cual lo hace poco eficiente. La portabilidad no puede ser asegurada.

#### Virtualización basada en contenedores
En la Virtualización basada en contenedores, en la base de este stack tenemos el servidor, este servidor puede ser físico o virtual, luego instalamos nuestro sistema operativo en el servidor  y encima de este le instalamos un Container Engine el cual nos permite correr múltiples instancias invitadas, cada una de estas instancias se llama un **contenedor** y dentro de este contenedor instalamos nuestra app con todas las dependencias que esta necesita. La diferencia con el modelo hipervisor esta en la replicación de kernels, en el modelo tradicional (hipervisor) cada aplicación corre su propia copia del kernel, en cambio en el nuevo modelo solo tenemos un kernel, el cual proporcionara binarios y librerías a las aplicaciones que están corriendo en contenedores aislados. Cada contenedor compartirá el kernel base el cual es el container engine la virtualización sucede a nivel de sistema operativo. Docker es una implementación de estos sistemas basados en contenedores.

### Arquitectura cliente servidor en Docker
Docker utiliza una arquitectura cliente-servidor con un _daemon_ siendo el servidor, el usuario no interactúa de manera directa con este daemon sino que lo a través del docker client. El docker-client  es la interfaz de uso primaria  para docker, este acepta comandos del usuario y se los comunica al daemon. Tenemos 2 tipos de docker-client, la tipica linea de comando (cli) o kitematic la cual es una interfaz gráfica.
El _daemon_ es el proceso persistente el cual hace el trabajo de levantar correr y distribuir nuestros contenedores de docker, este daemon es usualmente referido como docker-server o docker-engine. En una instalación típica de linux el docker-client, el docker-daemon y cada contenedor se encuentran en la misma maquina host, pero también podemos conectar nuestro docker-client a un docker-daemon remoto. En w10 o OSX no podemos correr docker de manera nativa para esto necesitamos una docker-machine la cual es una maquina virtual de linux especialmente construida para correr docker.

###  Conceptos Docker
 
#### Imágenes
Son templates de solo lectura que utilizaremos para crear nuestros contenedores, las imágenes son creadas con el comando  `build`  ya sea por nosotros o por otros usuarios de docker. Debido a que las imágenes pueden ser demasiado grandes, estas pueden estar compuestas por capas que contengan otras imágenes, esto nos permite enviar una mínima cantidad de datos cuando estamos  transfiriendo imágenes a través de la red. Las imágenes se almacenan en un registro de docker  como  puede ser docker-hub

####  Contenedores
Los contenedores son encapsulaciones livianas y portátiles de un entorno en el cual podemos correr nuestra aplicación. Haciendo una analogía con la POO si consideramos una imagen como una clase, entonces un contenedor seria un objeto de esa clase. Los contenedores se crean a partir de las imágenes, dentro de un contenedor  existen todas las dependencias y binarios necesarios para correr la aplicación. Los contenedores son instancias de imágenes corriendo.

#### Registros
Un registro es donde almacenamos nuestras imágenes, podemos hostear nuestro propio registro de imágenes o podemos utilizar el registro publico de imágenes llamado *DockerHub*.
Dentro de un registro las imágenes son almacenadas como repositorios. Un *Docker Repository* es una colección de diferentes imagenes de docker con el mismo nombre, que tienen diferentes tags, cada tag usualmente representa una versión de la imagen.

##### DockerHub
[DockerHub](https://hub.docker.com/) es un registro publico el cual contiene un gran numero de imágenes que podemos utilizar. Los repositorios oficiales son repositorios certificados por docker, para cada repositorio podemos ver la cantidad de descargas y estrellas, lo cual nos indica la popularidad del mismo.

### Instalando Docker en Linux (ubuntu)

##### Desinstalar versiones antiguas de docker 
Como primer paso para nuestra instalación debemos desinstalar las versiones viejas de docker si es que existen en nuestro sistema.
```
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

#####  Instalar Docker usando el repositorios
Antes de instalar Docker Engine por primera vez en una nueva maquina host, debemos configurar el Docker repository. Una vez hecho eso podremos instalar y actualizar docker desde el repositorio.

##### Setup del repositorio


 1. Actualizamos el paquete`apt`  e  instalamos paquetes para permitir a  `apt` to use a repository over HTTPS:
 ```
 sudo apt-get update
 
 sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

 2.  Agregamos la GPG key oficial de docker
```
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

3. Utilizamos el siguiente comando para configurar el repositorio estable

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#####  Instalar Docker Engine
```
 sudo apt-get update

```

> Si obtenemos el error *E: The Repository 'https://...' does not have a
> Release file* debemos editar el archivo 
> `/etc/apt/sources.list.d/docker.list` y cambiar la version en este
> caso "`tricia stable"` por "`focal stable`" y hacemos nuevamente el
> `update`


```
 sudo apt-get install docker-ce docker-ce-cli containerd.io
```
##### Verificando la instalacion
Comprobamos que Docker engine se haya instalado correctamente corriendo el siguiente comando
```
 sudo docker run hello-world
```
este comando descarga una imagen de prueba y la corre dentro de su contenedor. Cuando el container corre, imprime un mensaje y luego se cierra.

    Hello from Docker!
    This message shows tha your instalation appears to be working correctly...
    ....
    ...
    
##### Controlar Docker como usuario no root

1. Creamos el `docker` group  `$ sudo groupadd docker`
2.  Agregamos nuestro usuario al grupo de docker  `$ sudo usermod -aG docker $USER`
3. Verificamos que podemos correr docker `$ docker run hello-world`


### Corriendo nuestro primer Docker Container

##### chequeando las imágenes locales
Cuando usamos una imagen para crear un contenedor docker primero busca en nuestro registro local para  encontrar la imagen, si no puede encontrarla en nuestro sistema, se conectara al registro remoto.
Para ver las imágenes que tenemos en nuestro registro local podemos correr el comando 

    docker images
    
##### corriendo un contenedor
   Para correr un contenedor usamos el comando 

    $ docker run busybox:1.24 echo "hello world"
este comando descargar la imagen de nombre busybox y especificamos la versión 1.24, luego especificamos el comando que queremos correr dentro de ese contenedor en este caso el comando `echo "hello World"`
##### Trayendo imágenes
para traer una imagen a nuestro sistema sin correrla en el contenedor también podemos usar `docker pull`

    $ docker pull <image-name:imagetag>

### Profundizando en contenedores
En la sección anterior vimos como correr un contenedor en primer plano pero en la mayoría de los casos los contenedores van a estar corriendo en segundo plano para eso necesitamos iniciar un contenedor en *detached mode* utilizando el comando `run` con la bandera `-d`. Esto significa que podemos iniciar el contenedor y utilizar la consola para correr otros comandos

    $  docker run -d busybox:1.24 sleep 1000

el comando `sleep 1000` es utilizado para generar un delay por una cantidad especifica de tiempo.

al ejecutar el comando `docker run -d` este nos devuelve un hash el cual indica el id del contenedor, podemos verificar los contenedor que están corriendo utilizando el comando 

    $ docker ps
  
  también podemos usar 

      $ docker container ls

`docker ps` nos ofrece información como el `CONTAINER ID` , el `COMMAND` para ver el comando que se esta ejecutando, `STATUS` y `NAMES` .

Si utilizamos el comando `ps -a` nos da información sobre todos los contenedor que hayamos ejecutado incluso los que ya no estén en ejecución

    $ docker ps -a
  también podemos usar:

      $  docker container ls -a

Si no queremos mantener en el registro el contenedor que ejecutamos añadimos la bandera `-rm`y docker lo removerá automáticamente una vez que el contenedor se deje de ejecutar

    $ docker run --rm busybox:1.24 sleep 1

Podemos especificar el nombre del contenedor que queremos correr

    docker run --name hello_world busybox:1.24
  
  si hacemos `docker ps -a` veremos el contenedor con el nombre que le indicamos, en este caso `hello_world`. Si no especificamos un nombre para el contenedor docker asignara un nombre por defecto cuando corramos el comando `run`. 

Si queremos mas informacion a bajo nivel sobre los contenedores o imagenes podemos usar el comando docker inspect

    $ docker inspect <container_id>
esto nos desplegara un objeto JSON con informacion como `IP`, `MAC adress`, `ImageId`.

para bajar un contenedor que este corriendo usamos el comando 

    $docker stop <id_contenedor>

#### Detener un contenedor 
Para detener un contenedor usamos 

    $ docker stop <container-id>
#### Eliminar un contenedor
Para eliminar un contenedor usamos el comando `docker rm`

    $ docker rm <container-id> 
Podemos eliminar todos los contenedores de una lista usando el comando 

    $ docker rm $( docker  ps -aq)

> Si un contenedor se encontraba corriendo el comando anterior nos dará un mensaje indicando que no podemos remover un contenedor que esta corriendo si queremos forzarlo tenemos que hacer :

 

    $ docker rm -f $( docker  ps -aq)


#### Cambiar el nombre de un contenedor 
Podemos definir el nombre de un contenedor al momento de su ejecucion usando  la bandera --name

    $ docker run --name=mi-contenedor -d -p 5000:80 nginx:latest
esto nos hace mas fácil identificar nuestro contenedores 

#### formateando salida,  ps --format
Podemos definir la informacion que queremos ver con el comando `docker ps` 

    docker  ps --format="ID\t{{.ID}}\nNAME\t{{.Names}}\nIMAGE\t{{.Image}}\nPORTS\t{{.Ports}}\nCOMMAND\t{{.Command}}\nCREATED\t{{.CreatedAt}}\nSTATUS\t{{.Status}}\n"

como toda esa linea que va de argumento es bastante complicada de recordar la guardamos en una variable del sistema como format

    export FORMAT="ID\t{{.ID}}\nNAME\t{{.Names}}\nIMAGE\t{{.Image}}\nPORTS\t{{.Ports}}\nCOMMAND\t{{.Command}}\nCREATED\t{{.CreatedAt}}\nSTATUS\t{{.Status}}\n"

ahora podemos usar 

    docker ps --format=$FORMAT

ahora podemos ver la información de manera mas elegante

### Mapeo de puertos y logs

#### Mapeo Puertos
El formato  para mapear un puerto a otro puerto usamos es  `-p host_port: container_port` en este caso probamos con una imagen de tomcat.
Esto nos va vincular el puerto 8888 con el 8080 del contenedor

    $ docker run -it -p 8888:8080 tomcat:8.0

Una vez levantado el contenedor podemos acceder al servidor tomcat a través del navegador en la ruta `localhost:8888.`

para salir del contenedor usamos `ctrl+c`

#### mapeando varios puertos 
Podemos mapear varios al puerto de un contenedor de docker, en el ejemplo anterior mapeamos el port 8080 de nuestro host al puerto 80 del contenedor.

Si por ejemplo queremos mapear el puerto 3000 y 8080  de nuestro host al puerto 80 de nuestro contenedor hacemos lo siguiente: 

    $  docker run -d -p 3000:80 -p 8080:80 nginx:latest

Simplemente agregamos mas puertos usando la bandera `-p`
ahora podremos acceder al servidor de nginx desde nuestro navegador usando el puerto 8080 y el puerto 3000

#### Logs
para ver los logs del contenedor usamos el comando 

    $ docker logs <container_id>





