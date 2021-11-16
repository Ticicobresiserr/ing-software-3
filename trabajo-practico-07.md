## Trabajo Práctico 7 - Servidor de Build (de integración continua).
### 1- Objetivos de Aprendizaje
 - Adquirir conocimientos acerca de las herramientas de integración continua.
 - Configurar este tipo de herramientas.
 - Implementar procesos de construcción automatizado simples.

### 2- Unidad temática que incluye este trabajo práctico
Este trabajo práctico corresponde a la unidad Nº: 3 (Libro Continuous Delivery: Cap 3)

### 3- Consignas a desarrollar en el trabajo práctico:
 - Para una mejor evaluación del trabajo práctico, incluir capturas de pantalla de los pasos donde considere necesario.

### 4- Desarrollo:

#### 1- Poniendo en funcionamiento Jenkins
  - Bajar la aplicación y ejecutarla (ejemplo para Linux):
```bash
export JENKINS_HOME=~/jenkins

mkdir -p $JENKINS_HOME
cd $JENKINS_HOME

wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war

java -jar jenkins.war --httpPort=8081
```
  - Se puede también ejecutar en contenedor de Jenkins (pero para construir imágenes de Docker, el proceso se complica un poco):

```bash
# Windows

mkdir -p C:\jenkins
docker run -d -p 8081:8080 -p 50000:50000 -v C:\jenkins:/var/jenkins_home jenkins/jenkins:lts
```

```bash
# Linux / Mac OS

mkdir -p ~/jenkins
docker run -d -p 8081:8080 -p 50000:50000 -v ~/jenkins:/var/jenkins_home jenkins/jenkins:lts
```
  - Una vez en ejecución, abrir http://localhost:8081
  - Inicialmente deberá especificar el texto dentro del archivo ~/jenkins/secrets/initialAdminPassword
```bash
cat ~/jenkins/secrets/initialAdminPassword
```
  - Instalar los plugins por defecto

  - Crear el usuario admin inicial. Colocar cualquier valor que considere adecuado.

 - Se aconseja perisistir la variable **JENKINS_HOME**, ya sea por ejemplo en .bashrc o en las variables de entorno de Windows.
#### 2- Conceptos generales
  - Junto al Jefe de trabajos prácticos:
    - Explicamos los diferentes componentes que vemos en la página principal
    - Analizamos las opciones de administración de Jenkins

#### 3- Instalando Plugins y configurando herramientas
  - En Administrar Jenkins vamos a la sección de Administrar Plugins
  - De la lista de plugins disponibles instalamos **Docker Pipeline**
  - Instalamos sin reiniciar el servidor.
  - Abrir nuevamente página de Plugins y explorar la lista, para familiarizarse qué tipo de plugins hay disponibles.
  - En la sección de administración abrir la opción de configuración de herramientas
  - Agregar maven con el nombre de **M3** y que se instale automáticamente.

#### 4- Creando el primer Pipeline Job
  - Crear un nuevo item, del tipo Pipeline con nombre **hello-world**
  - Una vez creado el job, en la sección Pipeline seleccionamos **try sample Pipeline** y luego **Hello World**
  - Guardamos y ejecutamos el Job
  - Analizar la salida del mismo

**Resultado**

Este pipeline crea un stage llamado ‘Helo’ y adentro muestra como salida un mensaje de Hello World

![alt text here](https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/p1.png)


#### 5- Creando un Pipeline Job con Git y Maven
  - Similar al paso anterior creamos un ítem con el nombre **simple-maven**
  - Elegir **Git + Maven** en la sección **try sample Pipeline**
  - Guardar y ejecutar el Job
  - Analizar el script, para identificar los diferentes pasos definidos y correlacionarlos con lo que se ejecuta en el Job y se visualiza en la página del Job.

**REPO CON JENKINS FILE: https://github.com/Ticicobresiserr/springboot**

``` 
pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M3"
    }

    stages {
        stage('Build') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/jglick/simple-maven-project-with-tests.git'

                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true clean package"

                // To run Maven on a Windows agent, use
                // bat "mvn -Dmaven.test.failure.ignore=true clean package"
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    junit '**/target/surefire-reports/TEST-*.xml'
                    archiveArtifacts 'target/*.jar'
                }
            }
        }
    }
}
```

#### Dentro del pipeline:
- Se utiliza como herramienta maven 
- Se crea un stage de build: 
- se hace un checkout de un repo de github que tiene tests de maven
- se abre una consola y se corre un comando maven que hace un clean de posibles aritifacts que pueda tener, corre unos tests, y despues hace un package de unos tests convirtiendolos en formato JAR 
- se crea un post en donde si los tests corrieron de forma exitosa, se guardan los reportes de esos tests 
- se crea un artifact .jar para guardar el ejecutable, metadata

