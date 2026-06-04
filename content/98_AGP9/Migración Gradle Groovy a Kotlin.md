## Primeros pasos[^1]

 ### Declaración de Strings
 
 En [[Groovy]] los `String`s pueden declarase con una comilla `'` o con doble `"`. En Kotlin todos los `String` deberán ir con doble comilla `"`.
Groovy:
```
name 'release'
```

Kotlin:
```
name = "release"
```

Solución:
> **Buscar comillas simples en el archivo y reemplazarlas por dobles.**

 ### Asignaciones de propiedades
 
En [[Groovy]] se permite omitir el signo de `=` en asignaciones a propiedades, en Kotlin no.
 
Groovy:
```
defaultConfig { 
	compileSdk 36
}
```

Kotlin:
```
defaultConfig {
	compileSdk = 36
}
```

Solución:
> **Agregar un signo de `=` en todas las propiedades, incluso en asignaciones en las que no estemos seguros. Si en realidad se trataba de una invocación a una función, el compilador nos lo marcará en rojo. **

 ### Invocación de funciones
 
 [[Groovy]] permite omitir los paréntesis al invocar funciones, Kotlin no.
 
Groovy:
```
properties { 
	property "sonar.host.url", "https://devops.orona.es/sonarqube"
}
```

Kotlin:
```
properties {
	property("sonar.host.url", "https://devops.orona.es/sonarqube")
}
```

Solución:
> **Como se comenta previamente en Asignaciones de propiedades, ante la duda se recomienda probar a asignar primero con `=`, en caso de que el compilador se queje, procedemos a realizar la invocación a función con paréntesis `()`.**


 ### Transición del bloque plugins

En [[Groovy]], era habitual aplicar plugins y scripts con `apply plugin:` y `apply from:`. En Kotlin DSL, lo recomendado es centralizar plugins en el bloque `plugins {}` y evitar `apply` salvo casos muy puntuales.

Groovy:
```groovy
apply from: 'https://devops.orona.es/android/new-docker-global-lib-config.gradle'
apply from: 'https://devops.orona.es/android/new-docker-global-repo-config.gradle'
apply plugin: 'org.owasp.dependencycheck'
```

Kotlin:
```kotlin
plugins {
    alias(libs.plugins.orona.plugins.library)
    alias(libs.plugins.orona.plugins.repo)
    alias(libs.plugins.android.library)
    alias(libs.plugins.owasp.dependency.check)
}
```

Los plugin se deberán agregar como dependencia en el catalog así:
```
orona-plugins-library = { id = "com.oronagroup.android.library", version = "1.0.0" }
orona-plugins-application = { id = "com.oronagroup.android.application", version = "1.0.0" }
orona-plugins-repo = { id = "com.oronagroup.android.repo", version = "1.0.0" }
```

Solución:
> **Mover los `apply plugin:` al bloque `plugins {}` usando `alias(...)` desde el Version Catalog (`libs.versions.toml`). Mantener `apply(from = ...)` solo cuando no exista plugin equivalente o haya una dependencia fuerte con scripts legacy.**

### Tipado estricto en Kotlin DSL

[[Groovy]] permite tipado dinámico, cosa que Kotlin no, por lo que es probable que tengas que especificar los tipos en casos en los que antes no.

 Ejemplos habituales:
 
 1. `String?` (nullable)

 En Kotlin, cuando una propiedad puede no existir, su tipo suele ser nullable (`String?`).
 
 Groovy:
```
username project.findProperty("nexus.user")
```

 Kotlin:
```
username = project.findProperty("nexus.user") as String?
```

 
 2. Uso de `uri(...)`

 En Kotlin DSL, propiedades como `url` esperan un tipo `URI`, no un `String` directo.
 
 Groovy:
```
url project.findProperty("nexus.url")
```

 Kotlin:
```
url = uri(project.findProperty("nexus.url")!!)
```

 3. `orNull` 
 
 Si la propiedad a la que queremos asignar un valor es nullable, pero la función que invocamos para resolver su valor no lo es, podemos usar  `orNull`.

 Kotlin:
```
username = providers.environmentVariable("ORONA_NEXUS_USER").orNull
password = providers.environmentVariable("ORONA_NEXUS_PASS").orNull
```

 Solución:

