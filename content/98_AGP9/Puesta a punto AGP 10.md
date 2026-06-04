
## `android.defaults.buildfeatures.resvalues`

Esta opción hace que queden desactivados por defecto los recursos definidos dentro de Gradle, por ejemplo:

```kotlin
defaultConfig {
    resValue("string", "api_url", "https://api.example.com")
}
```

Se deberán quitar las definiciones de recursos en Gradle o bien activar en cada paquete la funcionalidad:

```kotlin
buildFeatures {
    resValues = true
}
```

Finalmente borrar el flag de gradle.properties.
## `android.sdk.defaultTargetSdkToCompileSdkIfUnset=false`

Por defecto viene a false para preservar el funcionamiento como hasta ahora, por lo que usa `compileSdkVersion` cuando `targetSdkVersion` no está definido.  

Con es flag a `true`, o borrado, se exige configuración explícita, requerido en AGP 10.

## `android.enableAppCompileTimeRClass`

Esta opción unifica el manejo de `R class` entre aplicaciones y librerías. Por defecto viene a `false` para preservar el comportamiento anterior donde los IDs de `R` son `final`.

Con el flag a `true`, o borrado, los IDs de `R` dejan de ser `final`. Esto afecta especialmente al código Java que usa `R.id.*` en sentencias `switch`, que dejarán de compilar (los `case` en `switch` requieren constantes en tiempo de compilación).

**Acciones requeridas:**

1. Buscar todas las sentencias `switch` que usen `R.id.*`:
```Java
switch (view.getId()) {
	case R.id.button_ok:
		// ...
		break;
	case R.id.button_cancel:
		// ...
		break;
}
```

2. Reemplazar por `if/else` o condicionales equivalentes:
```java
//Migrarlo a un bloque if
int id = view.getId();
if (id == R.id.button_ok) {
	// ...
} else if (id == R.id.button_cancel) {
	// ...
}
```
3. Finalmente borrar el flag de `gradle.properties`.


android.sdk.defaultTargetSdkToCompileSdkIfUnset=false
android.enableAppCompileTimeRClass=false
android.usesSdkInManifest.disallowed=false
android.uniquePackageNames=false
android.dependency.useConstraints=true
android.r8.strictFullModeForKeepRules=false
android.r8.optimizedResourceShrinking=false
android.builtInKotlin=false
android.newDsl=false
 

[^1]: https://qiita.com/Nabe1216/items/271b643df47b04f767cd