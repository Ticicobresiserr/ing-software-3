# Trabajo Práctico 5 - Herramientas de construcción de software

### 4- Desarrollo:

#### 1- Instalar Java JDK si no dispone del mismo. 
  - Java 8 es suficiente, pero puede utilizar cualquier versión.
  - Utilizar el instalador que corresponda a su sistema operativo 
  - http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
  - Agregar la variable de entorno JAVA_HOME
    - En Windows temporalmente se puede configurar
    ```bash
      SET JAVA_HOME=C:\Program Files\Java\jdk1.8.0_221
    ```
    - O permanentemente entrando a **Variables de Entorno** (Winkey + Pausa -> Opciones Avanzadas de Sistema -> Variables de Entorno)
  - Otros sistemas operativos:
    - https://www3.ntu.edu.sg/home/ehchua/programming/howto/JDK_Howto.html
    - https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-18-04


#### 2- Instalar Maven
- Instalar maven desde https://maven.apache.org/download.cgi (última versión disponible 3.6.1)
- Descomprimir en una carpeta, por ejemplo C:\tools
- Agregar el siguiente directorio a la variable de entorno PATH, asumiendo que los binarios de ant están en C:\tools\apache-maven-3.6.1\bin

  ```bash   
    SET PATH=%PATH%;C:\tools\apache-maven-3.6.1\bin
  ```  
- Se puede modificar permanentemente la variable PATH entrando a (Winkey + Pausa -> Opciones Avanzadas de Sistema -> Variables de Entorno)
- En Linux/Mac se puede agregar la siguiente entrada al archivo ~/.bash_profile

  ```bash
  export PATH=/opt/apache-maven-3.6.1/bin:$PATH
  ```

#### 3- Introducción a Maven
### Qué es Maven?

Maven es una herramienta de software para la gestión y construcción de proyectos Java, capaz de gestionar un proyecto software completo, desde la etapa en la que se comprueba que el código es correcto, hasta que se despliega la aplicación, pasando por la generación de informes y documentación. Incluyendo el empaquetado y la instalación de mecanismos de distribución de librerías, para que puedan ser utilizadas por otros desarrolladores y equipos de desarrollo. También contempla temas relacionados con la integración continua, para poder realizar la ejecución de test unitarios y pruebas automatizadas, test de integración, etc.

### Qué es el archivo POM?
    1. modelVersion
    2. groupId
    3. artifactId
    4. versionId

Maven utiliza un Project Object Model (POM) para describir el proyecto de software a construir, sus dependencias de otros módulos y componentes externos, y el orden de construcción de los elementos. Tiene un modelo de configuración de construcción más simple, basado en un formato XML
Este archivo se utiliza para administrar: código fuente, archivos de configuración, información y roles del desarrollador, sistema de seguimiento de problemas, información de la organización, autorización del proyecto, URL del proyecto, dependencias del proyecto, etc.

  - modelVersion: Declara qué versión del modelo POM sigue el descriptor del proyecto. Esto es para garantizar la estabilidad cuando Maven introduce nuevas funciones u otros cambios de modelo.
  - groupId: El logo único de la empresa u organización, y la ruta generada durante la configuración también se genera a partir de este, Por ejemplo, com.winner.trade, maven colocará el paquete jar del proyecto en la ruta local: / com / winner / trade
  - artifactId: El ID único de este proyecto. Puede haber varios proyectos bajo un groupId, que se distinguen por artifactId. The primary artifact for a project is typically a JAR file. A typical artifact produced by Maven would have the form <artifactId>-<version>.<extension> (for example, myapp-1.0.jar).
  - version: El número de versión actual de este proyecto
  - packaging: Tipo de paquete, valores posibles: pom, jar, maven-plugin, ejb, war, ear, rar, par, etc.
  - Name: El nombre del proyecto, utilizado por la documentación generada por Maven, se puede omitir
  - url: La URL de la página de inicio del proyecto, utilizada por los documentos de Maven, se puede omitir

### Repositorios Local, Central y Remotos http://maven.apache.org/guides/introduction/introduction-to-repositories.html