> **Cuando el compilador detecta incompatibilidad de tipos, revisa qué tipo espera esa propiedad en Kotlin y realiza la conversión explícita necesaria, bien con casting (`as String?`), métodos auxiliares `uri(...)`,  `orNull`, etc.).**

### Recoger credenciales de gradle.properties

```
username = providers.gradleProperty("ORONA_NEXUS_USER").getOrElse("")
```

### Recuperar la versión común (en librerías o multipaquete)

Si en settings.gradle se definen las versiones de manera similar a esto:
```kotlin
val libVersionName: String by extra { "1.0.0" }
val libVersionCode: Int by extra { 1 }
```
En cada el build.gradle de cada paquete habrá que agregar por encima del bloque android {}
``` kotlin
val appVersionName: String by rootProject.extra
val appVersionCode: Int by rootProject.extra
```

Para poder asignar la versión en defaultConfig{} -> versionName y versionCode.

## Guía de bloques
### Bloque dependencies

Un bloque de dependencias en Groovy es muy similar a la versión de Kotlin, pero en Kotlin DSL las llamadas deben ser explícitas.

Groovy:
```groovy
dependencies {
    desImplementation platform(libs.orona.bom.des)
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation libs.androidx.appcompat
    implementation(libs.orona.cas.api) {
        exclude module: 'commons-logging'
    }
    testImplementation libs.junit
    androidTestImplementation libs.androidx.test.junit
}
```

Kotlin:
```kotlin
dependencies {
    "desImplementation"(platform(libs.orona.bom.des))
    implementation(fileTree(mapOf("dir" to "libs", "include" to listOf("*.jar"))))
    implementation(libs.androidx.appcompat)
    implementation(libs.orona.cas.api) {
        exclude(module = "commons-logging")
    }
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.test.junit)
}
```

Solución:
> **En Kotlin DSL, usar siempre llamadas con paréntesis (`implementation(...)`) y convertir estructuras de Groovy (`[ ... ]`) a `mapOf(...)`/`listOf(...)`.**

### Bloque apply (from y plugins)

Los plugins o scripts remotos definidos así en Groovy:

```groovy
apply from: 'https://devops.orona.es/android/new-docker-global-lib-config.gradle'
apply from: 'https://devops.orona.es/android/new-docker-global-repo-config.gradle'
apply plugin: 'org.owasp.dependencycheck'
```

En Kotlin, los plugins deben ir en `plugins {}`:

```kotlin
plugins {
    alias(libs.plugins.orona.plugins.library)
    alias(libs.plugins.orona.plugins.repo)
    alias(libs.plugins.android.library)
    alias(libs.plugins.owasp.dependency.check)
}
```

Si necesitas mantener scripts externos legacy:

```kotlin
apply(from = "https://devops.orona.es/android/new-docker-global-lib-config.gradle")
apply(from = "https://devops.orona.es/android/new-docker-global-repo-config.gradle")
```

Solución:
> **Priorizar `plugins {}` para plugins versionados y dejar `apply(from = ...)` para casos excepcionales. NO deberían necesitarse nunca.**

### Bloques buildConfigField

En Groovy, se podía usar comillas simples para generar Strings.

Groovy:
```groovy
buildConfigField "String", "ORONA_LOGIN_SERVICE_NAME", "'com.orona.ol.service.LoginService_'"
```

Kotlin:
```kotlin
buildConfigField("String", "ORONA_LOGIN_SERVICE_NAME", "\"com.orona.ol.service.LoginService_\"")
```

En el caso de enteros:
```kotlin
buildConfigField("int", "UNLOCK_TIMER_SECONDS", "20")
```

Solución:
> **Transformar `buildConfigField` en invocación con paréntesis y escapar Strings anidados con `\"...\"`.**

### Parámetro `arguments` de `javaCompileOptions.annotationProcessorOptions`

Groovy:
```groovy
arguments = [
    "androidManifestFile": "$projectDir/src/main/AndroidManifest.xml".toString()
]
```

Kotlin (1 elemento):
```kotlin
arguments["androidManifestFile"] = "$projectDir/src/main/AndroidManifest.xml"
```

Kotlin (múltiples):
```kotlin
arguments += mapOf("androidManifestFile" to "$projectDir/src/main/AndroidManifest.xml")
```

Solución:
> **Sustituir mapas de Groovy por `mapOf(...)` y usar `+=` para agregar claves sin perder las existentes.**

### Parámetros varios

#### `minifyEnabled` pasa a `isMinifyEnabled`

