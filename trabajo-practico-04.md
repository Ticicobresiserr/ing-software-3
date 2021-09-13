# TP4
## 1- Instanciación del sistema
- Clonar el repositorio https://github.com/microservices-demo/microservices-demo
mkdir -p socks-demo
cd socks-demo
git clone https://github.com/microservices-demo/microservices-demo.git
-	Ejecutar lo siguiente
cd microservices-demo
docker-compose -f deploy/docker-compose/docker-compose.yml up -d
-	Una vez terminado el comando docker-compose acceder a http://localhost
-	Generar un usuario
-	Realizar búsquedas por tipo de media, color, etc.
-	Hacer una compra - poner datos falsos de tarjeta de crédito ;)
![alt text here]( https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/6.png)

![alt text here]( https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/7.png)
## 2- Investigación de los componentes
### Describa los contenedores creados, indicando cuales son los puntos de ingreso del sistema
```
version: '2'

services:
  front-end:
    image: weaveworksdemos/front-end:0.3.12
    hostname: front-end
    restart: always
    cap_drop:
      - all
    read_only: true
  edge-router:
    image: weaveworksdemos/edge-router:0.1.1
    ports:
      - '80:80'
      - '8080:8080'
    cap_drop:
      - all
    cap_add:
      - NET_BIND_SERVICE
      - CHOWN
      - SETGID
      - SETUID
      - DAC_OVERRIDE
    read_only: true
    tmpfs:
      - /var/run:rw,noexec,nosuid
    hostname: edge-router
    restart: always
  catalogue:
    image: weaveworksdemos/catalogue:0.3.5
    hostname: catalogue
    restart: always
    cap_drop:
      - all
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
  catalogue-db:
    image: weaveworksdemos/catalogue-db:0.3.0
    hostname: catalogue-db
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_ALLOW_EMPTY_PASSWORD=true
      - MYSQL_DATABASE=socksdb
  carts:
    image: weaveworksdemos/carts:0.4.8
    hostname: carts
    restart: always
    cap_drop:
      - all
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid
    environment:
      - JAVA_OPTS=-Xms64m -Xmx128m -XX:+UseG1GC -Djava.security.egd=file:/dev/urandom -Dspring.zipkin.enabled=false
  carts-db:
    image: mongo:3.4
    hostname: carts-db
    restart: always
    cap_drop:
      - all
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid
  orders:
    image: weaveworksdemos/orders:0.4.7
    hostname: orders
    restart: always
    cap_drop:
      - all
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid
    environment:
      - JAVA_OPTS=-Xms64m -Xmx128m -XX:+UseG1GC -Djava.security.egd=file:/dev/urandom -Dspring.zipkin.enabled=false
  orders-db:
    image: mongo:3.4
    hostname: orders-db
    restart: always
    cap_drop:
      - all
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid
  shipping:
    image: weaveworksdemos/shipping:0.4.8
    hostname: shipping
    restart: always
    cap_drop:
      - all
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid
    environment:
      - JAVA_OPTS=-Xms64m -Xmx128m -XX:+UseG1GC -Djava.security.egd=file:/dev/urandom -Dspring.zipkin.enabled=false
  queue-master:
    image: weaveworksdemos/queue-master:0.3.1
    hostname: queue-master
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    restart: always
    cap_drop:
      - all
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid
  rabbitmq:
    image: rabbitmq:3.6.8
    hostname: rabbitmq
    restart: always
    cap_drop:
      - all
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
      - DAC_OVERRIDE
    read_only: true
  payment:
    image: weaveworksdemos/payment:0.4.3
    hostname: payment
    restart: always
    cap_drop:
      - all
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
  user:
    image: weaveworksdemos/user:0.4.4
    hostname: user
    restart: always
    cap_drop:
      - all
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    environment:
      - MONGO_HOST=user-db:27017
  user-db:
    image: weaveworksdemos/user-db:0.4.0
    hostname: user-db
    restart: always
    cap_drop:
      - all
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid
  user-sim:
    image: weaveworksdemos/load-test:0.1.1
    cap_drop:
      - all
    read_only: true
    hostname: user-simulator
    command: "-d 60 -r 200 -c 2 -h edge-router"

```

En el archivo yaml se definen las imágenes de:
-	Front-end: API que orquesta los servicios
-	Edge-router: punto de ingreso del sistema
-	Catalogue: muestra los productos con su descripcion
-	catalogue-db: Mysql para guardar catalogo
-	carts: el carro en donde se guardan los productos
-	carts-db: MongoDB para guardar info de carro
-	orders: ordenes de pedidos y su info
-	orders-db: MongoDB para ordines
-	Shipping: servicio de compras
-	rabbitmq: middleware de mensajería. Cola que almacena los mensajes de shipping hasta que queue-master los consume.
-	Queue-master: Lectura de la cola shipping mediante rabbitmq (procesa las compras). Consumidor de mensajes.

