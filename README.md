# Docker

## Qué es

Docker es un sistema de código abierto para correr aplicaciones, sistemas operativos y cualquier otra herramienta que usualmente necesitaría una instalación normal.

Al usar imágenes y contenedores provee la gran ventaja de ocupar mucho menos espacio en disco y uso de memoria. Docker al correr en el kernel de Linux es compatible con casi todos los SO actuales

Al descargar imágenes de aplicaciones como mysql, se puede crear uno o varios contenedores que necesiten utilizar estas imágenes, de esta manera se ahorra todo el tedioso proceso de instalación así como los posibles errores que en muchas ocasiones ocurren al momento de obtener una aplicación de manera habitual

Al descargar una imagen por ejemplo MySql:5 y la usamos en un contenedor cualquiera, si creamos otro contenedor que use esa misma imagen, docker no la vuelve a descargar sino que utiliza la ya existente, de esta manera se ahora espacio en disco.

💡 Port Mapping o mapeo de puertos: Es cuando tenemos aplicaciones corriendo en una red privada y hacemos una configuración para que dichas aplicaciones sean accesibles desde puertos con direcciones IP que no estén en dicha red. Es decir abrir los puertos a específicas direcciones para que entre las mismas puedan intercambiar información

## Comandos
  - ## Imágenes
    - **docker pull “image”**: permite descargar la imagen indicada alojada en el servidor de repositorios docker.hub.
    - **docker images**: permite visualizar todas las imágenes descargadas.
    - **docker images rm ”image”**: Elimina la imagen indicada.
  - ### Contenedores
    - **docker create ”image”**: Crea un contenedor sobre la imagen indicada (es decir podemos crear un contenedor en base a la imagen de mysql con docker create mysql) .
    - **docker create —name “container_name” ”image”** : Crea un contenedor sobre la imagen indicada  pero con el nombre especificado en el parámetro “container_name.
    - **docker create -p “local-port:container-port”**: Crea un contenedor indicando que el mismo puede recibir solicitudes desde el puerto de nuestra maquina. En caso de no especificar el puerto al cual queremos dar acceso por ejemplo “-p27017”, esto le indicara a docker el crear un contenedor el cual corre en el puerto 27017, eligiendo de manera aleatoria un puerto de nuestro PC desde el cual se podrá acceder al contenedor.
    - **docker create -p 27017:27017 —name monguito -e MONGO_INITDB_ROOT_USERNAME=nico -e MONGO_INITDB_ROOT_PASSWORD=password mongo:**  Este comando indica crear un contenedor que permita el mapeo de puertos permitiendo acceder al puerto de mongodb (27017) desde la direccion IP (0.0.0.0:27017 o localhosto:27017), el contenedor se llamará “monguito”, le indicará a la imagen de mongo mediante las variables de entorno de USERNAME y ROOT_PASSWORD el nombre usuario y las contraseña para acceder a la base de datos de mongo. Todo esto en la imagen de Mongo
    - **docker start ”container”**: Inicia el contenedor indicado (se puede especificar el id del contenedor o el nombre).
    - **docker stop ”container”**: Detiene la ejecución del contenedor especificado.
    - **docker ps**: Muestra una lista de todos los container que están en ejecución.
    - **docker ps -a**: Muestra una lista de todos los container sin importar que estén o no en ejecución.
    - **docker rm “container”**: Eliminar el contenedor especificado.
    - **docker logs “container”**: Muestra el log o historial de información resultante durante la ejecución del contenedor especificado.
    - **docker logs —f | —follow “container”**: Muestra el log o historial de información resultante durante la ejecución del contenedor especificado, escuchando los cambios que ocurran dentro del mismo
    
  - ### Imágenes/Contenedores:
    - **docker run “image”** : Este comando descarga una imagen de no existir, luego de ser descargada crea su contenedor y por último lo pone en ejecución
    - **docker run -d“image”** : Hace lo mismo que el comando anterior pero sin mostrar constantemente los logs que vayan resultando del comando **docker run.**

## Introducir aplicaciones dentro de contenedores

- Crear un archivo llamado Dockerfile sin extensión.
- En dicho archivo debemos indicarle a Docker las acciones que debe hacer al momento de crear la imagen por ejemplo.
    - FROM “image”: Este comando le indica a Docker que se debe crear un contenedor en base a la imagen indicada.
    - Luego de haber indicado la imagen con la cual se va a crear el contenedor, debemos copiar el código fuente de nuestra aplicación a la raíz de nuestro contenedor o en la carpeta del mismo que consideremos conveniente. Para eso escribimos las siguientes lineas de comando.
        - RUN mkdir -p /home/app:  Ejecuta el comando mkdir  con el argumento -p crea las carpeta home y dentro de la misma crea a la vez la carpeta app.
        - COPY . /home/app: Este comando permite copiar desde la raíz de nuestra ruta actual hacia la estructura de carpetas previamente creadas.
        
**Se usa COPY en vez de RUN ya que la instrucción de RUN nos va a permitir ejecutar instrucciones del SO Linux pero COPY no permitirá acceder al sistema de archivos de nuestro computador anfitrión para tomar esos archivos y después colocarlos dentro de nuestro contenedor.**
        - EXPOSE 3000: Como la aplicación que queremos copiar en nuestro contenedor es un servidor realizado en express, debemos indicarle a Docker que queremos    exponer es decir que se mantendrá escuchando por peticiones.
        - CMD [”node” , ”/home/app/index.js”]: Luego de haber creado la imagen, creada la estructura de archivos y copiado los archivos de la aplicación, debemos ejecutar el comando que correrá la aplicación, el cual es el comando CMD descrito mas arriba. En otras palabras este comando le indica a Docker que debe ejecutar el archivo index.js con el comando node.
        

