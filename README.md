# Gradle Shadow

Shadow is an extension of the Gradle Jar task that optimizes FatJar/UberJar creation by using JarInputStream and
JarOutputStream to copy file contents. This avoids the unnecessary I/O overhead of expanding jar files to disk
before recombining them. Shadow provides the similar filtering, relocation, and transformation capabilities as the
Maven Shade plugin. Starting with version 0.9, Shadow is a complete re-write based on core Gradle classes and concepts
instead of a port of the Maven Shade code. Documentation for version 0.8 and prior can be found [here](README_old.md)

## Current Status

<a href='https://bintray.com/johnrengelman/gradle-plugins/gradle-shadow-plugin/view?source=watch' alt='Get automatic notifications about new "gradle-shadow-plugin" versions'><img src='https://www.bintray.com/docs/images/bintray_badge_color.png'></a>
[ ![Download](https://api.bintray.com/packages/johnrengelman/gradle-plugins/gradle-shadow-plugin/images/download.png) ](https://bintray.com/johnrengelman/gradle-plugins/gradle-shadow-plugin/_latestVersion)
[![Build Status](https://drone.io/github.com/johnrengelman/shadow/status.png)](https://drone.io/github.com/johnrengelman/shadow/latest)


## QuickStart

### Applying Shadow Plugin to Project

```
buildscript {
  repositories { jcenter() }
  dependencies {
    classpath 'com.github.jengelman.gradle.plugins:shadow:1.0.1'
  }
}

apply plugin: 'java' // or 'groovy'. Must be explicitly applied
apply plugin: 'com.github.johnrengelman.shadow'
```

Note: Applying the `ShadowPlugin` to a project applies the majority of its settings via a callback on the application of
other plugins. For example, the bulk of `shadow` is only added to the project if the `java` or `groovy` plugins are also
added. Shadow will **not** add them automatically, but instead listens for their application and responds.

### Using the default plugin task

```
$ gradle shadowJar //shadow the runtime configuration with project code into ./build/libs/
```

`shadowJar` by uses the same default configurations as `jar` and additionally configures the `classifier` to be `"all"`.
Additionally, it creates a 'shadow' configuration and assigns the jar as an artifact of it. This configuration can
be used to add dependencies that are excluded from the shadowing.

### Integrating with Application Plugin

```
apply plugin: 'application'
apply plugin: 'com.github.johnrengelman.shadow'
```

Applying both `shadow` and `application` to a project will create a number of additional tasks to be created. These
tasks mimic the `application` plugin but execute using the output of the `shadowJar` task.

Applying the `application` plugin will cause the `shadowJar` to include the `Main-Class` attribute in the manifest of
the `shadowJar` output. This is configured via the `mainClassName` attribute from the `application` plugin

## Advanced Configuration

### Configure MANIFEST file

By default, uses the same manifest as the `Jar` task.

```
jar {
  manifest {
    attributes("Implementation-Title": "Gradle", "Implementation-Version": version)
  }
}
```

### Modifying the MANIFEST file

Append to the Jar MANIFEST

```
shadowJar {
  appendManifest {
    attributes 'Test-Entry': 'PASSED'
  }
}
```

Replace the Jar MANIFEST

```
shadowJar {
  manifest {
    attributes("Implementation-Title": "Gradle", "Implementation-Version": version)
  }
}
```

### Merging Service files

```
shadowJar {
  mergeServiceFiles()
}
```

**OR**

```
import com.github.jengelman.gradle.plugins.shadow.transformers.ServiceFileTransformer

shadowJar {
  transform(ServiceFileTransformer)
}
```

### Appending Files

```
shadowJar {
  append('NOTICE')
}
```

**OR**

```
import com.github.jengelman.gradle.plugins.shadow.transformers.AppendingTransformer

shadowJar {
  transform(AppendingTransformer) {
    resource = 'NOTICE'
  }
}
```

### Filtering shadow jar contents by file pattern

```
shadowJar {
  exclude 'LICENSE'
}
```

### Filtering shadow jar contents by maven/project dependency

Exclude specific dependency (transitive dependencies are **not** excluded)

```
shadowJar {
  dependencies {
    exclude(dependency('asm:asm:3.3.1'))
  }
}
```

Include specific dependency (transitive dependencies are **not** included)

```
shadowJar {
  dependencies {
    include(dependency('asm:asm:3.3.1'))
  }
}
```

Exclude a project dependency in a multi-project build

```
shadowJar {
  dependencies {
    exclude(project(":myclient"))
  }
}
```

### Relocating dependencies

```
shadowJar {
  relocate 'org.objectweb.asm', 'myjarjarasm.asm'
}
```

### Filtering files in relocation

```
shadowJar {
  relocate('org.objectweb.asm', 'myjarjarasm.asm') {
    exclude 'org.objectweb.asm.ClassReader'
  }
}
```

### Transforming resources

Uses the [Transformer](src/main/grovy/com/github/jengelman/gradle/plugins/shadow/transformers/Transformer.groovy) interface.

```
shadowJar {
  transform(<Transformer class>) {
    //..configure the Transformer class instance
  }
}
```

### Publishing the shadow jar as an additional resource to the main jar

```
apply plugin: 'com.github.johnrengelman.shadow'
apply plugin: 'maven-publish'

publishing {
  publications {
    shadow(MavenPublication) {
      from components.java
      artifact shadowJar
    }
  }
}
```

### Publishing the shadow jar as a standalone artifact

```
apply plugin: 'com.github.johnrengelman.shadow'
apply plugin: 'maven-publish'

shadowJar {
  baseName = 'myproject-all'
  classifier = ''
}

publishing {
  publications {
    shadow(MavenPublication) {
      from components.shadow
      artifactId = 'myproject-all'
    }
  }
}
```

### Configuring additional POM dependencies for Shadow Jar

```
dependencies {
  compile 'asm:asm:3.3.1'
  compile 'org.bouncycastle:bcprov-jdk15on:1.47'
  shadow 'org.bouncycastle:bcprov-jdk15on:1.47'
}

shadowJar {
  dependencies {
    exclude(dependency('org.bouncycastle:bcprov-jdk15on:1.47'))
  }
}
```

This examples allows the project to compile against the BouncyCastle encryption library, but then excludes it from
the shadowed jar, but including it as a dependency on the 'shadow' configuration.

Additionally, any dependencies added to the `shadow` configuration will be added to the `Class-Path` attribute in
the JAR Manifest for the output of `shadowJar`.

## ChangeLog

[ChangeLog](ChangeLog.md)
