---
author: Rob
date: 2015-10-18 13:55:23+00:00
layout: post
slug: gradle-publish-library
title: Publish your Java libraries to a Maven repository with Gradle
categories:
- Gradle
- Java
---

## Introduction
A while ago I was working on a set of libraries that should be shared among several different projects. Most of these projects are based on Gradle and are using Maven repositories for the dependency management. Although the Gradle documentation describes how you can push versions of your library to a repository, I had to collect my information from several locations and combine them into one working solution. So maybe this working example is useful for others too.

Earlier Gradle had an upload task but now there's the incubating maven-publish plugin which is a lot easier. More information can be found in the documentation: [Gradle docs](https://docs.gradle.org/current/userguide/publishing_maven.html)

## Publish
Publishing is easy now, just one task to be executed, named publish:
``` bash
gradle publish
```

I'm publishing to a private [Apache Archiva](https://archiva.apache.org) installation but this can be any Maven repository of course.

## My build.gradle file
``` java
apply plugin: 'java'
apply plugin: 'maven-publish'

version '1.0'
group 'nl.tigrou.lib'
sourceCompatibility = 1.7

jar {
  manifest {
    attributes 'Implementation-Title': 'Some library',
        'Implementation-Version': version,
        'Built-By': System.getProperty('user.name'),
        'Built-Date': new Date(),
        'Built-JDK': System.getProperty('java.version'),
        'Built-Gradle': gradle.gradleVersion
  }
}

publishing {
  publications {
    mavenJava(MavenPublication) {
      version version
      groupId group
      artifactId 'some-lib'

      from components.java

      artifact sourceJar {
        classifier "sources"
      }
    }
  }

  repositories {
    maven {
      name 'Your own repository'
      url 'http://private.repo.com'
      credentials {
        username 'user'
        password 'password'
      }
    }
  }
}


task sourceJar(type: Jar) {
  from sourceSets.main.allJava
}
```
