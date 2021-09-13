# TP3
## 1- Sistema distribuido simple
#### Ejecutar el siguiente comando para crear una red en docker
Creamos una red (mybridge) y conectamos dos contenedores en esa red
``` 
docker network create -d bridge mybridge
``` 
4a9f91edc5c42de7ad03a6e8990e8c4096a2437d2ef94c89c29832256181b060
#### Instanciar una base de datos Redis conectada a esa Red.
``` 
 docker run -d --net mybridge --name db redis:alpine
``` 

#### Levantar una aplicacion web, que utilice esta base de datos
``` 
  docker run -d --net mybridge -e REDIS_HOST=db -e REDIS_PORT=6379 -p 5000:5000 --name web alexisfr/flask-app:latest
``` 
#### Abrir un navegador y acceder a la URL: http://localhost:5000/
#### Verificar el estado de los contenedores y redes en Docker, describir:
#### ¿Cuáles puertos están abiertos?
#### Mostrar detalles de la red mybridge con Docker.
#### ¿Qué comandos utilizó?

``` 

docker network ls
NETWORK ID     NAME       DRIVER    SCOPE
7528f87483db   bridge     bridge    local
ef260ecc9a14   host       host      local
4a9f91edc5c4   mybridge   bridge    local
9711aa856bcc   none       null      local
``` 
``` 


docker network inspect mybridge
[
    {
        "Name": "mybridge",
        "Id": "4a9f91edc5c42de7ad03a6e8990e8c4096a2437d2ef94c89c29832256181b060",
        "Created": "2021-08-30T14:54:12.787309Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "4390ad0d274b551ca58d336e8a6963dcbbc3001a632a48f77c7bdd7ed381f877": {
                "Name": "db",
                "EndpointID": "228ff31a673595af97bea1a2d88815b71abe89b7d827456313ee47f6dd1dccd9",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "6512575bee9cd75355c70b86df1989f423ca641964c6bda1d81e13e4eb24422e": {
                "Name": "web",
                "EndpointID": "6356a43264a46b7030b3f046a98e5e721bd753df9ef071b32ef947628f3b1bcc",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
``` 
``` 

lsof -PiTCP -sTCP:LISTEN

com.docke  2209 ticianacobresi   73u  IPv6 0x52db3624c4fb278b      0t0  TCP *:3305 (LISTEN)
com.docke  2209 ticianacobresi   75u  IPv6 0x52db3624df6a612b      0t0  TCP *:5000 (LISTEN)
Dropbox   43192 ticianacobresi   81u  IPv6 0x52db3624c51a9deb      0t0  TCP *:17500 (LISTEN)
Dropbox   43192 ticianacobresi   82u  IPv4 0x52db3624d8f6a96b      0t0  TCP *:17500 (LISTEN)
Dropbox   43192 ticianacobresi  125u  IPv4 0x52db3624d6beadbb      0t0  TCP localhost:17600 (LISTEN)
Dropbox   43192 ticianacobresi  132u  IPv4 0x52db3624d8fdcaf3      0t0  TCP localhost:17603 (LISTEN)
``` 
## 2- Análisis del sistema
•	Siendo el código de la aplicación web el siguiente:
import os

from flask import Flask
from redis import Redis


app = Flask(__name__)
redis = Redis(host=os.environ['REDIS_HOST'], port=os.environ['REDIS_PORT'])
bind_port = int(os.environ['BIND_PORT'])


@app.route('/')
def hello():
    redis.incr('hits')
    total_hits = redis.get('hits').decode()
    return f'Hello from Redis! I have been seen {total_hits} times.'


if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True, port=bind_port)

### Explicar cómo funciona el sistema
En el programa se inicializa una variable que va a ser una instancia de aplicación de Flask y una variable que instancia la base de datos redis. Se instancian el puerto y nombre de host que son las variables de entorno que definimos en nuestra instancia de app/flask-docker (En la red mybrindge instancie la bd redis y conecte la aplicacion app/flask a esa base de datos a partir de las variables de entorno definidas)
Al ejecutar el programa en el localhost:<REDIS_PORT>/ el programa nos va a responder con 'Hello from Redis! …'

