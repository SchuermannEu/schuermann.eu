---
layout: post
title: Gradle task for deploying to Glassfish
author: Dominik Schürmann
date: 2013-08-03
slug: gradle-glassfish
aliases:
    - "/2013/08/03/gradle-glassfish.html"
---

I am switching to Gradle build system for all Java projects I am working on.
As I didn’t found a way to “run” a generated war file of a webproject on Glassfish, I wrote this small Gradle tasks, which uses asadmin executeable from the directory of your Glassfish instance:

```groovy
/**
 *  ~/.gradle/gradle.properties:
 *  glassfishHome=/path/to/glassfish_home
 *
 *  or in Netbeans, right click project, Properties, Manage Build in Tasks, Run
 *  Add line to Arguments: -Dorg.gradle.project.glassfishHome=/path/to/glassfish_home
 *
 *  For more information about Exec tasks see
 *  http://www.gradle.org/docs/current/dsl/org.gradle.api.tasks.Exec.html
 */
task run(dependsOn: 'war', type:Exec) {
    workingDir "${glassfishHome}${File.separator}bin"

    if (System.properties['os.name'].toLowerCase().contains('windows')) {
        commandLine 'cmd', '/c', 'asadmin.bat'
    } else {
        commandLine "./asadmin"
    }

    args "deploy", "--force=true", "${war.archivePath}"
}
```