Groovy:
```groovy
buildTypes {
    release {
        minifyEnabled true
    }
}
```

Kotlin:
```kotlin
buildTypes {
    getByName("release") {
        isMinifyEnabled = true
    }
}
```

Solución:
> **Las propiedades booleanas suelen migrar al prefijo `is...` en Kotlin DSL.**

#### `lintOptions` pasa a `lint`

Groovy:
```groovy
lintOptions {
    disable "InvalidPackage"
}
```

Kotlin:
```kotlin
lint {
    disable += "InvalidPackage"
}
```

Solución:
> **Renombrar el bloque a `lint` y adaptar colecciones con operadores de Kotlin como `+=`.**

#### `fileTree`

Groovy:
```groovy
implementation fileTree(dir: "libs", include: ["*.jar"])
```

Kotlin:
```kotlin
implementation(fileTree(mapOf("dir" to "libs", "include" to listOf("*.jar"))))
```

Solución:
> **Convertir parámetros nombrados estilo Groovy a `mapOf(...)` en Kotlin DSL.**

#### `exclude` en dependencias

Groovy:
```groovy
implementation(libs.orona.rpc.security.api) {
    exclude module: "commons-logging"
}
```

Kotlin:
```kotlin
implementation(libs.orona.rpc.security.api) {
    exclude(module = "commons-logging")
}
```

Solución:
> **Usar parámetros nombrados de Kotlin en exclusiones (`exclude(module = ...)`, `exclude(group = ...)`).**

#### `flavorDimensions` y `productFlavors`

Groovy:
```groovy
flavorDimensions "environment"

productFlavors {
    pro {
    }
}
```

Kotlin:
```kotlin
flavorDimensions += "environment"

productFlavors {
    create("pro") {
    }
}
```

Solución:
> **Crear flavors explícitamente con `create("...")` y añadir dimensiones con `+=`.**

#### `proguardFiles`

Groovy:
```groovy
proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
```

Kotlin:
```kotlin
proguardFiles(
    getDefaultProguardFile("proguard-android-optimize.txt"),
    "proguard-rules.pro"
)
```

Solución:
> **Convertir la invocación a formato Kotlin con paréntesis y comillas dobles y renombrar de 'proguard-android.txt' a 'proguard-android-optimize.txt'**


outputFileName

antes
    applicationVariants.all { variant ->  
        variant.outputs.all { output ->  
            output.outputFileName = "${variant.applicationId}-${variant.versionName}-${variant.versionCode}.apk"  
            true  
        }  
    }

después
androidComponents {  
    onVariants { variant ->  
        variant.outputs.forEach { output ->  
            output.outputFileName.set("${variant.applicationId.getOrElse("")}-$libVersionName-$libVersionCode.apk")  
        }  
    }}

Generar sourceJar

```groovy
tasks.register('sourceJar', Jar) {
    from android.sourceSets.main.java.srcDirs
    archiveClassifier.set("sources")
}
```

```kotlin
val sourceJar by tasks.registering(Jar::class) {
    from(
        fileTree("src/main") {
            include("**/*.java")
            include("**/*.kt")
        }
    )
```

## Referencias útiles

- https://github.com/gradle/gradle/issues/24491
- Propiedades de gradle.properties: https://qiita.com/Nabe1216/items/271b643df47b04f767cd
- Migrate to Kotlin DSL: https://developer.android.com/build/migrate-to-kotlin-dsl
- Migrate to built-in Kotlin: https://developer.android.com/build/migrate-to-built-in-kotlin
- Update KSP version: https://stackoverflow.com/questions/79901152/using-kotlin-sourcesets-dsl-to-add-kotlin-sources-is-not-allowed-with-built-in-k
- Sonar properties: https://docs.sonarsource.com/sonarqube-server/analyzing-source-code/scanners/sonarscanner-for-gradle
- gradle.property: https://github.com/gradle/gradle/issues/24491
- Dependency check gradle: https://github.com/dependency-check/dependency-check-gradle
- signingConfig ref: https://gist.github.com/mileskrell/7074c10cb3298a2c9d75e733be7061c2
- Gradle recipes: https://github.com/android/gradle-recipes
https://developer.android.com/build/migrate-to-built-in-kotlin


[^1]: https://docs.gradle.org/current/userguide/migrating_from_groovy_to_kotlin_dsl.html#prepare_your_groovy_scripts