**💡 Como tanto la app de express como el manejador de base de datos MongoDB están en contenedores distintos, no necesariamente se podrán comunicar entre si, es decir posiblemente estén ísolados (aislados uno del otro). Pueden tener comunicación hacia afuera de la red de su mismo contenedor gracias al mapeo de sus puertos, pero so no quiere decir que entre los mismos contenedores se puedan comunicar. Para resolver esta problemática necesitaremos agrupar los contenedores que queremos tengan comunicación entre si.**

## Agrupar contenedores o asociarlos a la misma red
  - docker network ls: Lista las redes de contenedores existentes en docker.
  - docker create “red_nam”: Crea una red de contenedores con el nombre indicado.       
 
  **💡Como tanto la app de express y el manejador de BD MongoDB están en la misma red de contenedores, la forma en la que se debe hacer referencia en el URI para acceder a la BD en vez de localhost se debe indicar el nombre del contenedor que hace referencia a mongodb, en este caso seria de esta forma:**
  
  ```jsx
{
	Antes:'mongodb://nico:password@localhost:27017/miapp?authSource=admin'
Despues:'mongodb://nico:password@monguitoÏ:27017/miapp?authSource=admin'
}
// Ya que el servidor de express no esta corriendo en el host 
// de la PC afitrion, sino en el host de la red de contenedores creada
```

Continuando con los pasos para preparar la creación del contenedor con el archivo Dockerfile solo resta realizar lo siguiente:

  - docker build -t miapp:1 . : el cual creara una imagen con la etiqueta “miapp:1” usando los comandos indicados en el archivo Dockerfile que esta en la raíz “.” de la carpeta en la que nos encontramos, la cual es la carpeta raíz de la aplicación de express.


**💡 Cuando varios contenedores de docker dentro de una misma red interna de docker necesitan comunicarse entre si, el nombre del dominio correspondiente a cada contenedor es el nombre con el cual este creado dichos contenedores.**

## Docker compose

Permite crear contenedores con toda las configuraciones necesarias mediante un archivo llamado docker-compose.yml

### Estructura de archivo docker-compose.yml.

```
version: "3.9" <-- Version del grupo de contenedores
  services:<-- Servicios
    chanchito: <-- Nombre de contenedor
      build: . <-- Ejecuta el conjunto de instrucciones indicadas en el archivo Dockerfile
      ports: <-- Puertos mediante los cuales se podra acceder al contenedor
        - "3000:3000" <-- El puerto de la derecha es el de la PC afitrion (Donse se instalo Docker),
											<-- El de la derecha es el del contenedor	
      links: <-- Listado de contenedores que estaran conectados a este
        - monguito

    monguito:
      image: mongo <-- Imagen a utilizar para este contenedor
      environment: <-- Listado de variables de entorno
        - MONGO_INITDB_ROOT_USERNAME=nico
        - MONGO_INITDB_ROOT_PASSWORD=password
      ports:
        - "27017:27017"
```

- ### Comandos de docker compose
    - “docker-compose o docker compose up”: Crea o inicia (en caso de ya existir) los contenedores y servicios definidos en el archivo docker-compose.yml.
    - “”docker-compose o docker compose down”: Finaliza y desmonta los servicios así como los contenedores especificados en el arhivo docker-composeéyml.
    - “docker-compose -f `archivo` o docker compose `archivo` up": Hace lo mismo que el comando docker-compose up o docker compose up, con la excepción que usará la configuración de un archivo docker-compose personalizado (por ejemplo docker-compose-dev.yml).
    

### Volumenes

Los volúmenes son el mecanismo preferido para persistir la data generada por uno o varios contenedores de docker. Mientras que las monturas enlazadas (bind mounts) son dependientes a la estructura de carpetas an SO de la PC anfitrión, los volúmenes son completamente manejados por docker.

#### Tipos de volúmenes

1. **Anónimos:**  En este tipo de volúmenes su ubicación es manejada por Docker, Lo malo con este tipo de volumen es que después no vamos a poder hacer referencia a los mismos para que lo utilice otro contenedor.
2. **Host:** En este caso tu decides qué carpeta montar y en donde montarla. Este tipo de volumen su data es almacenada en la PC anfitriona y no en el contenedor de Docker.
3. **Nombrado:** Es igual que el anónimo solo que en este si se puede hacer referencia para ser utilizado por otro contenedor eventualmente cuando vayas a crear otro contenedor. De esta manera puedes reutilizarlo no solo con la misma imagen sino con otras imágenes que vayas creando en el futuro.

**Ejemplo**

```
monguito:
    image: mongo
    environment:
      - MONGO_INITDB_ROOT_USERNAME=nico
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes: <-- listado de volúmenes a utilizar en este contenedor
      - mongo-data:/data/db <-- A la izquierda el nombre del volumen definido en la etiquta volumes del archivo en general
			    <-- A la derecha la ruta por defecto y definida para el usuario donde se guardará la data.
    ports:
      - "27017:27017"
volumes: <-- Etiqueta que indica el listado de los volúmenes a utilizar en nuestros contenedores
  mongo-data:
```


## Aplicacion de ejemplo para curso de Docker, del canal Hola Mundo.

Curso completo gratis aca: https://www.youtube.com/watch?v=4Dko5W96WHg

