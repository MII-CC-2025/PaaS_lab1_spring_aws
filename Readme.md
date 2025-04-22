# PaaS con AWS Elastic Beanstalk

Desarrollaremos una Aplicación con Spring


## Requisitos

Java 21:

- Eliminar versiones instaladas (supongamos la 11):

```sh
$ sudo apt-get autoremove openjdk-11-jre-headless

$ sudo apt-get autoremove openjdk-11-jdk-headless

$ sudo apt-get purge openjdk-11-*
```

- Instalar versión 21:

```sh
$ sudo apt update -y
$ sudo apt upgrade -y
$ sudo apt install openjdk-21-jdk -y
$ java --version
openjdk 21.0.6 2025-01-21
OpenJDK Runtime Environment (build 21.0.6+7-Ubuntu-122.04.1)
OpenJDK 64-Bit Server VM (build 21.0.6+7-Ubuntu-122.04.1, mixed mode, sharing)
$ sudo update-alternatives --config java
```


## Crear Aplicación

Vamos a crear una aplicación Spring Boot usando Spring Initializr

- Utiliza Spring Initializr: https://start.spring.io/

    - Project: Maven
    - Language: Java, versión 21
    - Spring Boot: 3.4.4
    - Group: cc.paas.aws
    - Artifact: holamundo
    - Packaging: Jar
    - Dependencias: Spring Web y Thymeleaf

- Genera el proyecto, guárdalo en local y descomprime en tu IDE (si usas AWS Cloud9, tendrás que subirlo desde local)

## Crear un controlador y una vista

Controlador: Fichero HomeController.java en el directorio src/main/java/cc/paas/aws/holamundo

```
package cc.paas.aws.holamundo.controllers;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;


@Controller
public class HomeController {
    
    @GetMapping("/")
	public String index() {
	    
		return "index";
	}
    
}
```
Vista: fichero index.html en src/main/resource/templates

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

    <head>
        <title>Hola Mundo</title>
    </head>
    
    <body>
        <h1>Hola Mundo!!</h1>
        <p>versión: 0.1</p>
	
	</body>
</html>

```

## Construye el proyecto

- Compila el proyecto para generar el paquete .jar

```
./mvnw clean package
```

- Comprueba la aplicación de forma local
 
```
./mvnw spring-boot:run

```

Una vez iniciada, clic en Preview - Preview Running Application y aparecerá una ventana con la aplicación.

Si haces clic en la URL, puedes copiarla y pegarla en el mismo navegador donde uses AWS Cloud9, desde otro navegador no será accesible, 
a menos que lo configures para ello.

La URL será similar a la siguiente: https://d5fb5be6358943a8922adb524bb8819f.vfs.cloud9.us-east-1.amazonaws.com


## Despliega a producción 

- Cambia el puerto de la aplicación al 5000

AWS Beanstalk utiliza NGINX como servidor web que, por defecto, delegará las peticiones hacía el puerto 5000: por lo que cambiaremos
el puerto de nuestra aplicación. Para ello, en el fichero application.properties incluye la línea:

```
server.port=5000
```
Vuelve a construir el proyecto, para generar el fichero .jar.

```
./mvnw clean package
```

El fichero .jar de la aplicación se localiza en el directorio target, descarga este fichero a tu equipo local.

- En la consola Web de AWS accede a Elastic Beanstalk

- Crear Aplicación

    - Nivel de entorno: Entorno de servidor web
    - Nombre de aplicación: (el que desees, por ejemplo HolaMundo)
    - Nombre del entorno: (se autogenera, pero puedes cambiarlo, será el nombre la MV EC2 que se cree)
    - Dominio: (se autogenera, pero si quieres puedes cambiarlo, comprobando siempre la disponibilidad)
    - Plataforma: Java
    - Versión (Ramificación de la plataforma): Corretto 21 Amazon Linux 203
    - Código de aplicación: selecciona "Cargar el código" y establece la "Etiqueta de versión" (por ejemplo, ver1.0)
    - Marca "Archivo local", haz clic en el botón "Elegir archivo" y selecciona el fichero .jar descargado en el apartado anterior.
    - En "Valores preestablecidos" (Elementos preestablecidos de configuración): Instancia única (compatible con la capa gratuita)
    
    Clic en Siguiente.
    
- En "Acceso al Setvicio" establece la configuración propia de AWS Academy

    - Acceso al servicio: selecciona Usar un rol de servicio existente
    - En el desplegable "Roles de servicio existentes" elige LabRole
    - En "Par de claves de EC2" elige vockey
    - En "Perfil de instancia de EC2" elige LabInstanceProfile

    Clic en Siguiente.
    
    El resto de opciones podemos dejarlas por defecto, por lo tanto, en "Configuración de redes, bases de datos y etiquetas" pulsa en siguiente;
    en "Configuración del escalado y del tráfico de instancias" pulsa siguiente; en "Configuración de actualizaciones, monitoreo y registros" pulsa siguiente
    
    Finalmente, en la pantalla de revisión, comprueba que todo es correcto y pulsa en enviar. 
    El proceso pasa por diferentes estados: creará e iniciará el entorno en una MV EC2, 
    creará un Bucket S3 para almacenar las versiones de la aplicación
    y desplegará la aplicación; por lo tanto, habrá que esperar que el estado esté en OK (verde).

- Comprueba la app: http://holamundo-env.eba-jpzdcpsd.us-east-1.elasticbeanstalk.com/

    Si todo ha ido bien, podrás acceder a la aplicación en una URL similar a la anterior que verás en el campo dominio de la información general del entorno.
    
Recursos creados:

- Máquina Virtual: se elimina con el entorno

- Bucket S3: Con las versiones de la App, no se elimina. Para eliminarlo es necesario vaciarlo, borrar la política y eliminar el Bucket.


## Elastic Beanstalk CLI 

https://github.com/aws/aws-elastic-beanstalk-cli-setup

pip install awsebcli --upgrade --user

