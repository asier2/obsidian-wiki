## Plugins

En caso de no haber podido importar los plugins en su momento, deberemos hacerlo ahora. Existen tres plugins que reemplazan a los scripts originales. Los scripts de definían al inicio del archivo de la siguiente manera:

```groovy
apply from: "https://devops.orona.es/android/new-docker-global-lib-config.gradle"
apply from: "https://devops.orona.es/android/new-docker-global-repo-config.gradle"
apply from: "https://devops.orona.es/android/new-docker-global-app-config.gradle"
```

Ahora pasaríamos a usar los plugins tal que así:
```kotlin
plugins {
	alias(libs.plugins.orona.plugins.application)  
    alias(libs.plugins.orona.plugins.library)  
    alias(libs.plugins.orona.plugins.repo)  
}
```

Las aplicaciones harán uso del script/plugins de 'application'. 
Las librerías el de 'library' y el de 'repo' para poder publicarse. Se puede ver el contenido de gradle que estamos inyectando en cada caso en el repositorio de los plugins [aquí](https://devops.orona.es/gitlab/dto/mobile/libraries/orona-android-plugins/-/blob/main/src/main/kotlin/com/oronagroup/plugins/AndroidRepoPlugin.kt?ref_type=heads).

## Bloque Android Component