Los repositorios de Maven son directorios físicos que contienen archivos JAR empaquetados junto con metadatos adicionales sobre estos archivos jar. Estos metadatos están en forma de archivos POM que tienen información del proyecto del archivo jar, incluidas las otras dependencias externas que tiene este archivo JAR. Estas otras dependencias externas se descargan de forma transitiva en su proyecto y se convierten en parte del pom efectivo del proyecto. 

Hay exactamente dos tipos de repositorios: locales y remotos:

El repositorio local: de Maven reside en la máquina del desarrollador. Siempre que ejecute objetivos de maven que requieran estas dependencias, maven descargará las dependencias de servidores remotos y las almacenará en la máquina del desarrollador.De forma predeterminada, Maven crea el repositorio local dentro del directorio de inicio del usuario, es decir, el directorio C: /Users/superdev/.m2. 

Tener almacenadas las dependencias en la máquina local tiene dos beneficios principales. En primer lugar, varios proyectos pueden acceder al mismo artefacto, por lo que se reduce la necesidad de almacenamiento. En segundo lugar, como la dependencia se descarga solo una vez, también reduce el uso de la red.

Repositorio central: El repositorio central de Maven se encuentra en http://repo.maven.apache.org/maven2/. Siempre que ejecute el trabajo de compilación, maven primero intente encontrar la dependencia del repositorio local. Si no está allí, entonces, de forma predeterminada, maven activará la descarga desde esta ubicación de repositorio central.


Repositorio remoto: Además del repositorio central, es posible que necesiten artefactos implementados en otras ubicaciones remotas. Por ejemplo, en su oficina corporativa puede haber proyectos o módulos específicos de la organización únicamente. En estos casos, la organización puede crear un repositorio remoto e implementar estos artefactos privados. Este repositorio remoto será accesible solo dentro de la organización.
Estos repositorios remotos de maven funcionan exactamente de la misma manera que el repositorio central de maven. Siempre que se necesita un artefacto de estos repositorios, primero se descarga al repositorio local del desarrollador y luego se utiliza.

### Entender Ciclos de vida de build
  - default
  - clean
  - site
  - Referencia: http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Build_Lifecycle_Basics

Hay tres ciclos de vida de compilación integrados: default, clean, site. El ciclo de vida default maneja la implementación de su proyecto, el ciclo de vida clean maneja la limpieza del proyecto, mientras que el ciclo de vida del site maneja la creación del sitio web de su proyecto.

### Comprender las fases de un ciclo de vida, por ejemplo, default:

| Fase de build | Descripción                                                                                                                            |
|---------------|----------------------------------------------------------------------------------------------------------------------------------------|
| validate      | valida si el proyecto está correcto y toda la información está disponible                                                             |
| compile       | compila el código fuente del proyecto                                                                                 |
| test          | prueba el código fuente compilado utilizando un marco de prueba de unidad adecuado. Estas pruebas no deberían requerir que el código se empaquete o implemente |
| package       | toma el código compilado y lo empaqueta en su formato distribuible, como un JAR.                                                     |
| verify        | ejecuta cualquier verificación de los resultados de las pruebas de integración para garantizar que se cumplan los criterios de calidad                                                      |
| install       | instal1 el paquete en el repositorio local, para usarlo como dependencia en otros proyectos localmente                                       |
| deploy        | hecho en el entorno de compilación, copia el paquete final en el repositorio remoto para compartirlo con otros desarrolladores y proyectos.      |

validate >> compile >> test (optional) >> package >> verify >> install >> deploy

- Copiar el siguiente contenido a un archivo, por ejemplo ./trabajo-practico-02/maven/vacio/pom.xml

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>ar.edu.ucc</groupId>
    <artifactId>proyecto-01</artifactId>
    <version>0.1-SNAPSHOT</version>
</project>
```

- Ejecutar el siguiente comando en el directorio donde se encuentra el archivo pom.xml

```
mvn clean install
```
validate >> compile (src)>> test (optional) >> package >> verify >> install

#### Sacar conclusiones del resultado

El comando lo que hace es limpia los artefactos creados por compilaciones anteriores(target) e instala el paquete en el repositorio local, para usarlo como una dependencia en otros proyectos localmente.
Lo que ocurre es que se empieza a builder el proyecto-01 version 0.1-SNAPSHOT, a partir del archivo pom, para lo cual comienza a descargar dependencias del repositorio central y se descargan en local, por ejemplo para hacer los tests. Finalmente se crea una carpeta target/maeven-archiver , target/proyecto-01-0.1-SNAPSHOT.jar. Hace el proceso de validate, compile, test, package, etc.


#### 4- Maven Continuación

- Generar un proyecto con una estructura inicial:

```bash
mvn archetype:generate -DgroupId=ar.edu.ucc -DartifactId=ejemplo -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

