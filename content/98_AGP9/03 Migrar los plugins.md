### Migrate plugins

Add an entry for each plugin in both the versions and the plugins sections of
the `libs.versions.toml` file. Sync your project, and then replace their
declarations in the `plugins{}` block in the build files with their catalog
names.

This code snippet shows the `build.gradle.kts` file before removing the
plugin:

### Kotlin

```kotlin
// Top-level `build.gradle.kts` file
plugins {
   id("com.android.application") version "7.4.1" apply false

}

// Module-level `build.gradle.kts` file
plugins {
   id("com.android.application")

}
```

### Groovy

```groovy
// Top-level `build.gradle` file
plugins {
   id 'com.android.application' version '7.4.1' apply false

}

// Module-level `build.gradle` file
plugins {
   id 'com.android.application'

}
```

This code snippet shows how to define the plugin in the version catalog file:

    [versions]
    androidGradlePlugin = "7.4.1"

    [plugins]
    android-application = { id = "com.android.application", version.ref = "androidGradlePlugin" }

As with dependencies, the mandatory way of formatting for `plugins` block catalog
entries is kebab case (such as `android-application`).

The following code shows how to define the `com.android.application` plugin in
the top and module level `build.gradle.kts` files. Use `alias` for plugins
that come from the version catalog file and `id` for plugins that don't come
from the version catalog file, such as
[convention plugins](https://docs.gradle.org/current/samples/sample_convention_plugins.html#organizing_build_logic).

### Kotlin

```kotlin
// Top-level build.gradle.kts
plugins {
   alias(libs.plugins.android.application) apply false

}

// module build.gradle.kts
plugins {
   alias(libs.plugins.android.application)

}
```

### Groovy

```groovy
// Top-level build.gradle
plugins {
   alias libs.plugins.android.application apply false

}

// module build.gradle
plugins {
   alias libs.plugins.android.application

}
```

> [!NOTE]
> **Note:** If you are using a version of Gradle below 8.1, you need to annotate the `plugins{}` block with `@Suppress("DSL_SCOPE_VIOLATION")` when using version catalogs. Refer to [issue #22797](https://github.com/gradle/gradle/issues/22797) for more info.
