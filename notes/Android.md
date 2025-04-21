# Android 

## rootProject

在 Android Studio 的 `build.gradle` 文件中，`rootProject` 指的是整个 Gradle 项目的根目录。它通常用于访问和管理项目中定义的属性和设置。

在一个多模块的 Gradle 项目中，`rootProject` 是最上层的项目，包含一个或多个子模块。每个子模块也有自己的 `build.gradle` 文件，但所有模块共享根项目的设置。

假设您在根项目的 `build.gradle` 文件中定义了一个扩展属性：

```
// 根项目 build.gradle
ext {
    kotlinVersion = "1.5.31"
    appVersion = "1.0.0"
}
```

在子模块的 `build.gradle` 文件中，您可以使用 `rootProject` 来引用这些属性：

```groovy
// 子模块 build.gradle
dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib:$rootProject.kotlinVersion"
}
```