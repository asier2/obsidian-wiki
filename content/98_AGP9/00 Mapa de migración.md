
A continuación se define un "mapa" para seguir y guiarse en la transición de las librerías a AGP 9. Está en orden "cronológico", puesto que cambiar el orden de los pasos nos puede llevar a bloqueos.

## 01 Migrar depencias a Catalog

[[01 Migrar dependencias a Catalog]].

## 02 Agregar plugins de Orona a Catalog

[[02 Agregar plugins extra a Catalog.]]

## 03 Requisitos para migrar plugins al Catalog 

Revisar que no haya bloques como `buildscript {}` o `apply plugin:` en los build.gradle de los módulos. 

### 03.1 Si los hay

Sólo se podrán migrar las dependencias y no los plugins por el momento. Volveremos a este punto más adelante.

### 03.2 Si no los hay
[[03 Migrar los plugins]]

## 04 Migrar los .gradle de proyecto a Kotlin
[[04 Migrar settings.gradle y build.gradle de proyecto a Kotlin + plugins]]

## 05 Migrar los .gradle de módulo a Kotlin
[[05 Migrar los .gradle de módulo a Kotlin]]

## 06 Migrar a AGP 9
[[06 Migrar a AGP 9]]

## 07 (Opcional) Reponer contenido comentado

En caso de que hayamos comentado contenido como los plugins, el bloque publishing, étc... 
[[07 Reponer contenido comentado]]

Si no nos queda contenido comentado que recuperar, seguimos conel siguiente bloque:

## 08 