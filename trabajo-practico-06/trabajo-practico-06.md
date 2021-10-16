## Trabajo Práctico 6 - Construcción de Imágenes de Docker

### 1- Objetivos de Aprendizaje
 - Adquirir conocimientos para construir y publicar imágenes de Docker.
 - Familiarizarse con el vocabulario.

### 2- Unidad temática que incluye este trabajo práctico
Este trabajo práctico corresponde a la unidad Nº: 3

### 3- Consignas a desarrollar en el trabajo práctico:
 - En los puntos en los que se pida alguna descripción, realizarlo de la manera más clara posible.

### 4- Desarrollo:

#### 1- Conceptos de Dockerfiles
  - Leer https://docs.docker.com/engine/reference/builder/ (tiempo estimado 2 horas)
  - Describir las instrucciones
     - FROM
     - RUN
     - ADD
     - COPY
     - EXPOSE
     - CMD
     - ENTRYPOINT

Un docker file es un documento de texto que contiene todos los comandos que un usuario usaria por linea de comandos paara reensamblar una imagen.
Usando docker build PATH podemos construir unaa imagen desde un Dockerfile.
Formato del dockerfile:
- FROM: Un dockerfile debe empezar con un FROM. Indica la imagen base sobre la que se construirá la aplicación dentro del contenedor. Por ejemplo la imagen puede ser un sistema operativo como Ubuntu, Centos, etc. O una imagen ya existente en la cual con base a esta queramos construir nuestra propia imagen.
- RUN: nos permite ejecutar comandos en el contenedor, por ejemplo, instalar paquetes o librerías (apt-get, yum install, etc.). La instrucción RUN ejecutará cualquier comando en una nueva capa encima de la imagen actual y commiteara los resultados. La imagen commiteada resultante se utilizará para el siguiente paso en el Dockerfile. La creación de capas de instrucciones RUN y la generación de confirmaciones se ajusta a los conceptos centrales de Docker, donde las confirmaciones son económicas y los contenedores se pueden crear desde cualquier punto del historial de una imagen, al igual que el control de código fuente.
- ADD: esta instrucción copia archivos a un destino especifico dentro del contenedor, normalmente nos sirve para dejar ubicados ciertos archivos que queremos mover entre directorios.

     - COPY: La instrucción COPY copia nuevos archivos o directorios de <src> y los agrega al sistema de archivos del contenedor en la ruta <dest>. Toma un src y un destino. Solo le permite copiar en un archivo o directorio local desde su host (la máquina que crea la imagen de Docker) en la propia imagen de Docker.ADD también te permite hacer eso, pero también admite otras 2 fuentes. Primero, puede usar una URL en lugar de un archivo / directorio local. En segundo lugar, puede extraer un archivo tar de la fuente directamente al destino.
     - EXPOSE: La instrucción EXPOSE informa a Docker que el contenedor escucha en los puertos de red especificados en tiempo de ejecución. Puede especificar si el puerto escucha en TCP o UDP, y el valor predeterminado es TCP si no se especifica el protocolo.La instrucción EXPOSE no publica realmente el puerto. Funciona como un tipo de documentación entre la persona que construye la imagen y la persona que ejecuta el contenedor, sobre qué puertos se pretenden publicar.
     - CMD: esta instrucción nos provee valores por defecto a nuestro contenedor, es decir, mediante esta podemos definir una serie de comandos que solo se ejecutaran una vez que el contenedor se ha inicializado, pueden ser comandos Shell con parámetros establecidos.


     - ENTRYPOINT: la instrucción entrypoint define el comando y los parámetros que se ejecutan primero cuando se ejecuta el contenedor. En simples palabras, todos los comandos pasados en la instrucción docker run <image> serán agregados al comando entrypoint

En conclusión, un dockerfile se encarga de construir una imagen para una determinada función. Como he mencionado anteriormente, es una receta de cocina donde seguimos ciertas instrucciones para construir un resultado final esperado.