### ¿Para qué se sirven y porque están los parámetros -e en el segundo Docker run del ejercicio 1?
Con -e se setean las variables de entorno sea nombre de host, puerto, password, nombre bd para la base de datos. Estas variables son las que despues utilizo para conectar mi programa con la BD de redis

### ¿Qué pasa si ejecuta docker rm -f web y vuelve a correr docker run -d --net mybridge -e REDIS_HOST=db -e REDIS_PORT=6379 -p 5000:5000 --name web alexisfr/flask-app:latest ?
Borramos el contenedor aun cuando esta corriendo de la flask-app. Por lo tanto ya no puedo entrar al localhost:5000 y despues corri la imagen que si la encuentra, no tiene que ir a descargar la imagen de nuevo por mas que la hayamos puesto borrar instancia

### ¿Qué occure en la página web cuando borro el contenedor de Redis con docker rm -f db?
redis.exceptions.ConnectionError
Lo que ocurre es que la flask-app ya no encuentra posible la conexión a la base de datos redis, que es la bd con la cual la habiamos conectado dentro de la red mybridge

###  Y si lo levanto nuevamente con docker run -d --net mybridge --name db redis:alpine ?
Si levanto nuevamente una instancia del bd, la aplicación si puede conectarse y levantarse porque la conexión es factible
### ¿Qué considera usted que haría falta para no perder la cuenta de las visitas?
Podriamos dejar en local los registros que fuimos haciendo y persistiendo en la base de datos redis, especificamente el control de vistas para la app-flask
###  Para eliminar los elementos creados corremos:
docker rm -f db
docker rm -f web
docker network rm mybridge

## 3- Utilizando docker compose
###  Normalmente viene como parte de la solucion cuando se instaló Docker
###  De ser necesario instalarlo hay que ejecutar:
sudo pip install docker-compose
###  Crear el siguente archivo docker-compose.yaml en un directorio de trabajo:
version: '3.6'
services:
  app:
    image: alexisfr/flask-app:latest
    depends_on:
      - db
    environment:
      - REDIS_HOST=db
      - REDIS_PORT=6379
    ports:
      - "5000:5000"
  db:
    image: redis:alpine
    volumes:
      - redis_data:/data
volumes:
  redis_data:


Yo armo mi aplicacion y le digo como quiero que este configurada.

###  Ejecutar docker-compose up -d
``` 
docker-compose up -d
Creating network "practico_default" with the default driver
Creating volume "practico_redis_data" with default driver
Creating practico_db_1 ... done
Creating practico_app_1 ... done
``` 
###  Acceder a la url http://localhost:5000/
###  Ejecutar docker ps, docker network ls y docker volume ls
###  ¿Qué hizo Docker Compose por nosotros? Explicar con detalle.
###  Desde el directorio donde se encuentra el archivo docker-compose.yaml ejecutar:
docker-compose down


Docker Compose nos permite usar un archivo YAML para definir aplicaciones de múltiples contenedores. Es posible configurar tantos contenedores como queramos, cómo se deben construir y conectar, y dónde se deben almacenar los datos. Podemos ejecutar un solo comando para compilar, ejecutar y configurar todos los contenedores cuando el archivo YAML esté completo.
Creamos entonces un archivo yaml y definimos una app: el contenedor de la aplicacion flask y le decimos que depende de la imagen db que es el contenedor que tiene la base de datos, le definimos las variables de entorno para que la aplicación se conecte con la bd y los puertos de la imagen flask-app. Luego definimos una base de datos que en nuestro caso esta dada por el contenedor redis:alpine y definimos el volumen local para que la bd almacene lo persistido.