Una cola de mensajes es una forma de comunicación asíncrona de servicio a servicio que se usa en arquitecturas de microservicios y sin servidor. Los mensajes se almacenan en la cola hasta que se procesan y eliminan. Cada mensaje se procesa una vez sola, por un solo consumidor. Las colas de mensajes se pueden usar para desacoplar procesos pesados, para acumular trabajo y para clasificar cargas de trabajo.
-	Payment: proceso de metodo de pago
-	User: registro y login de usuarios
-	BD para user: almacena los usuarios
-	Load-test


Si analizamos la red:


```

docker network inspect docker-compose_default
```

### Dentro del driver bridge creamos la red docker-compose_default a la cual le conectamos los siguientes contenedores:
-	docker-compose_user-db_1
-	docker-compose_shipping_1
-	docker-compose_payment_1
-	docker-compose_edge-router_1
-	docker-compose_carts_1
-	docker-compose_orders_1
-	docker-compose_catalogue-db_1
-	docker-compose_carts-db_1
-	docker-compose_rabbitmq_1
-	docker-compose_user_1
-	docker-compose_orders-db_1
-	docker-compose_catalogue_1
-	docker-compose_queue-master_1
-	docker-compose_front-end_1


## Clonar algunos de los repositorios con el código de las aplicaciones
cd socks-demo
git clone https://github.com/microservices-demo/front-end.git
git clone https://github.com/microservices-demo/user.git
git clone https://github.com/microservices-demo/edge-router.git
## ¿Por qué cree usted que se está utilizando repositorios separados para el código y/o la configuración del sistema? Explique puntos a favor y en contra.
-	Se separa el codigo de lo que es la configuracion del sistema ya que de esta manera nuestro sistema se vuelve mas mantenible, flexible, cada microservicio puede evolucionar de manera independiente. 
-	El codigo se puede reutilizar
-	Seguramente el codigo de mi programa se vera modificado indefinidas veces, en las cuales se puede romper el programa, por lo tanto mantener la configuracion del sistema alejado del codigo es lo mas optimo. 
-	Por otra parte, si el codigo sera el mismo y no se vera modificado por mucho tiempo pero ese codigo tiene que ser instalado bajo diferentes entornos, lo mejor es que yo sea capaz de modificar esos archivos de configuracion sin modificar el codigo de la aplicación. 
-	Brinda una mayor organización y entendimiento del sistema, auque como desventaja es mas complejo de implementarlo
## ¿Cuál contenedor hace las veces de API Gateway?
-	docker-compose_front-end_1

## Cuando ejecuto este comando:
curl http://localhost/customers

