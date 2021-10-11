
# Docker

### Introduccion a las tecnologias de virtualizacion
Docker es una implementacion de las tecnologias de virtualizacion basadas en contendores, 

#### Mundo Pre- Virtualizacion
Antes de que exista la virtualizacion utilizabamos servidores fisicos, en la base teniamos los servidores, luego le instalabamos  el sistema operativo y corriamos nuestra aplicacion en el servidor. Cada Servidor corria una aplicacion. Esto llevaba a grandes costos debido a los precios de los servidores y muchas  veces solo se aprovechaba una fraccion de la capacidad de procesamiento del servidor. Otro gran problema era el tiempo de despliegue de una aplicacion y lo dificil de migrar de un servidor a otro en caso de que fuera necesario.

#### Virtualizacion basada en  Hypervisor
En esta tecnologia tambien tenemos el servidor fisico en la base y luego le instalamos nuestro sistema operativo y encima de nuestro sistema operativo instalamos una capa hipervisor, la cual nos permite instalar multiples maquinas virtuales en una sola maquina, cada una de estas maquinas puede correr un sistema operativo distinto y asi podemos correr distintas aplicaciones en una sola maquina. Los provedores mas populares de estas tecnologia basada en hipervisor son VMware, VirtualBox. En los inicios de esta tecnologias los usuarios desplegaban sus maquinas virtuales en sus propios servidores fisicos pero hoy en dia hay una migracion a utilizar servicios de virtualizacion en la nube utilizando servicios como AWS, o  Microsoft Azure. Donde no necesitamos comprar nuestro servidores fisicos.
Hay multiples beneficios que esta tecnologia nos ofrece (cloud) , por ejemplo es mas eficiente en terminos de costo ya que cada maquina virtual esta divida en multiples VMs( virtual machines)   y cada una solo utiliza su propia CPU memoria y recursos de almacenamiento, solo pagamos por lo que utilizamos. Tambien es facil de escalar ya que solo cliqueamos y desplegamos mas VMs en la nube.
Dentro de las desventajas encontramos que cada maquina virtual necesita su sistema operativo, lo cual lo hace poco eficiente. La portabilidad no puede ser asegurada.

#### Virtualizacion basada en contenedores
En la Virtualizacion basada en contenedores, en la base de este stack tenemos el servidor, este servidor puede ser fisico o virtual, luego instalamos nuestro sistema operativo en el servidor  y encima de este le instalamos un Container Engine el cual nos permite correr multiples instancias invitadas, cada una de estas instancias se llama un **contenedor** y dentro de este contenedor instalamos nuestra app con todas las dependencias que esta necesita. La diferencia con el modelo hipervisor esta en la replicacion de kernels, en el modelo tradicional (hipervisor) cada aplicacion corre su propia copia del kernel, en cambio en el nuevo modelo solo tenemos un kernel, el cual proporcionara binarios y librerias a las aplicaciones que estan corriendo en contenedores aislados. Cada contenedor compartira el kernel base el cual es el container engine la virtualizacion sucede a nivel de sistema operativo. Docker es una implementacion de estos sistemas basados en contenedores.

### Arquitectura cliente servidor en Docker
Docker utiliza una arquitectura cliente-servidor con un _daemon_ siendo el servidor, el usuario no interactua de manera directa con este daemon sino que lo a traves del docker client. El docker-client  es la interfaz de uso primaria  para docker, este acepta comandos del usuario y se los comunica al daemon. Tenemos 2 tipos de docker-client, la tipica linea de comando (cli) o kitematic la cual es una interfaz grafica.
El _daemon_ es el proceso persistente el cual hace el trabajo de levantar correr y distribuir nuestros contenedores de docker, este daemon es usualmente refererido como docker-server o docker-engine. En una instalacion tipica de linux el docker-client, el docker-daemon y cada contenedor se encuentran en la misma maquina host, pero tambien podemos conectar nuestro docker-client a un docker-daemon remoto. En w10 o OSX no podemos correr docker de manera nativa para esto necesitamos una docker-machine la cual es una maquina virtual de linux especialmente construida para correr docker.

###  Conceptos Docker
 
#### Imagenes
Son templates de solo lectura que utilizaremos para crear nuestros contenedores, las imagenes son creadas con el comando  `build`  ya sea por nosotros o por otros usuarios de docker. Debido a que las imagenes pueden ser demasiado grandes, estas pueden estar compuestas por capas que contengan otras imagenes, esto nos permite enviar una minima cantidad de datos cuando estamos  transfiriendo imagenes a traves de la red. Las imagenes se almacenan en un registro de docker  como  puede ser docker-hub

####  Contenedores
Los contenedores son encapsulaciones livianas y portables de un entorno en el cual podemos correr nuestra aplicacion. Haciendo una analogia con la POO si consideramos una imagen como una clase, entonces un contenedor seria un objeto de esa clase. Los contenedores se crean a partir de las imagenes, dentro de un contenedor  existen todas las dependencias y binarios necesarios para correr la aplicacion.

#### Registros
Un registro es donde almacenamos nuestras imagenes, podemos hostear nuestro propio registro de imagenes o podemos utilizar el registro publico de imagenes llamado *DockerHub*.
Dentro de un registro las imagenes son almacenadas como repositorios. Un *Docker Repository* es una coleccion de diferentes imagenes de docker con el mismo nombre, que tienen diferentes tags, cada tag usualmente representa una version de la imagen.

##### DockerHub
[DockerHub](https://hub.docker.com/) es un registro publico el cual contiene un gran numero de imagenes que podemos utilizar. Los repositorios oficiales son repositorios certificados por docker, para cada repositorio podemos ver la cantidad de descargas y estrellas, lo cual nos indica la popularidad del mismo.

### Instalando Docker en Linux (ubuntu)

##### Desinstalar versiones antiguas de docker 
Como primer paso para nuestra instalacion debemos desinstalar las versiones viejas de docker si es que existen en nuestro sistema.
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
    

### Corriendo nuestro primer Docker Container

##### chequeando las imagenes locales
Cuando usamos una imagen para crear un contenedor docker primero busca en nuestro registro local para  encontrar la imagen, si no puede encontrarla en nuestro sistema, se conectara al registro remoto.
Para ver las imagenes que tenemos en nuestro registro local podemos correr el comando 

    docker images
    
   ##### corriendo un contenedor
   Para correr un contenedor usamos el comando 

    docker run busybox:1.24 echo "hello world"
este comando descargar la imagen de nombre busybox y especificamos la version 1.24, luego especificamos el comando que queremos correr dentro de ese contenedor en este cado el comando `echo "hello World"`