- Analizar la estructura de directorios generada:

```
.
└── ejemplo
    ├── pom.xml
    └── src
        ├── main
        │   └── java
        │       └── ar
        │           └── edu
        │               └── ucc
        │                   └── App.java
        └── test
            └── java
                └── ar
                    └── edu
                        └── ucc
                            └── AppTest.java

12 directories, 3 files
```

#### Compilar el proyecto
mvn compile – compila el proyecto y deja el resultado en target/classes

#### Analizar la salida del comando anterior y luego ejecutar el programa
```bash
mvn clean package
```
El comando toma el código compilado y lo empaqueta. En este caso como pom dice que el proyecto es un jar, creará un jar y lo pone en algún lugar del directorio de destino (por defecto). Entonces cuando corremos el comando, el ciclo de vida que seguimos es:

validate >> compile >> test (optional) >> package

```
java -cp target/ejemplo-1.0-SNAPSHOT.jar ar.edu.ucc.App
```
java -cp target/ejemplo-1.0-SNAPSHOT.jar ar.edu.ucc.App
Hello World!

#### 6- Manejo de dependencias

- Crear un nuevo proyecto con artifactId **ejemplo-uber-jar**

mvn archetype:generate -DgroupId=ar.edu.ucc -DartifactId=ejemplo-uber-jar -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

- Modificar el código de App.java para agregar utilizar una librería de logging:

```java
package ar.edu.ucc;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Hello world!
 *
 */
public class App 
{
    public static void main( String[] args )
    {
        Logger log = LoggerFactory.getLogger(App.class);
        log.info("Hola Mundo!");
    }
}
```

- Compilar el código e identificar el problema.
En vez de imprimir por pantalla quiero usar un Logger. El error dice que no se puede hacer el impor de import org.slf4j.Logger; import org.slf4j.LoggerFactory porque no puede encontrar los paquetes o dependencias. Esto es porque nunca se definieron esas dependencias en el pom.xml, que es necesario para que puedan ser usadas en la app.java

- Agregar la dependencia necesaria al pom.xml

```xml
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>1.2.1</version>
    </dependency>
```
A nivel de complicacion, estas dependencias sirven para que el codigo de la app.java cree las clases pero falta la parte de decir agregame estas clases en el .jar. Entonces en este caso, esta dependencia que agregamos, me permite compilarlo pero a la hora de ejecutarlo, no sabe a donde ir a busccar, resolver esas clases. Le tengo que decir quiero estas dependencias y ponelas en mi ejecutable final (en mi .jar)
	
- Verificar si se genera el archivo jar y ejecutarlo
mvn compile
mvn clean package

```bash
java -cp target/ejemplo-uber-jar-1.0-SNAPSHOT.jar ar.edu.ucc.App
```

- Sacar conclusiones y analizar posibles soluciones
```
Exception in thread "main" java.lang.NoClassDefFoundError: org/slf4j/LoggerFactory
	at ar.edu.ucc.App.main(App.java:14)
Caused by: java.lang.ClassNotFoundException: org.slf4j.LoggerFactory
	at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:641)
	at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:188)
	at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:520)
	... 1 more
```
El problema es que le puse las dependencias, lo compile y lo empaquete, pero en la ejecucion no sabe donde esta la clase. (slf4j). Entonces yo deberia de introducir en el comando “java -cp target/ejemplo-uber-jar-1.0-SNAPSHOT.jar ar.edu.ucc.App” el classpath de la clase para que la aplicación pueda encontrarla.
El profe hizo algo asi para hacerlo de forma manual:
	![alt text here]( https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/Pic.png)

- Implementar la opción de uber-jar: https://maven.apache.org/plugins/maven-shade-plugin/

