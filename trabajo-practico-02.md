# TP2
## Instalar Docker Community Edition
``` 
docker version
```
Client:
 Cloud integration: 1.0.17
 Version:           20.10.7
 API version:       1.41
 Go version:        go1.16.4
 Git commit:        f0df350
 Built:             Wed Jun  2 11:56:22 2021
 OS/Arch:           darwin/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.7
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       b0f5bc3
  Built:            Wed Jun  2 11:54:58 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.6
  GitCommit:        d71fcd7d8303cbf684402823e425e9dd2e99285d
 runc:
  Version:          1.0.0-rc95
  GitCommit:        b9ee9c6314599f1b4a7f497e1f1f856fe433d3b7
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
~
## Obtener la imagen BusyBox
```
$ docker pull busybox
```
Using default tag: latest
latest: Pulling from library/busybox
b71f96345d44: Pull complete
Digest: sha256:0f354ec1728d9ff32edcd7d1b8bbdfc798277ad36120dc3dc683be44524c8b60
Status: Downloaded newer image for busybox:latest
docker.io/library/busybox:latest
~
```
$ docker images
```
REPOSITORY          TAG       IMAGE ID       CREATED          SIZE
docker101tutorial   latest    50968e82779d   13 minutes ago   28.2MB
busybox             latest    69593048aa3a   2 months ago     1.24MB
alpine/git          latest    b8f176fa3f0d   2 months ago     25.1MB
~
## Ejecutando contenedores
```
$ docker run busybox
```
~
Cuando llamas a ejecutar, el cliente de Docker encuentra la imagen (busybox en este caso), carga el contenedor y luego ejecuta un comando en ese contenedor. Cuando ejecutamos docker run busybox, no proporcionamos un comando, por lo que el contenedor se inició, ejecutó un comando vacío y luego salió.
```
$ docker run busybox echo "Hola Mundo"
```

Hola Mundo
~
En este caso, el cliente de Docker ejecutó el comando echo en nuestro contenedor busybox y luego salió de él. 
```
$ docker ps
```

CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
~	
No hay contenedores corriendo
```
$ docker ps -a
```

CONTAINER ID   IMAGE     COMMAND               CREATED          STATUS                      PORTS     NAMES
50e45b242836   busybox   "echo 'Hola Mundo'"   24 seconds ago   Exited (0) 23 seconds ago             great_beaver
571f0b1cd767   busybox   "sh"                  23 minutes ago   Exited (0) 23 minutes ago             sweet_cerf
~

Son todos los contenedores que corrimos, tal que en la columna de status marca exit

## Ejecutando en modo interactivo
```
docker run -it busybox sh
```

/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 sh
    8 root      0:00 ps
/ # uptime
 16:25:39 up  1:00,  0 users,  load average: 0.06, 0.01, 0.00
/ # free
              total        used        free      shared  buff/cache   available
Mem:        2035400      431096      292308      374048     1311996     1082060
Swap:       1048572       52280      996292
/ # ls -l /
total 36
drwxr-xr-x    2 root     root         12288 Jun  7 17:34 bin
drwxr-xr-x    5 root     root           360 Aug 17 16:25 dev
drwxr-xr-x    1 root     root          4096 Aug 17 16:25 etc
drwxr-xr-x    2 nobody   nobody        4096 Jun  7 17:34 home
dr-xr-xr-x  184 root     root             0 Aug 17 16:25 proc
drwx------    1 root     root          4096 Aug 17 16:25 root
dr-xr-xr-x   13 root     root             0 Aug 17 16:25 sys
drwxrwxrwt    2 root     root          4096 Jun  7 17:34 tmp
drwxr-xr-x    3 root     root          4096 Jun  7 17:34 usr
drwxr-xr-x    4 root     root          4096 Jun  7 17:34 var
/ #
```

$ docker rm 571f0b1cd767
```

571f0b1cd767

```

docker rm $(docker ps -a -q -f status=exited)
docker container prune
```
## Montando volúmenes
?
## Publicando puertos
```
$ docker run -d daviey/nyan-cat-web
```
Unable to find image 'daviey/nyan-cat-web:latest' locally
latest: Pulling from daviey/nyan-cat-web
b7f33cc0b48e: Pull complete
5f9b58fd6dd4: Pull complete
1adeef8edfca: Pull complete
cc8a2986b124: Pull complete
7220539c61d6: Pull complete
Digest: sha256:57ac8fd383ada137e22a2894e92f74287f4566be0ae21ca97828b34a93a646c6
Status: Downloaded newer image for daviey/nyan-cat-web:latest
86562690efcdf00fc20478eb6738a66e6378643b8bcde4ba4295740df437568b
~
```
$ docker ps
```
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS             NAMES
86562690efcd   daviey/nyan-cat-web   "nginx -g 'daemon of…"   13 seconds ago   Up 12 seconds   80/tcp, 443/tcp   musing_raman
~
```
$ docker kill 86562690efcd
```
86562690efcd
~
```
$ docker rm 86562690efcd
```
86562690efcd
~
```
$ docker run -d -p 8000:80 daviey/nyan-cat-web
```
350a217e27b5ddbc725071ba1bf60ea34a8f38ca88517fffac7c28370e425287
~
Nos conectamos por el puerto 8000 y accedemos a http://localhost:8000 
## 9- Utilizando una base de datos
//donde se creo esa carpeta que es lo que tiene?
```
$ mkdir $HOME/.postgres
```

~

```
docker run --name my-postgres -e POSTGRES_PASSWORD=mysecretpassword -v $HOME/.postgres:/var/lib/postgresql/data -p 5432:5432 -d postgres:9.4
```

Unable to find image 'postgres:9.4' locally
9.4: Pulling from library/postgres
619014d83c02: Pull complete
7ec0fe6664f6: Pull complete
9ca7ba8f7764: Pull complete
9e1155d037e2: Pull complete
febcfb7f8870: Pull complete
8c78c79412b5: Pull complete
5a35744405c5: Pull complete
27717922e067: Pull complete
36f0c5255550: Pull complete
dbf0a396f422: Pull complete
ec4c06ea33e5: Pull complete
e8dd33eba6d1: Pull complete
51c81b3b2c20: Pull complete
2a03dd76f5d7: Pull complete
Digest: sha256:42a7a6a647a602efa9592edd1f56359800d079b93fa52c5d92244c58ac4a2ab9
Status: Downloaded newer image for postgres:9.4
ffdaa48594c295d6e0d2147edaae7bb3b356f6c44873300423982fab7d420793
~
```
docker exec -it my-postgres /bin/bash
```

root@ffdaa48594c2:/# psql -h localhost -U postgres
psql (9.4.26)
Type "help" for help.

postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)

postgres=# create database test;
CREATE DATABASE
postgres=# \connect test
You are now connected to database "test" as user "postgres".
test=# create table tabla_a (mensaje varchar(50));
CREATE TABLE
test=# insert into tabla_a (mensaje) values('Hola mundo!');
INSERT 0 1
test=# select * from tabla_a;
   mensaje
-------------
 Hola mundo!
(1 row)

test=#
test=# \q
root@ffdaa48594c2:/#
root@ffdaa48594c2:/# exit

La diferencia entre "docker run" y "docker exec" es que "docker exec" ejecuta un comando en un contenedor en ejecución. Por otro lado, "docker run" crea un contenedor temporal, ejecuta el comando en él y detiene el contenedor cuando termina.



