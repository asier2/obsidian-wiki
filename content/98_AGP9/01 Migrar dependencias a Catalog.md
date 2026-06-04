## Create a version catalog file

Start by creating a version catalog file. In your root project's `gradle`
folder, create a file called `libs.versions.toml`. 

In your `libs.versions.toml` file, add these sections:

    [versions]

    [libraries]

    [plugins]

The sections are used as follows:

- In the `versions` block, define variables that hold the versions of your dependencies and plugins. You use these variables in the subsequent blocks (the `libraries` and `plugins` blocks).
- In the `libraries` block, define your dependencies.
- In the `plugins` block, define your plugins.

### Migrate dependencies

Add an entry for each dependency in both the `versions` and `libraries` sections
of the `libs.versions.toml` file. Sync your project, and then replace their
declarations in the build files with their catalog names.

This code snippet shows the `build.gradle.kts` file before removing the
dependency:

### Kotlin

```kotlin
dependencies {
    implementation("androidx.core:core-ktx:1.9.0")
}
```

### Groovy

```groovy
dependencies {
    implementation 'androidx.core:core-ktx:1.9.0'
}
```

This code snippet shows how to define the dependency in the version
catalog file:

    [versions]
    ktx = "1.9.0"

    [libraries]
    androidx-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "ktx" }

The naming for dependencies block in catalogs is kebab case (such as `androidx-ktx`), it's mandatory. 

In the `build.gradle.kts` file of each module that requires the dependency,
define the dependencies by the names you defined in the TOML file.
### Kotlin

```kotlin
dependencies {
   implementation(libs.androidx.ktx)
}
```

### Groovy

```groovy
dependencies {
   implementation libs.androidx.ktx
}
```