```xml
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>2.0</version>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>shade</goal>
            </goals>
            <configuration>
              <finalName>${project.artifactId}</finalName>
              <transformers>
                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                  <mainClass>ar.edu.ucc.App</mainClass>
                </transformer>
              </transformers>
              <minimizeJar>false</minimizeJar>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
```
- Volver a generar la salida y probar ejecutando
mvn compile
mvn clean package

```bash
java -jar target\ejemplo-uber-jar.jar
```
#### Salida

$ java -jar target/ejemplo-uber-jar.jar
14:45:41.077 [main] INFO ar.edu.ucc.App - Hola Mundo!

Este plug in basicamente navega por todos los lugares posibles donde pueda encontrar dependencias y va a generar un jar (ejemplo-uber-jar.jar), mucho mas pesado que el original que no funcionaba (ejemplo-uber-jar-1.0-SNAPSHOT.jar)

Este complemento proporciona la capacidad de empaquetar el artefacto en un uber-jar, incluidas sus dependencias y sombrear, es decir, cambiar el nombre de los paquetes de algunas de las dependencias.
Normalmente en Maven, dependemos de la gestión de dependencias. Un artefacto contiene solo las clases/recursos de si mismo. Maven será responsable de averiguar todos los artefactos (JAR, etc.) que el proyecto dependerá de cuándo se construya el proyecto.
Un uber-jar es algo que toma todas las dependencias, extrae el contenido de las dependencias y las coloca con las clases/recursos del proyecto en sí, en un gran JAR. Al tener tal uber-jar, es fácil de ejecutar, ya que solo necesitarás un gran JAR en lugar de toneladas de pequeños JAR para ejecutar tu aplicación. También facilita la distribución en algunos casos.

#### 7- Utilizar una IDE
  - Importar el proyecto anterior en Eclipse o Intellij como maven project:
    - Si no dispone de Eclipse puede obtenerlo desde este link http://www.eclipse.org/downloads/packages/release/2018-09/r/eclipse-ide-java-ee-developers
    - Para importar, ir al menú Archivo -> Importar -> Maven -> Proyecto Maven Existente:
![alt text](./import-existing-maven.png)
    - Seleccionar el directorio donde se encuentra el pom.xml que se generó en el punto anterior. Luego continuar:
![alt text](./path-to-pom.png)

  - Familiarizarse con la interfaz grafica
#### Ejecutar la aplicación

![alt text here]( https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/Picture1.png)

#### Depurar la aplicación
![alt text here]( https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/Picture2.png)

#### Correr unit tests y coverage
![alt text here]( https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/Picture3.png)

![alt text here]( https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/Picture4.png)

#### Ejecutar los goals de maven
- Maven clean
- Maven install
- Maven test
- Maven build
- Maven generte-resources
    #### Encontrar donde se puede cambiar la configuración de Maven.
Eclipse/Preferences/Maven
    - etc.

#### 8- Ejemplo con nodejs

- Instalar Nodejs: https://nodejs.org/en/

- Instalar el componente para generar aplicaciones Express

```bash
npm install express-generator -g
```

- Crear una nueva aplicación
```bash
express --view=ejs hola-mundo
```

- Ejecutar la aplicación

```bash
cd hola-mundo
npm install
npm start
```

- La aplicación web estará disponible en http://localhost:3000

- Analizar el manejo de paquetes y dependencias realizado por npm.
#### package.json & package-lock.json

El package.json se usa para más que dependencias, como definir propiedades del proyecto, descripción, información de autor y licencia, scripts, etc. El package-lock.json se usa únicamente para bloquear dependencias a un número de versión específico. registra la versión exacta de cada paquete instalado lo que permite reinstalarlos. Las instalaciones futuras podrán construir un árbol de dependencias idéntico.

#### 9- Ejemplo con python
- Instalar dependencias (Ejemplo Ubuntu) varía según el OS:
```
sudo apt install build-essential python3-dev
pip3 install cookiecutter
```
- Correr el scaffold
```bash
$ cookiecutter https://github.com/candidtim/cookiecutter-flask-minimal.git
application_name [Your Application]: test
package_name [yourapplication]: test
$
```
- Ejecutar la aplicación
```bash
cd test
make run
```
- Acceder a la aplicación en: http://localhost:5000/
- Explicar que hace una tool como cookiecutter, make y pip.

