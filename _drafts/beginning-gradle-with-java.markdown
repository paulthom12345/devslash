---
layout: post
title: Beginning Gradle with Java
---

_Compare with javac for a larger project_
Gradle has come along relatively recently. The first version was released back in 2009. The project saw minimal change until 2012 when work started in earnest on the project. Google has been a major proponent to Gradle as it is now the build runner for Android Studio, their Android specific IDE. 

Gradle gets around a lot of the build specific annoyances in much the same way that Ant or Maven have been doing for years. Gradle attempts to extend on what these tools have been doing as well as attempting to make the initial process and maintaining the build script a lot easier than it's XML based counterparts.

###If you're new to this
If you've used/enjoy using build systems before skip to the next section

If gradle is the first chance you've had at using a build system such as gradle/maven/ant then you might be a bit curious as to the reason that you'd want to use one. This is often the case when you're starting out and are not creating projects that are super complex or have a lot of dependencies. It's still often worth using something such as gradle before too long though. I'll show you why with a few examples. 

**For Example** whether or not you are working yet as a developer you might be having to perform group work through university or even just online for fun with friends. When doing so it's often really nice to be able to have everyone have  a consistent build experience. 

It's possible that your project relies on a project such as Google's GSON to parse JSON strings to Objects. When you downloaded GSON you downloaded the latest version. Unfortunately your group memeber downloaded GSON ages ago and so when building your code they find that it fails. Not due to their being any real technical errors but just the fact that you have two separate versions of the library. 

This would be fixed trivially with a build system via creating dependencies that are resolved when building your application. Specifically in gradle you'd add the line:
```language-groovy
compile 'com.google.code.gson:gson:2.3.1'
```
and suddenly whenever you ran the build Gradle would ensure that GSON was available and on version 2.3.1.

###Minimum Working Example

_Compare with the other tools_
To get Gradle running for a very simple Java application is as easy as the following

```
apply plugin 'java'
```

The lack of any sort of configuration is due to Gradle's encouragment of convention over configuration. Applying the standard Java plugin will set up Gradle to expect the sources/tests and builds to be put in the standard Maven layout. This can all be changed of course. But to start our build of the able we just need to go
```
gradle build
```

This will run through a series of build steps. First compiling, assembling resources, testing and then finally building the artifacts. 



###Moving from Maven to Gradle
_Provide examples of how you change from Maven to Gradle_
Moving from Maven is a relatively painless process. This is due to the fact that 

###Multi Project Gradle Setup