![alt text here](https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/p2.png)


#### 6- Utilizando nuestros proyectos
  - Utilizando lo aprendido en el ejercicio 5
    - Crear un Job que construya el proyecto **spring-boot** del [trabajo práctico 6](06-construccion-imagenes-docker.md).
    - Obtener el código desde el repositorio de cada alumno (se puede crear un repositorio nuevo en github que contenga solamente el proyecto maven).
    - Generar y publicar los artefactos que se producen.
  - Como resultado de este ejercicio proveer el script en un archivo **spring-boot/Jenkinsfile**

![alt text here](https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/p3.1.png)

![alt text here](https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/p3.2.png)

```
pipeline {

    agent any

    stages {
        stage('Build') {
            steps {

                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [[$class: 'CleanCheckout']], userRemoteConfigs: [[credentialsId: 'credenciales_git_2', url: 'https://github.com/Ticicobresiserr/springboot.git']]])
                sh "echo 'github 1'"
                sh "docker build ."
                // sh "mvn clean package springboot:repackage "

            }

        }
    }
}
```

#### 7- Utilizando nuestros proyectos con Docker
  - Extender el ejercicio 6
  - Generar y publicar en Dockerhub la imagen de docker ademas del Jar.
  - Se puede utilizar el [plugin de docker](https://docs.cloudbees.com/docs/admin-resources/latest/plugins/docker-workflow) o comandos de shell.
  - No poner usuario y password en el pipeline en texto plano, por ejemplo para conectarse a DockerHub, utilizar [credenciales de jenkins](https://github.com/jenkinsci/credentials-plugin/blob/master/docs/user.adoc) en su lugar.
  - Como resultado de este ejercicio proveer el script en un archivo **spring-boot/Jenkinsfile**
  - Referencia: https://tutorials.releaseworksacademy.com/learn/building-your-first-docker-image-with-jenkins-2-guide-for-developers


![alt text here](https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/p41.png)

```
pipeline {
    
    environment { 
        registryCredential = 'dockerhub_id' 
        dockerImage = "ticicobresiserr/spring-boot:"
        ver= ''
        
    }

    agent any

    stages {
        stage('Build') {
            steps {

                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [[$class: 'CleanCheckout']], userRemoteConfigs: [[credentialsId: 'credenciales_git_2', url: 'https://github.com/Ticicobresiserr/springboot.git']]])
                sh "echo 'github 1'"
                sh "docker build -t test-spring-boot ."
                // sh "mvn clean package springboot:repackage "

            }

        }
        
        stage('Publish') { 
            steps { 
                script { 
                    ver = "v1.0." + "$BUILD_NUMBER" 
                    sh "echo $ver"
                    dockerImage = dockerImage + ver
                    
                    docker.withRegistry( '', registryCredential ){
                        sh "docker tag test-spring-boot:latest $dockerImage"
                        
                        sh "docker push $dockerImage"
                    }
                } 
                
            
            }
        }
    }
}
```

![alt text here](https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/p3DH.png)


### clases
Se descarga el codigo de github, el servidor de biuld lo compila en un entorno controlado y lo distribuye.

Jenkins esta escrita en Java, 


Separamos responsabilidades: 1 imagen en tiempo de build y la segunda para runtime

En el caso de Java dejamos los jdk, maven, todos los residuos de la compilacion en la primera imagen
En la segunda imagen usamos jre(mas liviano que jdk que contiene a ambos) y solo el .jar que sera una imagen mucho mas limpia, mucho mas chica. 

Al primer from se le asigna como un id y. queda cacheado para luego ser tomado por la segunda imagen. Esa primera imagen esta cacheada asi que si cambiamos algo en la imagen de producion no se tiene que cargar de nuevo


En el caso de nodejs
Por mas que usemos para los dos from la misma imagen, 
Alpine se usa para compilar normalmente 
Si usamos la imagen de nginex, copia la salida del build y lo copia en el servidor de nginex y el que sirve la aplicación no es node start sino que es ngnex

 


Con el segundo FROM 
Se hace un run npm install –onl;y=production y solo me instala los paquetes necesarios para produccion, no todo el resto de paquetes y librerias necesarias para la compilacion

Si es una aplicación estatica sin backend es mejor usar nginex. Sirve para paginaas estaticas con HTML CSS, entonces solo me hace faaltaa servir la aplicación en el browser

Si tengo que ejecutar un backend, es decir que falta mas logica para interactuar con backend (conectarme a una bd) en mi aplicaacion, puedo hacer como en la foto, es una buena practica