Cookiecutter es un módulo para pip que creará el layout de un proyecto Django incluyendo:
- Todos sus directorios, Ficheros necesarios, Carpetas, Módulos, Configuraciones de usuario, Configuraciones de aplicación, Dependencias, Opciones de despliegue, Entornos de testing y módulos para debug, Calidad de código, Envío de emails para local y producción, Incluye compiladores de Javascript, Opciones para trabajar con contenedores.
- Además, dentro de cada archivo ya sea Python, HTML, JavaScript, CSS... se crearán las plantillas iniciales para empezar a programar cuanto antes, minimizando el tiempo invertido en arrancar un proyecto.

El propósito de la utilidad make es determinar automáticamente qué partes de un programa grande necesitan ser recompiladas y emitir los comandos necesarios para recompilarlas. Se puede usar make con cualquier lenguaje de programación cuyo compilador se pueda ejecutar con un comando de shell. De hecho, make no se limita a los programas. Se puede usarlo para describir cualquier tarea en la que algunos archivos deban actualizarse automáticamente desde otros siempre que estos cambien.
Para prepararse para usar make, debe escribir un archivo llamado makefile que describa las relaciones entre los archivos en su programa y los estados de los comandos para actualizar cada archivo.
Pe, make decide si es necesario volver a generar un objetivo comparando los tiempos de modificación del archivo. Esto resuelve el problema de evitar la creación de archivos que ya están actualizados.
	Profe: usamos make cuando queremos compilar y linkear librerias dinamicas o estaticas y escibiendo una receta en make se hace mucho mas facil