#### 2- Generar imagen de docker
   - Clonar/Actualizar el repositorio de https://github.com/alexisfr/ing-soft-3-2020 
   - El código se encuentra en la carpeta `./proyectos/spring-boot`
   - Se puede copiar al repositorio personal en una carpeta `trabajo-practico-06/spring-boot`
   - Compilar la salida con:
```bash
cd proyectos/spring-boot
mvn clean package spring-boot:repackage  
```
   - Agregar un archivo llamado **Dockerfile** (en el directorio donde se corrió el comando mvn)
```Dockerfile
FROM java:8-jre-alpine

RUN apk add --no-cache bash

WORKDIR /app

COPY target/*.jar ./spring-boot-application.jar

ENV JAVA_OPTS="-Xms32m -Xmx128m"
EXPOSE 8080

ENTRYPOINT exec java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar spring-boot-application.jar
```
Para este dockerfile:
- corremos la imagen bajo java 8
- instalamos bash
- le damos el directorio en donde trabajamos
- copiamos todos los .jar generados con el previo comando de mvn clean package en /target y lo pegamos en un nuevo jar ./spring-boot-application.jar
- exponemos puerto 8080
- con entrypoint le decimos el comando que queremos ejecutar cuando el contenedor se ejecute

 
   - Generar la imagen de docker con el comando build
```bash
docker build -t test-spring-boot .
```
  - Ejecutar el contenedor
```bash
docker run -p 8080:8080 test-spring-boot
```
  - Capturar y mostrar la salida.
  - Verificar si retorna un mensaje (correr en otro terminal o browser)
```bash
curl -v localhost:8080
```

![alt text here]( https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/curl.png)


#### 3- Dockerfiles Multi Etapas
Se recomienda crear compilaciones de varias etapas para todas las aplicaciones (incluso las heredadas). En resumen, las compilaciones de múltiples etapas:

- Son independientes y auto descriptibles
- Resultan en una imagen de Docker muy pequeña
- Puede ser construido fácilmente por todas las partes interesadas del proyecto (incluso los no desarrolladores)
- Son muy fáciles de entender y mantener.
- No requiere un entorno de desarrollo (aparte del código fuente en sí)
- Se puede empaquetar con pipelines muy simples

Las compilaciones de múltiples etapas también son esenciales en organizaciones que emplean múltiples lenguajes de programación. La facilidad de crear una imagen de Docker por cualquier persona sin la necesidad de JDK / Node / Python / etc. no puede ser sobrestimado.

- Modificar el dockerfile para el proyecto Java anterior de la siguiente forma
```dockerfile
FROM maven:3.5.2-jdk-8-alpine AS MAVEN_TOOL_CHAIN
COPY pom.xml /tmp/
RUN mvn -B dependency:go-offline -f /tmp/pom.xml -s /usr/share/maven/ref/settings-docker.xml
COPY src /tmp/src/
WORKDIR /tmp/
RUN mvn -B -s /usr/share/maven/ref/settings-docker.xml package

FROM java:8-jre-alpine

EXPOSE 8080

RUN mkdir /app
COPY --from=MAVEN_TOOL_CHAIN /tmp/target/*.jar /app/spring-boot-application.jar

ENV JAVA_OPTS="-Xms32m -Xmx128m"

ENTRYPOINT exec java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app/spring-boot-application.jar

HEALTHCHECK --interval=1m --timeout=3s CMD wget -q -T 3 -s http://localhost:8080/actuator/health/ || exit 1
```
- Construir nuevamente la imagen
```bash
docker build -t test-spring-boot .
```
- Analizar y explicar el nuevo Dockerfile, incluyendo las nuevas instrucciones.

#### Para este Dockerfile:
- FROM: Construimos la imagen bajo una imagen base de jdk 8
- COPY: copiamos el pom.xmml a /tmp/
- RUN: corremos comando mvn para decirle que ejecute en comando no interactivo, utilice forzadamente un .xml y setee un camino alternativo de usr para settings-docker.xml
- COPY: copiamos el src a /tmp/src