``` 

docker ps
f214c93cd5e1   alexisfr/flask-app:latest   "python /app.py"         About a minute ago   Up About a minute   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   practico_app_1
45809928ab6f   redis:alpine                "docker-entrypoint.s…"   About a minute ago   Up About a minute   6379/tcp                                    practico_db_1

$ docker network ls
45441967eeb8   bridge             bridge    local
ef260ecc9a14   host               host      local
4a9f91edc5c4   mybridge           bridge    local
9711aa856bcc   none               null      local
a540dc3e3777   practico_default   bridge    local

$ docker volume ls
local     8b0cd3f340cb2b1accdbfeb3ef14aedba5b3852c74efdba8f1646701f2814695
local     60c38bcf52ce3f4f7a196cc18d8b828c8230faeaad35f086d6016101c3677cb5
local     094daa3ca391414a8127d083c06f76a9b57c4736734e608f1e500fe2c62907e0
local     c8d759973771a792abe5e077f3e1a73051d4991bf4847009a420dd2f34fd4f79
local     practico_redis_data
``` 
## 4- Aumentando la complejidad, análisis de otro sistema distribuido.
Este es un sistema compuesto por:
Una aplicación web de Python que te permite votar entre dos opciones
Una cola de Redis que recolecta nuevos votos
Un trabajador .NET o Java que consume votos y los almacena en...
Una base de datos de Postgres respaldada por un volumen de Docker
Una aplicación web Node.js que muestra los resultados de la votación en tiempo real.

### Pasos:
Clonar el repositorio https://github.com/dockersamples/example-voting-app. Abrir una línea de comandos y ejecutar
cd example-voting
docker-compose -f docker-compose-javaworker.yml up -d
- Una vez terminado acceder a http://localhost:5000/ y http://localhost:5001
- Emitir un voto y ver el resultado en tiempo real.
- Para emitir más votos, abrir varios navegadores diferentes para poder hacerlo
- Explicar como está configurado el sistema, puertos, volumenes componenetes involucrados, utilizar el Docker compose como guía.
``` 
version: "3"

services:
   vote:
       build: ./vote
       command: python app.py
       volumes:
        - ./vote:/app
      ports:
         - "5000:80"
      networks:
         - front-tier
         - back-tier

    result:
       build: ./result
       command: nodemon server.js
       volumes:
       - ./result:/app
        ports:
         - "5001:80"
        - "5858:5858"
    networks:
      - front-tier
      - back-tier

  worker:
    build:
      context: ./worker
      dockerfile: Dockerfile.j
    networks:
      - back-tier

redis:
    image: redis:alpine
    container_name: redis
    ports: ["6379"]
    networks:
      - back-tier

  db:
    image: postgres:9.4
    container_name: db
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
    volumes:
      - "db-data:/var/lib/postgresql/data"
    networks:
      - back-tier

volumes:
  db-data:

networks:
  front-tier:
  back-tier:
``` 

 “build .”: Indica el path para builderar el contexto. Se utiliza para indicar donde está el Dockerfile que queremos utilizar para crear el contenedor. Al definier “.” automaticamente considerará el Dockerfile existente en directorio actual. Esta en ./vote

“command”: Una vez creado el contenedor, aqui lanzamos el comando python que permite ejecutar la app.py 

“volumes”: Aqui hacemos que el directorio actual se mapee directamente con el /app, lugar donde hemos creado la aplicación. De este modo, cualquier cambio en el directorio local en el host, se hará de inmediato en el contenedor.

ports: Mapeamos los puertos "5000:80", es decir que vamos a hacer un localhost:5000 y el host(donde corre docker) pone la conexión dentro del contenedor en el puerto 80 del mismo que es el que dejamos abierto, que es donde se ejecuta la app

## Analizamos parametros de network

``` 
docker network ls
8dc76739826a   example-voting-app_back-tier    bridge    local
96873d156201   example-voting-app_front-tier   bridge    local
``` 
## Analizando back-tier con un docker network inspect example-voting-app_back-tier:
### Podemos observer que en el archivo yaml que levantamos con docker-compose, para la configuracion de red tenemos: 
networks:
     - front-tier
     - back-tier