#### 10- Build tools para otros lenguajes
- Hacer una lista de herramientas de build (una o varias) para distintos lenguajes, por ejemplo (Rust -> cargo)
La mayoría de los lenguajes de programación proporcionan un administrador de paquetes para una fácil instalación. 
![alt text here]( https://github.com/Ticicobresiserr/ing-software-3/blob/main/screen/Picture5.png)

### PIP
pip es un sistema de administración de paquetes escrito en Python que se usa para instalar y administrar paquetes de software. Se conecta a un repositorio en línea de paquetes públicos, llamado Python Package Index.

#### Conda
Conda es un sistema de gestión de paquetes de código abierto y un sistema de gestión del entorno. Conda instala, ejecuta y actualiza rápidamente paquetes y sus dependencias. Conda crea, guarda, carga y cambia fácilmente entre entornos en su computadora local. Fue creado para programas Python, pero puede empaquetar y distribuir software para cualquier idioma.
Conda también se incluye en Anaconda Enterprise, que proporciona gestión de entornos y paquetes empresariales en el sitio para Python, R, Node.js, Java y otras pilas de aplicaciones. 

#### Composer
Composer es una herramienta para la gestión de dependencias en PHP. Le permite declarar las bibliotecas de las que depende su proyecto y las administrará (instalará / actualizará) por usted.
Composer no es un administrador de paquetes en el mismo sentido que Yum o Apt. Sí, trata con "paquetes" o bibliotecas, pero los administra por proyecto, instalándolos en un directorio (por ejemplo, proveedor) dentro de su proyecto. De forma predeterminada, no instala nada globalmente. Por tanto, es un gestor de dependencias. 

#### Yarn
YARN es un gestor dependencias de JavaScript, que está enfocado en la velocidad y la seguridad, y a diferencia de otros gestores como NPM, YARN es muy rápido y muy fácil de usar.
 En Node las dependencias se colocan dentro de un directorio  llamado node_modules de tu proyecto. Lo que pasaba es que la estructura puede diferir del árbol de dependencias real, pues las dependencias duplicadas se fusionan.
npm instala las dependencias de forma no determinista, y la carpeta node_modules podría ser diferente de una instalación a otra y causar errores
Por eso nació Yarn, un sistema que resuelve los problemas relacionados con el control de versiones usando archivos de bloqueo y un algoritmo de instalación determinista y confiable, asegurando  la misma estructura de archivos en node_modules en todas las máquinas.

#### npm
NPM son las siglas de Node Package Manager. Es lo que describe su nombre. Es un administrador de paquetes para entornos basados en nodos. Realiza un seguimiento de todos los paquetes y sus versiones y permite al desarrollador actualizar o eliminar fácilmente estas dependencias. Todas estas dependencias externas se almacenan dentro de un archivo llamado package.json.

#### Grandle
Gradle es una herramienta de automatización moderna que se utiliza en el desarrollo de software para la automatización de la construcción de proyectos. Gradle te permite especificar la compilación del proyecto (juntando el código fuente, enlazando bibliotecas, etc.), y luego, cada vez que realizas un cambio, puedes simplemente "presionar el botón" y Gradle automáticamente completa todos los pasos por ti.

Gradle se usa a menudo en el desarrollo de aplicaciones JVM, escritas en lenguajes como Java o Kotlin. Sus competidores directos son Maven y Ant.

#### RubyGems
RubyGems es un gestor de paquetes para el lenguaje de programación Ruby que proporciona un formato estándar y autocontenido para poder distribuir programas o bibliotecas en Ruby, una herramienta destinada a gestionar la instalación de estos, y un servidor para su distribución. 

#### Homebrew
Homebrew es un sistema de gestión de paquetes que simplifica la instalación, actualización y eliminación de programas en los sistemas operativos Mac OS de Apple y GNU/Linux.

#### NuGet
NuGet es el administrador de paquetes para .NET. Las herramientas de cliente de NuGet brindan la capacidad de producir y consumir paquetes. La Galería NuGet es el repositorio central de paquetes que usan todos los autores y consumidores de paquetes.

#### Gulp 
Gulp es un manejador de tareas(Task manager) una incredible herramienta para todo desarrollador que permite de forma automática, gestionar toda clases de tareas comunes y a la ves tediosas en el desarrollo de cualquier aplicación… (Mover archivos de una carpeta a otra, eliminar archivos que se crean de forma automática en algunas instalaciones, minificar tu css o js, compilar tus archivos .less o .sass de manera manual pues gulp no solo te permite solucionar estos problemas sino que de igual manera te provee de herramientas que te permiten hacer cosas increíbles como sincronizar el navegador cuando modificas tu código para que se vean tus cambios de manera automática, validar sintaxis para encontrar mas fácilmente tus errores y un largo etcétera de de ventajas plugins y tareas que podemos ejecutar de forma automática desde esta increíble herramienta.

#### ppm 
El Administrador de paquetes de Perl (PPM) proporciona una interfaz de línea de comandos para administrar módulos y extensiones de Perl (paquetes). PPM le permite acceder a los repositorios de paquetes, instalar y eliminar paquetes de su sistema y actualizar los paquetes que instaló previamente usando PPM con las últimas versiones.

https://devopedia.org/package-manager
Que es un manejador de tareas vs manejador de paquetes?

- Elegir al menos 10 lenguajes de la lista de top 20 o top 50 de tiobe: https://www.tiobe.com/tiobe-index/

#### 11- Presentación

- Subir todo el código, ejemplos y respuestas a una carpeta trabajo-practico-05.

> Tip: Agregar un archivo .gitignore al repositorio para evitar que se agreguen archivos que son resultado de la compilación u otros binarios, que no son necesarios, al mismo.

### clase
En continue development, si yo mergeo algo eso se va directo a produccion, porque esto es cuando se supone que ya tengo test de unidad, test de calidad, etc, fue suficiente, el software paso por todos los filtros y se sube a produccion. Tenes que tener muchos test paara que el usuario no se encuentre con errores al final

En continue delivery, hay alguien que aprieta un boton y lo sube a produccion. Entonces puedo llevarlo a un entorno de staging pero alguien lo aprueba en el medio 

Para construir codigo:
Tienen manejadores de paquetes/dependencias

#### Profe: Grupnt, gulp, webpack
Cuando estamos en un navegador web, si hacemos un inspeccionar/source de esa pagina vamos a ver miles de carpetas que son las que se deberian "descargar" (de 10Megas pe) para que el cliente pueda usar el navegador (si el navegador no me guardo en cache los ingresos). Entonces esas carpetas se comprimprimen al maximo posible, sin espacios por ejemplo para que esos 10mb puedan reducirse. Estas herramientas funcionan para estas cosas



