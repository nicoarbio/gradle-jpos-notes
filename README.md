# Notas sobre Gradle para framework jPOSv2

🐜 Apache Ant<br>
Ant es utilizado en jPOSv1 y es una herramienta que usada para estructurar y compilar o buildear proyectos JAVA.
En su archivo más importante `build.xml` encontraremos como se describe el proceso de generación de ejecutable y sus dependencias.

🐘 Gradle<br>
Gradle es utilizado en jPOSv2 y es una herramienta open-source y multiplataforma, utilizada para estructurar y automatizar y estandarizar la construcción de proyectos de desarrollo en JAVA (y Kotlin, para Android). Esta está construida sobre los conceptos de Apache Ant y Apache Maven, y utiliza un lenguaje específico (DSL) basado en Groovy para definir configuraciones y gestionar dependencias.

## Archivo build.gradle
Este archivo es el principal en gradle, está ubicado en el directorio raíz del proyecto y entre sus caracteristicas más importantes podemos destacar que:
- Se escriben los scripts del proyecto como para generar y limpiar el directorio de compilación, así como para también generar archivos ejecutables.
- Se <u>aplican plugins</u>
- Se <u>definen dependencias</u> que se aplican a todos los módulos del proyecto.

Como ejemplo en jPOS podemos ver el archivo [build.gradle](https://github.com/jpos/jPOS-template/blob/master/build.gradle) del proyecto jPOS-template utilizado para implementar jPOS en nuevos proyectos.

## Archivo settings.gradle
Este archivo, al igual que build.gradle, se ubica en el directorio raíz del proyecto, define la configuración del repositorio a nivel de proyecto y le indica a Gradle qué módulos debe incluir cuando compila la app.

Ejemplo: [settings.gradle](https://github.com/jpos/jPOS-EE/blob/master/settings.gradle) del proyecto jPOS-EE.

## Gradle Plugins
Gradle por si solo no proveé mucha automatización, por lo que los plugins en Gradle son extensiones que nos permiten, por ejemplo, compilar codigo Java y nos facilitan (mucho) el armado de los scripts específicos del proyecto.

Los plugins añaden nuevas tasks, domain objects y convenciones (por ejemplo para el directorio src/main/java). Por ende, permiten extender el modelo de Gradle, configurar el proyecto de acuerdo a convenciones o aplicar configuraciones especificas (como agregar repositorios de la organizacion o hacer cumplir normas)

### Tipos de plugins
Referencia: [Gradle Plugins](https://docs.gradle.org/current/userguide/plugins.html)
- <u>Plugins de scripts</u> es código que encontraremos en un archivo `plugin-script.gradle`, que permiten agregar más configuraciones, funciones y tareas. Como ejemplo en jPOS podemos ver en el proyecto template, los archivos [build.gradle](https://github.com/jpos/jPOS-template/blob/master/build.gradle) y [jpos-app.gradle](https://github.com/jpos/jPOS-template/blob/master/jpos-app.gradle). Aquí se puede observar como en el archivo `build.gradle` simplemente invocando `apply from: 'jpos-app.gradle'`, se importan los scripts que existen en el archivo `jpos-app.gradle`. Por lo general, se usan dentro del building, aunque se pueden externalizar y acceder desde una ubicación remota.
- <u>Plugins binarios</u> son librerías que se aplican por id. Y se dividen entre:
    - Core de Gradle: Se proveen nombres cortos como los que veremos luego: `java`, `maven-publish`, `war`, etc.
    - Plugins comunitarios: Se deben llamar desde su nombre completo, por ejemplo `com.github.foo.bar`.

### Como aplicar un Plugin
Los plugins DSL (domain-specific lenguage) proveen una forma facil de declararse. Funciona con el portal de complementos de Gradle para proporcionar un fácil acceso a los complementos core y comunitarios.

Para aplicar un plugin core:
```
plugins {
    id 'java'
}
```

Para aplicar un plugin comunitario desde el portal, utilizando el id completo:
```
plugins {
    id 'com.jfrog.bintray' version '1.8.5'
}
```

Generalmente, un plugin comienza como un plugin de scripts (porque son fáciles de escribir) y luego, a medida que el código se vuelve más valioso, se migra a un plugin binario que se puede probar y compartir fácilmente entre múltiples proyectos u organizaciones.

### Plugins utilizados en jPOSv2
Estos son algunos de los plugins utilizados en un proyecto jPOSv2 (todos ellos son Gradle Core):

#### [The Base Plugin](https://docs.gradle.org/current/userguide/base_plugin.html) - 'base'
El Plugin Base provee tareas y convensiones que son comunes a la mayoría de los proyectos y añade una estructura que promueve la consistencia a la hora de ejecutarse. Su contribución más importante son las tareas build, clean, check, entre otras varias.

#### [The Java Plugin](https://docs.gradle.org/current/userguide/java_plugin.html) - 'java'
El Plugin Java extiende de Plugin Base y nos permite entre otras cosas:
    - Compilar archivos Java utilizando JDK
    - Ejecutar tests
    - Generar Javadoc
    - Eliminar el directorio build/*

También nos permite estructurar el directorio de proyecto (el cual se puede customizar desde `build.gradle`, ver documentación):
| Directorio | Descripción |
| :--- | :--- |
| `src/main/java` | Production Java source. |
| `src/main/resources` | Production resources, such as XML and properties files. |
| `src/test/java` | Test Java source. |
| `src/test/resources` | Test resources. |

#### [The Java Library Plugin](https://docs.gradle.org/current/userguide/java_library_plugin.html) - 'java-library'
El Plugin Java Library extiende de Plugin Java y nos permite construir librerías java, las cuales exponen APIs, su característica más importante. Una librería es un componente Java que será consumido por otros componentes.
Introduce la configuración de dependencias `api`. (Ver en apartado de dependencias)

#### [Maven Publish Plugin](https://docs.gradle.org/current/userguide/publishing_maven.html) - 'maven-publish'
El Plugin Maven Publish nos permite incluir los módulos o librerías que querramos en nuestro proyecto. Las podemos agregar como dependencias, buscando las releases en [MVN Repository](https://mvnrepository.com/) (las cuales sirven tanto para Gradle, Maven y otras herramientas).

#### [The IDEA Plugin](https://docs.gradle.org/current/userguide/idea_plugin.html) - 'idea'
El Plugin IDEA genera archivos que facilitan el uso del IDE IntelliJ IDEA. Con sus Gradle Tasks, facilita la apertura de proyectos y la generación de directorios y archivos necesarios para el proyecto. Este plugin está pensado para ser personalizado y por otra parte, provee un set estandarizado de ayudas para agregar y eliminar contenido de los archivos generados.

#### [The Eclipse Plugins](https://docs.gradle.org/current/userguide/eclipse_plugin.html) - 'eclipse'
El Plugin Eclipse genera archivos que facilitan el uso del IDE Eclipse.  Con sus Gradle Tasks, facilita la apertura de proyectos y la generación de directorios y archivos necesarios para el proyecto. Este plugin está pensado para ser personalizado y por otra parte, provee un set estandarizado de ayudas para agregar y eliminar contenido de los archivos generados.

Cuando se utiliza en cominación con otros plugins, agrega diferentes configuraciones a los archivos `.project` y/o `.classpath`.

Este plugin aplica automaticamente el Plugin `eclipse-wtp` cuandose se utiliza en un proyecto War o Ear. Si es otro tipo de proyecto, por ejemplo, web, se debe aplicar explicitamente el Plugin eclipse-wtp (el cual a su vez extiende de ecplise, por lo que no habría de utilizar ambos). 

#### [The War Plugin](https://docs.gradle.org/current/userguide/war_plugin.html) - 'war'
El Plugin War extiende de Plugin Java, añadiendo la capacidad de generar archivos WAR de aplicaciones web y cambiando la configuración por defecto de archivos JAR por una de arechivos WAR.
Este Plugin añade el directorio `src/main/webapp` y las tareas `providedCompile` y `providedRuntime`.

## Configuración de dependencias de Gradle
Cada dependencia para un proyecto aplica para un propósito específico. Por ejemplo, algunas dependencias deberian ser utilizadas para compilar el código fuente, mientras que otras necesitan estar disponibles al momento de ejecución. Gradle representa el alcance de una dependencia con la ayuda de las configuraciones. Cada configuración puede ser identificada con un único nombre.

Muchos plugins de Gradle agregan configuraciones predefinidas al proyecto. Por ejemplo, el Plugin Java, añade configuraciones para representar los varios classpaths que necesita para la compilación del codigo fuente, ejecutando test, entre otros.

Las dependencias se declaran dentro del archivo `build.gradle`, dentro del apartado `dependencies { }` y existen distintos tipos de configuración que podemos usar para aplicarlas como las que podemos ver a continuación.

| Configuración | Gradle Plugin | Comentarios | Descripcion |
| :--- | :--- | :--- | :--- |
| `api` | `java-library` | Anteriormente la deprecada `compile` desde Gradle 7+. | Expone las implementaciones de las librerías internas de esta dependencia, es decir, hacemos transitivas las librerías que componen la dependencia en cuestión. |
| `implementation` | `java` | La opción más "segura" | Por su parte, esta configuración NO permite acceder a las implementaciones de las librerías internas de esta dependencia. Por ejemplo, si dependencia utiliza Hibernate para la implementación de su capa interna de persistencia, no podremos acceder desde nuestra aplicación.|
| `testImplementation` | `java` | Extiende de `implementation`. Anteriormente la deprecada `testCompile` desde Gradle 7+. | Se utiliza para configurar las dependencias requeridas para compilar y ejecutar tests. Generalmente se utiliza el framework JUnit. |
| `providedCompile` | `war` | | Esta configuración debería ser usada por dependencias requeridas en tiempo de **compilación** pero que ya son provistas por el entorno en el cual el WAR es deployado. Estas dependencias son visibles para las clases main y test. Al ser `provided`, tiene un comportamiento parecido a `api` en cuanto a la transitividad de librerías internas |
| `providedRuntime` | `war` | | Esta configuración debería ser usada por dependencias requeridas en tiempo de **ejecución** pero que ya son provistas por el entorno en el cual el WAR es deployado. Estas dependencias son visibles para las clases main y test. Al ser `provided`, tiene un comportamiento parecido a `api` en cuanto a la transitividad de librerías internas |
| `compileOnly` | `java` | Anteriormente la deprecada `provided` | Dependencias utilizadas solo en tiempo de compilación, no ejecución. |
| `runtimeOnly` | `java` | Anteriormente la deprecada `apk` | Dependencias utilizadas solo en tiempo de ejecución, no compilación. |
| `testCompileOnly` | `java` |  | Dependencias utilizadas **para testing** solo en tiempo de compilación, no ejecución. |
| `testRuntimeOnly` | `java` | Extiende de `runtimeOnly` | Dependencias utilizadas **para testing** solo en tiempo de ejecución, no compilación. |

## Gradle tasks con jPOSv2
Estas son las tareas más comunes que ejecutaremos en un proyecto jPOSv2:
| Comando Gradle | Descripcion |
| :--- | :--- |
| `gradle clean` | Para eliminar la carpeta ./build/* y todo lo que genera el proyecto. |
| `gradle installApp` | Para generar el directorio ./build/* con todos los archivos necesarios de proyecto, entre ellos el java ejecutable .jar. |
| `gradle run` | Ejecuta el comando anterior `installApp` y luego ejecuta el archivo .jar del proyecto, iniciando el servicio Q2. |
