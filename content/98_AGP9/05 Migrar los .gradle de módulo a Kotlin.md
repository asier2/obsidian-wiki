Para hacer la migración más sencilla en librerías, lo ideal es comentar temporalmente los bloques de publishing y las tareas relacionadas. Después, acercar lo que queda de código lo máximo posible a la estructura de Kotlin.

```groovy
/*tasks.register('sourceJar', Jar) {  
    from android.sourceSets.main.java.srcDirs    archiveClassifier.set('source')}  
  
tasks.withType(PublishToMavenRepository) { task ->  
    def match = task.name =~ '^publish(.*)AarPublicationTo(.*)$'    dependsOn("assemble${match[0][1]}")}  
  
publishing {  
    publications {        android.libraryVariants.all { variant ->            if (variant.buildType.name == "release") {                println("Publish variant:")                println("${variant.flavorName}-${variant.buildType.name}")                "${variant.name.capitalize()}Aar"(MavenPublication) {                    groupId 'com.orona.framework'                    artifactId "diagnostic-lib"                    version "$libVersionName-${variant.flavorName.toUpperCase()}"  
                    final artifactName = "${project.name}-${variant.flavorName}-${variant.buildType.name}.aar"                    final artifactPath = "${buildDir}/outputs/aar/"                    println("${artifactPath}${artifactName}")                    //Check output file name                    artifact("${artifactPath}${artifactName}")                    artifact(sourceJar)                    pom.withXml {                        final dependenciesNode = asNode().appendNode('dependencies')  
                        ext.addDependency = { Dependency dep, String scope ->                            if (dep.group == null || dep.version == null || dep.name == null || dep.name == "unspecified")                                return // ignore invalid dependencies  
                            final dependencyNode = dependenciesNode.appendNode('dependency')                            dependencyNode.appendNode('groupId', dep.group)                            dependencyNode.appendNode('artifactId', dep.name)                            dependencyNode.appendNode('version', dep.version)                            dependencyNode.appendNode('scope', scope)  
                            if (!dep.transitive) {                                // If this dependency is transitive, we should force exclude all its dependencies them from the POM                                final exclusionNode = dependencyNode.appendNode('exclusions').appendNode('exclusion')                                exclusionNode.appendNode('groupId', '*')                                exclusionNode.appendNode('artifactId', '*')                            } else if (!dep.properties.excludeRules.empty) {                                // Otherwise add specified exclude rules                                final exclusionNode = dependencyNode.appendNode('exclusions').appendNode('exclusion')                                dep.properties.excludeRules.each { ExcludeRule rule ->                                    exclusionNode.appendNode('groupId', rule.group ?: '*')                                    exclusionNode.appendNode('artifactId', rule.module ?: '*')                                }                            }                        }  
                        // List all "api" dependencies (for new Gradle) as "compile" dependencies                        configurations.api.getDependencies().each { dep -> addDependency(dep, "compile") }                        configurations."${variant.flavorName}Api".getDependencies().each { dep -> addDependency(dep, "compile") }                        // List all "implementation" dependencies (for new Gradle) as "runtime" dependencies                        configurations."${variant.flavorName}Implementation".getDependencies().each { dep -> addDependency(dep, "runtime") }                        configurations.implementation.getDependencies().each { dep -> addDependency(dep, "runtime") }                    }                }            }        }    }}*/
```

También el bloque que renombra las apks:

```groovy
applicationVariants.all { variant ->  
    variant.outputs.all {  
        outputFileName = "${variant.applicationId}-${variant.versionName}-${variant.versionCode}.apk"  
    }  
    variant.getRuntimeConfiguration().exclude(group: "com.google.code.findbugs", module: "jsr305")  
    variant.getRuntimeConfiguration().exclude(group: "com.google.code.findbugs", module: "annotations")  
}
```

Tendremos que comenzar por lo más sencillo, agregar paréntesis a funciones, usar signos de asignación, comillas dobles... A continuación se detallan los cambios comunes más habituales:

### Recuperar la versión común (en librerías o multipaquete)

Si en settings.gradle se definen las versiones de manera similar a esto:
```kotlin
val libVersionName: String by extra { "1.0.0" }
val libVersionCode: Int by extra { 1 }
```
En cada el build.gradle de cada paquete habrá que agregar por encima del bloque android {}
``` kotlin
val libVersionName: String by rootProject.extra
val libVersionCode: Int by rootProject.extra
version = "$libVersionCode-$libVersionName"
```

Para poder asignar la versión en defaultConfig{} -> versionName y versionCode.
Para librerías usamos el sufijo _lib_ y para apps _app_. 
El bloque version es para que SONAR pueda acceder a él. Hay que agregarlo siempre al menos al módulo principal de la app o librería.

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

En elcaso de los buildConfigField, hay que _escapar_ las comillas dobles con `\` en el caso de los Strings para que los mantenga como tal:

```groovy
        buildConfigField 'String', 'ORONA_LOGIN_SERVICE_ACTION', '"com.orona.login.Service"'
```

```kotlin
        buildConfigField("String", "ORONA_LOGIN_SERVICE_ACTION", "\"com.orona.login.Service\"")
```

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


### fileTree implementation

Si tenemos un bloque fileTree:

```groovy
implementation fileTree(dir: 'libs', include: ['*.jar'])
```

Pasará a ser así:

```kotlin
implementation(fileTree(mapOf("dir" to "libs", "include" to listOf("*.jar"))))
```

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

### Depencias por flavour
En Groovy:
```groovy
desApi(libs.orona.rpcsecurity.lib.des)  
preApi(libs.orona.rpcsecurity.lib.pre)  
proApi(libs.orona.rpcsecurity.lib.pro)
```
En Kotlin pasan a estar entrecomilladas.
```kotlin
"desApi"(libs.orona.rpcsecurity.lib.des)  
"preApi"(libs.orona.rpcsecurity.lib.pre)  
"proApi"(libs.orona.rpcsecurity.lib.pro)
```

Debería estar todo, compilado y ejecutable. Es cierto que hemos desactivado funcionalides que luego habrá que reponer, pero este estado es suficiente para avanzar a AGP 9 y terminar de migrar todo.

Si aún queda algún bloque por migrar a Kotlin, apóyate en la IA y actualiza el documento después ;)