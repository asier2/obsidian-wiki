
### Archivo settings.gradle

Comenzamos por renombrar el archivo de *settings.gradle* a nivel de proyecto renombrándolo a *settings.gradle.kts* y agregando los repositorios de Orona si no lo están:

```
pluginManagement {
    repositories {
        maven {
            setUrl("https://devops.orona.es/nexus/repository/maven-devops_mobile/")
            credentials {
                username = providers.gradleProperty("ORONA_NEXUS_USER").getOrElse("")
                password = providers.gradleProperty("ORONA_NEXUS_PASS").getOrElse("")
            }
            authentication {
                create<BasicAuthentication>("basic")
            }
        }
        mavenCentral()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        maven {
            setUrl("https://devops.orona.es/nexus/repository/maven-devops_mobile/")
            credentials {
                username = providers.gradleProperty("ORONA_NEXUS_USER").getOrElse("")
                password = providers.gradleProperty("ORONA_NEXUS_PASS").getOrElse("")
            }
            authentication {
                create<BasicAuthentication>("basic")
            }
        }
        mavenCentral()
    }
}
```

Si los repositorios ya estaban en el build.gradle de alguno o varios de los módulos, deberemos eliminarlos.

Además, probablemente solo tengamos los include de los paquetes:
```
include ':lib'
```

Que deberemos migrar a Kotlin así:
```kotlin
include(":lib")
```

Para migrar cualquier contenido que pudiera quedar revisar el documento de migración de [[Migración Gradle Groovy a Kotlin]].

Si no se ha renombrado el archivo, si no que sea replicado con la extensión .kts, se deberá borrar el anterior archivo.

### Archivo build.gradle

Renombramos el archivo para pasarlo a Kotlin con .kts.

Cualquier dependencia o repositorio que quede definido en esta clase se deberá **borrar**, por ejemplo:

```groovy
buildscript {  
    repositories {  
        maven {  
            url 'https://devops.orona.es/nexus/repository/maven-devops_mobile/'  
            credentials {  
                username ORONA_NEXUS_USER  
                password ORONA_NEXUS_PASS  
            }  
  
            authentication {  
                basic(BasicAuthentication)  
            }  
        }  
    }  
    dependencies {  
        classpath 'com.android.tools.build:gradle:8.6.0'  
        classpath 'org.owasp:dependency-check-gradle:8.1.0'  
        classpath 'org.sonarsource.scanner.gradle:sonarqube-gradle-plugin:3.3'  
        // NOTE: Do not place your application dependencies here; they belong  
        // in the individual module build.gradle files    }  
}  
  
allprojects {  
    repositories {  
        maven {  
            url 'https://devops.orona.es/nexus/repository/maven-devops_mobile/'  
            credentials {  
                username ORONA_NEXUS_USER  
                password ORONA_NEXUS_PASS  
            }  
  
            authentication {  
                basic(BasicAuthentication)  
            }  
        }  
    }  
}
```

Si había bloque de dependencias, deberemos recuperarlas con un nuevo bloque de plugins. En el ejemplo anterior se usaba AGP, Dependency Check y Sonar, de modo que los recuperaremos. Es posible no tener que traer todas las dependecias de momento para compilarm como hacemos aquí con SONAR, pero tendremos que acordarnos de reponerla más adelante.

Además, agregaremos los nuevos plugins de Orona que vienen a sustituir los apply-from que veremos más adelante, pero de momento comentados.

```kotlin
plugins {  
    alias(libs.plugins.android.application) apply false  
    alias(libs.plugins.owasp.dependency.check) apply false  
    //alias(libs.plugins.orona.plugins.library) apply false  
    //alias(libs.plugins.orona.plugins.repo) apply false}
```

No debería quedar mucho más allá del versionado común de los módulos. El bloque pasaría de ser algo así en groovy:

```groovy
subprojects {  
    //Keep subproject versions in common  
    ext.libVersionName = "1.0.0"  
    ext.libVersionCode = 1 
}
```

A ser así en Kotlin:
```kotlin
//Keep subproject versions in common
val libVersionName: String by extra { "1.0.0" }
val libVersionCode: Int by extra { 1 }
```



Si no se ha renombrado el archivo, si no que sea replicado con la extensión .kts, se deberá borrar el anterior archivo.

Si queda alguna task específica del proyecto o alguna excepción similar, habrá que migrarla a Kotlin.

### Archivos build.gradle de módulos
Para finalizar este paso y poder compilar la pp, debemos migrar los plugins de android, dependecy check y sonar de los distintos módulos. Si hubiese alguno más, debería estar agregado ya al Catalog en el [[01 Migrar dependencias a Catalog|segundo paso]] para que podamos recuperarlo.

Los bloques de groovy para aplicar plugins así:

```groovy
apply plugin: 'com.android.application'
```

Pasarán a usar el bloque plugins tal que así:

```kotlin
plugins {  
    id 'com.android.application'  
}
```

En caso de hacer uso de los plugins de Orona con `apply from`, los dejaremos de usar temporalmente para poder compilar y seguir con la migración.
Los apply from originales en Groovy:

```groovy
apply from: 'https://devops.orona.es/android/new-docker-global-lib-config.gradle'  
apply from: 'https://devops.orona.es/android/new-docker-global-repo-config.gradle'
```

Pasarán a recuperarse a mano, tal que así:

```kotlin
plugins {  
    id 'com.android.library'  
    id 'maven-publish'  
}
```