- FROM: corremos la imagen bajo jre 8
- exponemos puerto 8080
- creamaos carpeta /app
- copiamos todos los .jar de /tmp/target/*.jar y lo pegamos en /app/spring-boot-application.jar
- ENTRYPOINT: ejecutamos el jar /app/spring-boot-application.jar


#### 4- Python Flask
  - Utilizar el código que se encuentra en la carpeta `./proyectos/python-flask`
  - Se puede copiar al repositorio personal en una carpeta `trabajo-practico-06/python-flask`
  - Correr el comando
```bash
cd ./proyectos/python-flask
docker-compose up -d
```

  - Explicar que sucedió!
  - ¿Para qué está la key `build.context` en el docker-compose.yml?

Al correr el comando lo que estamos haciendo es levantar el docker-compose.yml 
  - Con build: context: ./ lo que hacemos es indicar donde esta el Dockerfile que queremos utilizar para crear el contenedor. Al definier “./” automaticamente considerará el Dockerfile existente en directorio actual
```
FROM python:3.6.3

ENV BIND_PORT 5000
ENV REDIS_HOST localhost
ENV REDIS_PORT 6379

COPY ./requirements.txt /requirements.txt

RUN pip install -r /requirements.txt

COPY ./app.py /app.py

EXPOSE $BIND_PORT

CMD [ "python", "/app.py" ]
```
En este dockerfile creamos el entorno de nuestra aplicación, identificamos las variables de entorno y al final terminamos ejecutando python /app.py

Luego el docker-compose:

  - Identifica las variables de entorno para Redis
  - Los puertos que utilizara ese contenedor 5000:5000
  - Identificamos la imagen de redis y el volumen en donde persistira los datos
  - Crea la network python-flask_default y el volumen python-flask_redis_data

#### 5- Imagen para aplicación web en Nodejs
  - Crear una la carpeta `trabajo-practico-06/nodejs-docker`
  - Generar un proyecto siguiendo los pasos descriptos en el trabajo práctico 5 para Nodejs
  - Escribir un Dockerfile para ejecutar la aplicación web localizada en ese directorio
    - Idealmente que sea multistage, con una imagen de build y otra de producción.
    - Usar como imagen base **node:13.12.0-alpine**
    - Ejecutar **npm install** dentro durante el build.
    - Exponer el puerto 3000
  - Hacer un build de la imagen, nombrar la imagen **test-node**.
  - Ejecutar la imagen **test-node** publicando el puerto 3000.
  - Verificar en http://localhost:3000 que la aplicación está funcionando.
  - Proveer el Dockerfile y los comandos ejecutados como resultado de este ejercicio.

```
$ npm start

> hola-mundo@0.0.0 start
> node ./bin/www

```
![alt text here]( https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/dockerfile.png)

```
$ docker build -t test-node .
[+] Building 31.2s (12/12) FINISHED
```
```
$ docker run -d -p 3000:3000 test-node
d3ce987374fa3b217a520001d395a251836ebed57946b8ad19eddf9daf2870ba
```
![alt text here]( https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/node.png)


#### 6- Publicar la imagen en Docker Hub.
  - Crear una cuenta en Docker Hub si no se dispone de una.
  - Registrase localmente a la cuenta de Docker Hub:
```bash
docker login
```
  - Crear un tag de la imagen generada en el ejercicio 3. Reemplazar <mi_usuario> por el creado en el punto anterior.
```bash
docker tag test-node <mi_usuario>/test-node:latest
```
  - Subir la imagen a Docker Hub con el comando
```bash
docker push <mi_usuario>/test-node:latest
``` 
  - Como resultado de este ejercicio mostrar la salida de consola, o una captura de pantalla de la imagen disponible en Docker Hub.

![alt text here]( https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/busybox.png)


## Anotaciones:

#### Construcción de imágenes Docker en múltiples etapas

Necesitamos construir imágenes Docker reducidas. Esto es, que sean poco pesadas, con sólo lo necesario para hacer lo que tengamos que hacer en dicha imagen (construir una aplicación, ejecutar una aplicación, etc.).
Esto ayuda a la hora de realizar construcciones más rápidamente, la descarga de dicha imagen es más rápida, se necesita menos almacenamiento, tiene mejor rendimiento y, además, es más seguro (un atacante cuenta con menos posibilidades si no vamos dejando herramientas o cualquier cosa por ahí, que luego pueda aprovechar).

#### Builder Patter
Como en producción sólo se quiere tener aquello necesario para que nuestro sistema se ejecute, y utilizando como ejemplo la problemática de utilizar lenguajes compilados (por llamarlo de alguna manera, ya que no hay nada de malo en usar un tipo de lenguaje u otro), es decir, tener que construir antes de ejecutar, era muy común contar con tres ficheros para poder solucionar esto.
Concretamente, se tenía un fichero llamado Dockerfile.build el cual contenía las instrucciones de construcción
```
FROM golang:1.7.3
WORKDIR /go/src/github.com/alexellis/href-counter/
COPY app.go .
RUN go get -d -v golang.org/x/net/html && CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

```

Otro fichero, llamado Dockerfile, contenía las instrucciones de construcción de la imagen que se desea llevar a producción
```
FROM alpine:latest 
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY app .
CMD ["./app"] 

```

Finalmente, se necesitaba de un script (llamado build.sh) de construcción de ambas imágenes 
```
#!/bin/sh
echo Building alexellis2/href-counter:build

docker build --build-arg https_proxy=$https_proxy --build-arg http_proxy=$http_proxy \ 
-t alexellis2/href-counter:build . -f Dockerfile.build

docker container create --name extract alexellis2/href-counter:build 
docker container cp extract:/go/src/github.com/alexellis/href-counter/app ./app 
docker container rm -f extract

echo Building alexellis2/href-counter:latest

docker build --no-cache -t alexellis2/href-counter:latest .
rm ./app

```


#### Multi-stage build
Utilizando la principal características de Docker, las capas, esta nueva funcionalidad nos ayuda a reducir complejidad a la hora de realizar el proceso descrito anteriormente.
Se trata de utilizar un único fichero Dockerfile, nombrando los stages (o etapas):

```
FROM golang:1.7.3 as builder
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html 
COPY app.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest 
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
CMD ["./app"]

```
Básicamente, hacemos lo mismo que en el patron anterior, sólo que indicamos que en la construcción de la segunda imagen (la que tiene como objetivo ejecutarse en producción) tome el ejecutable de la aplicación de la imagen previa (from=builder), nombrada como builder.

La compilación de múltiples etapas permite usar múltiples comandos FROM en el mismo Dockerfile. El último comando FROM produce la imagen final de Docker, todas las demás imágenes son imágenes intermedias (no se produce ninguna imagen final de Docker, pero todas las capas se almacenan en caché).

Para copiar archivos de imágenes intermedias, use COPY --from = <image_AS_name | image_number>, donde el número comienza desde 0 (pero es mejor usar el nombre lógico a través de la palabra clave AS).

Youtube
Hay imágenes de docker alpine/ busy box que son como un linux entero que pesan tan solo 5 megas, entonces por ejemplo si tenemos una aplicación en go que la queremos hacer ejecutar necesitamos descargar muchaas librerias y archivos binarios para que esto suceda y terminamos teniendo una imagen demasiado pesada.
Antes se usaba tener una imagen para compilar y otra para correr la imagen. Entonces lo compilamos y despues la copiabamos en otro contenedor para correrla.
 
Entonces en el primer from compilamos la aplicación en go y despues en el segundo contenedor copiamos la aplicación de go dentro de una base alpine que seria un linux muy ligero (la llevo a produccion). Finalmente levanta el binario de Go 