``` 

docker network inspect example-voting-app_back-tier
[
    {
        "Name": "example-voting-app_back-tier",
        "Id": "8dc76739826a615e2a9666c0f55f341fd1eb9efd1a38c2178f504fd7f2cd3b13",
        "Created": "2021-09-06T14:13:28.2703332Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.21.0.0/16",
                    "Gateway": "172.21.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "93067bbdc6e9c2081826e1b2e40d6bdf4736eeb85040eec1f504cfd7714ab5c8": {
                "Name": "example-voting-app_vote_1",
                "EndpointID": "ab9a318dae5f2a86c8a316de0a2b7a6c59042c92b8a6220721f590b9bb9fb086",
                "MacAddress": "02:42:ac:15:00:05",
                "IPv4Address": "172.21.0.5/16",
                "IPv6Address": ""
            },
            "a9865dac8b711243e08893a5b7d9fe4bfd7f5b13362bc2a5dda6fdc135e73a65": {
                "Name": "example-voting-app_result_1",
                "EndpointID": "0178eff0d1c6b9b22ecacc6e24cc40f0139cc8a676f49afc66b287a33a1bc201",
                "MacAddress": "02:42:ac:15:00:04",
                "IPv4Address": "172.21.0.4/16",
                "IPv6Address": ""
            },
            "d74d5222569f16922fa7c5d1ed7a53357521652f9066a25cc0ab5211009d2536": {
                "Name": "redis",
                "EndpointID": "3cc4d06550470d4681b7b5b1f410b99c60b2e285c1af274cbaef4fa5ffce22e0",
                "MacAddress": "02:42:ac:15:00:02",
                "IPv4Address": "172.21.0.2/16",
                "IPv6Address": ""
            },
            "d85c704ba5640f8990f7f013476221d1bcea5e3b2f83ed2818cf4b331d275443": {
                "Name": "example-voting-app_worker_1",
                "EndpointID": "a2a4d1be6c9dcd78a783d936d7bf146fe91d1eafbea9432f3c493ca8abfc57ed",
                "MacAddress": "02:42:ac:15:00:03",
                "IPv4Address": "172.21.0.3/16",
                "IPv6Address": ""
            },
            "e02458d3c7bbcf064d1423ce5a022630211e85f19ec84a5cd5ee15823a6393b4": {
                "Name": "db",
                "EndpointID": "f067724a427bbef96f3ecb198b1b8c6f99fcfadf393bed258f4bf67f98fe8a6f",
                "MacAddress": "02:42:ac:15:00:06",
                "IPv4Address": "172.21.0.6/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "back-tier",
            "com.docker.compose.project": "example-voting-app",
            "com.docker.compose.version": "1.29.2"
        }
    }
]
``` 

### Dentro del driver bridge creamos la red back-tier a la cual le conectamos los siguientes contenedores:
- example-voting-app_vote_1
- example-voting-app_result_1
- redis
- example-voting-app_worker_1
- db

## Analizando front-tier con un docker network inspect example-voting-app_front-tier:

``` 
docker network inspect example-voting-app_front-tier
[
    {
        "Name": "example-voting-app_front-tier",
        "Id": "96873d1562015a4d135b6e9e5c8cad99f19d391d3033f7e21e98b0bcb3c1fae1",
        "Created": "2021-09-06T14:13:27.9546432Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.20.0.0/16",
                    "Gateway": "172.20.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "93067bbdc6e9c2081826e1b2e40d6bdf4736eeb85040eec1f504cfd7714ab5c8": {
                "Name": "example-voting-app_vote_1",
                "EndpointID": "8dbaf5a521ae399f4ed43a9986683d49ecb1db9f0dfc1993ab4d6c2d2039d9b2",
                "MacAddress": "02:42:ac:14:00:02",
                "IPv4Address": "172.20.0.2/16",
                "IPv6Address": ""
            },
            "a9865dac8b711243e08893a5b7d9fe4bfd7f5b13362bc2a5dda6fdc135e73a65": {
                "Name": "example-voting-app_result_1",
                "EndpointID": "8526a3fdbf5421e7fc2cc177f328e68cf4d95bcdefaf823edff455e3df45266d",
                "MacAddress": "02:42:ac:14:00:03",
                "IPv4Address": "172.20.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "front-tier",
            "com.docker.compose.project": "example-voting-app",
            "com.docker.compose.version": "1.29.2"
        }
    }
]
``` 

### Dentro del driver bridge creamos la red example-voting-app_front-tier a la cual le conectamos los siguientes contenedores:
- example-voting-app_vote_1
- example-voting-app_result_1

docker-compose ps

           Name                          Command               State                          Ports