```

{"_embedded":{"customer":[{"firstName":"Eve","lastName":"Berger","username":"Eve_Berger","id":"57a98d98e4b00679b4a830af","_links":{"addresses":{"href":"http://user/customers/57a98d98e4b00679b4a830af/addresses"},"cards":{"href":"http://user/customers/57a98d98e4b00679b4a830af/cards"},"customer":{"href":"http://user/customers/57a98d98e4b00679b4a830af"},"self":{"href":"http://user/customers/57a98d98e4b00679b4a830af"}}},{"firstName":"User","lastName":"Name","username":"user","id":"57a98d98e4b00679b4a830b2","_links":{"addresses":{"href":"http://user/customers/57a98d98e4b00679b4a830b2/addresses"},"cards":{"href":"http://user/customers/57a98d98e4b00679b4a830b2/cards"},"customer":{"href":"http://user/customers/57a98d98e4b00679b4a830b2"},"self":{"href":"http://user/customers/57a98d98e4b00679b4a830b2"}}},{"firstName":"User1","lastName":"Name1","username":"user1","id":"57a98d98e4b00679b4a830b5","_links":{"addresses":{"href":"http://user/customers/57a98d98e4b00679b4a830b5/addresses"},"cards":{"href":"http://user/customers/57a98d98e4b00679b4a830b5/cards"},"customer":{"href":"http://user/customers/57a98d98e4b00679b4a830b5"},"self":{"href":"http://user/customers/57a98d98e4b00679b4a830b5"}}},{"firstName":"ticiana","lastName":"cobresi","username":"ticicobresiserr","id":"61365216ee11cb0001c095b9","_links":{"addresses":{"href":"http://user/customers/61365216ee11cb0001c095b9/addresses"},"cards":{"href":"http://user/customers/61365216ee11cb0001c095b9/cards"},"customer":{"href":"http://user/customers/61365216ee11cb0001c095b9"},"self":{"href":"http://user/customers/61365216ee11cb0001c095b9"}}},{"firstName":"ticiana","lastName":"cobresi","username":"ticiana","id":"613f9650ee11cb0001c095c0","_links":{"addresses":{"href":"http://user/customers/613f9650ee11cb0001c095c0/addresses"},"cards":{"href":"http://user/customers/613f9650ee11cb0001c095c0/cards"},"customer":{"href":"http://user/customers/613f9650ee11cb0001c095c0"},"self":{"href":"http://user/customers/613f9650ee11cb0001c095c0"}}}]}}
```
## ¿Cuál de todos los servicios está procesando la operación?
-	La operación esta siendo procesada por front-end/api/user/index.js
## ¿Y para los siguientes casos?
curl http://localhost/catalogue
```
[{"id":"03fef6ac-1896-4ce8-bd69-b798f85c6e0b","name":"Holy","description":"Socks fit for a Messiah. You too can experience walking in water with these special edition beauties. Each hole is lovingly proggled to leave smooth edges. The only sock approved by a higher power.","imageUrl":["/catalogue/images/holy_1.jpeg","/catalogue/images/holy_2.jpeg"],"price":99.99,"count":1,"tag":["action","magic"]},{"id":"3395a43e-2d88-40de-b95f-e00e1502085b","name":"Colourful","description":"proident occaecat irure et excepteur labore minim nisi amet irure","imageUrl":["/catalogue/images/colourful_socks.jpg","/catalogue/images/colourful_socks.jpg"],"price":18,"count":438,"tag":["brown","blue"]},{"id":"510a0d7e-8e83-4193-b483-e27e09ddc34d","name":"SuperSport XL","description":"Ready for action. Engineers: be ready to smash that next bug! Be ready, with these super-action-sport-masterpieces. This particular engineer was chased away from the office with a stick.","imageUrl":["/catalogue/images/puma_1.jpeg","/catalogue/images/puma_2.jpeg"],"price":15,"count":820,"tag":["sport","formal","black"]},{"id":"808a2de1-1aaa-4c25-a9b9-6612e8f29a38","name":"Crossed","description":"A mature sock, crossed, with an air of nonchalance.","imageUrl":["/catalogue/images/cross_1.jpeg","/catalogue/images/cross_2.jpeg"],"price":17.32,"count":738,"tag":["blue","action","red","formal"]},{"id":"819e1fbf-8b7e-4f6d-811f-693534916a8b","name":"Figueroa","description":"enim officia aliqua excepteur esse deserunt quis aliquip nostrud anim","imageUrl":["/catalogue/images/WAT.jpg","/catalogue/images/WAT2.jpg"],"price":14,"count":808,"tag":["green","formal","blue"]},{"id":"837ab141-399e-4c1f-9abc-bace40296bac","name":"Cat socks","description":"consequat amet cupidatat minim laborum tempor elit ex consequat in","imageUrl":["/catalogue/images/catsocks.jpg","/catalogue/images/catsocks2.jpg"],"price":15,"count":175,"tag":["brown","formal","green"]},{"id":"a0a4f044-b040-410d-8ead-4de0446aec7e","name":"Nerd leg","description":"For all those leg lovers out there. A perfect example of a swivel chair trained calf. Meticulously trained on a diet of sitting and Pina Coladas. Phwarr...","imageUrl":["/catalogue/images/bit_of_leg_1.jpeg","/catalogue/images/bit_of_leg_2.jpeg"],"price":7.99,"count":115,"tag":["blue","skin"]},{"id":"d3588630-ad8e-49df-bbd7-3167f7efb246","name":"YouTube.sock","description":"We were not paid to sell this sock. It's just a bit geeky.","imageUrl":["/catalogue/images/youtube_1.jpeg","/catalogue/images/youtube_2.jpeg"],"price":10.99,"count":801,"tag":["formal","geek"]},{"id":"zzz4f044-b040-410d-8ead-4de0446aec7e","name":"Classic","description":"Keep it simple.","imageUrl":["/catalogue/images/classic.jpg","/catalogue/images/classic2.jpg"],"price":12,"count":127,"tag":["brown","green"]}]
```

curl http://localhost/tags
```
{"tags":["brown","geek","formal","blue","skin","red","action","sport","black","magic","green"],"err":null}
```
-	La operación esta siendo procesada por front-end/api/catalogue
## ¿Como perisisten los datos los servicios?
Tenemos una bd mysql paara el catalogo, otra base de datos mongodb para las ordenes el carrito y almacenar usuarios y una base de datos de colas paaraa el shipping de productos
A estas se les asignan espacios de memoria temporales (tmpfs), por lo que al dar de baja los servicios sus datos se pierden, lo cual no sucedería si se le asignara un volumen.
## ¿Cuál es el componente encargado del procesamiento de la cola de mensajes?
El componente que procesa la cola de mensajes de RabbitMQ es Queue-master
docker-compose_queue-master_1

## ¿Qué tipo de interfaz utilizan estos microservicios para comunicarse?
Investigar sobre RestAPI o mas potente grPC