---------------------------------------------------------------------------------------------------------------
db                          
example-voting-app_result_1   
example-voting-app_vote_1     
example-voting-app_worker_1  
redis

# version is now using "compose spec"
# v2 and v3 are now combined!
# docker-compose v1.27+ required

``` 
services:
  vote:
    build: ./vote
    # use python rather than gunicorn for local dev
    command: python app.py
    depends_on:
      redis:
        condition: service_healthy
    volumes:
     - ./vote:/app
    ports:
      - "5000:80"
    networks:
      - front-tier
      - back-tier

  result:
    build: ./result
    # use nodemon rather than node for local dev
    command: nodemon server.js
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - ./result:/app
    ports:
      - "5001:80"
```    
                      
## 5- Análisis detallado
### Exponer más puertos urtos para ver la configuración de Redis, y las tablas de PostgreSQL con alguna IDE como dbeaver.
Creamos un nuevo archivo docker-compose-javaworker-edited.yml con los puertos abiertos para PostgreSQL("5432:5432") y Redis ("6379:6379") y nos conectamos a la bd por un ide

![alt text here](https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/1.png)

![alt text here](https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/2.png)

![alt text here]( https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/3.png)

### Revisar el código de la aplicación Python example-voting-app\vote\app.py para ver como envía votos a Redis.
Dentro del codigo app.py
-	crea una funcion “def get_redis(): “ en donde genera una instancia de la base de datos redis y la retorna
-	crea una segunda funcion “def hello():” en la cual crea una variable “voter_id” que va a ser igual al valor del id del votante que fue guardado en cache.
-	Si voter_id no tiene valor, le asignamos un valor hexadecimal a ese id
-	Si hacemos un POST en el endpoint, 1) creamos una variable con la bd, 2) se guarda en una variable el “vote” que hicimos (a o b). A este valor lo tomamos del objeto response(content, headers) que nos da el request.
-	Se hace un push a la base de datos de redis en la tabla “votes” y se guarda el voter_id y el vote

### Revisar el código del worker example-voting-app\worker\src\main\java\worker\Worker.java para entender como procesa los datos.
-	Dentro de la clase worker:
-	Hace un pop de lo que esta en el lugar 0 de la tabla “votes” de redis, es decir lo ultimo que entró en el registro de la tabla
-	Lo guarda en una variaable como objeto JSON
-	Desarma el json y guarda en distintaas variables a “voter_id voterID y vote vote”
-	Crea una funcion “updateVote(conexion_a_bd, id_de_votante, voto)” en la cual:
-	Inserta los valores de voterID y vote dentro de la tabla “votes” de Postgres
-	Si encuentra una excepcion (que ocurre cuando el id que obtengo del pop en la cola de redis ya habia sido previamente cargado en postgres con un determinado) hace un update con el nuevo voto para ese id.

### Revisar el código de la aplicacion que muestra los resultados example-voting-app\result\server.js para entender como muestra los valores.
-	Crea una funcion que cuenta la cantidad de votos para a y b sumando los id de los votantes para c/u

### Escribir un documento de arquitectura sencillo, pero con un nivel de detalle moderado, que incluya algunos diagramas de bloques, de sequencia, etc y descripciones de los distintos componentes involucrados es este sistema y como interactuan entre sí.
-	En la arquitectura de este sistema tenemos:
-	Una aplicación para votar en Python que registra 2 tipos de votos
-	Una base de datos redis que guarda el voto para un determinado id de votante
-	Una aplicación .NET que hace un insert del ultimo voto para ese id
-	Una base de datos Postgres que guarda el ultimo voto para un determinado id
-	Un programa que muestra los resultados de cuantos votantes hay por cada voto sumando los diferentes id para cada uno

![alt text here]( https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/4.png)

![alt text here](https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/5.png) 

Presentación del trabajo práctico 3
La presentación de este práctico forma parte del trabajo integrador, especialemente el último punto con el analisis del sistema, todos los documentos e imagenes pueden ser subidos a una carpeta trabajo-practico-03 con las salidas de los comandos utilizados, explicaciones y respuestas a las preguntas.
Reveer las clase de microservicios y ultima parte de docker
https://www.youtube.com/watch?v=CZ3wIuvmHeM

