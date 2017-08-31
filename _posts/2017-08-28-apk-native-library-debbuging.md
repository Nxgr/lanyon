---
published: false
layout: post
title: Apk native library debbuging
---
## Context

During [HackIT CTF](https://ctf.com.ua/) last weekend I faced an APK reverse engineering task.
Getting the code from an APK is pretty straightforward nowadays, there are even online [services](http://www.javadecompilers.com/apk).

The code was basic, however it called a method from an embeded compiled library that was essential in order to get the final flag.

In this blog post, I am going to describe the steps I took to be able to interact and debug this library in an APK context.

***DISCLAIMER :*** _I am an absolute neophyte regarding APK and android studio. I may not be doing this the right way. However I managed to get it working, and I am writting this as a memento for myself and others facing the same issue._

## Requirements

* Android studio 2.3.3
* A native library extracted from an APK.
* SDK Platform : Android 7.1.1 (Nougat)

## Walkthrough

### 1. Install android studio (That may take some time)

### 2. Get a function name from the library you want to break on.

This name is necessary to name your new android studio project.

In my case it was :
```
Java_anative_hackit2017_com_apiclient_MainActivity_signature
```

This string can be splitted in 4 parts : 
* Standard prefix : "Java"
* Package name : "anative_hackit2017_com_apiclient"
* Class name : "MainActivity"
* exported name : "signature"

### 3. Create a new Android Studio Project
The most important part here is to set the package name right since it'll be used to resolve the function name. You can either toy with the Application name and Company domain, or set it manually.

![Create new project]({{site.baseurl}}/images/2017-08-28-apk-native-library-debbuging/new_project.png)

Then end the project creation wizard by clicking Next -> Next -> Next -> Finish.

If everything went alright, you just ended up in with a prefilled MainActivity.java in edition mode.
In case the class name extracted from the function name	isn't 'MainActivity' you'd have to create your own class.

### 4. Library import and name check.

Let's add this snippet to your MainActivity class :

```
    static {
        System.loadLibrary("LIBRARY_NAME");
    }
    public static native byte[] EXPORTED_NAME(byte[] bArr);
```
and replace LIBRARY_NAME and FUNCTION_NAME with their according values.

* LIBRARY_NAME : If your library filename is "libfoo.so", LIBRARY_NAME equals "foo".
* EXPORTED_NAME : If your function name is "Java_anative_hackit2017_com_apiclient_MainActivity_signature", EXPORTED_NAME equals "signature".

There is a basic way to check if you got everything allright so far : hover the EXPORTED_NAME in your .java file, and Android Studio should display the following tooltip :
```
Cannot resolve corresponding JNI function FUNCTION_NAME
```
with FUNCTION_NAME being Java_anative_hackit2017_com_apiclient_MainActivity_signature in my case.

### 5. Gradle script edition

For now, your project tree should look like this :
![project tree]({{site.baseurl}}/images/2017-08-28-apk-native-library-debbuging/project_tree.png)

Edit _build.gradle (Module:app)_ by double clicking it. Then paste this block into the main android block :
```
    sourceSets.main {
        jniLibs.srcDirs = ['Libs']
    }
```

Right after this edition, a banner should pop-up asking you to sync your project with the Gradle file. Click _Sync Now_. A wild _jniLibs_ directory just appeared in the project tree.

### 6. Library import

This step depends on the arch of the library you want to debug. In my case it was x86, but just adapt it according to the standard android naming scheme (x86 / x86_64 / mips / mips64 / armeabi / armeabi-v7a / arm64-v8a).
In my case the arch was "x86".

* Right click the _jniLibs_ folder in the project tree, then New -> Directory and name it according to the arch quoted above.
* Drag and drop to copy your library file into the freshly created folder.

***On this step you should be able to build your APK using the imported native Library. The following instructions are there to debug said library***

### 7. Setting up debugger
From the menu in the top banner, go to _Run -> Edit Configurations..._

Then switch to the Debugger tab, and select 'Native' in the 'Debug type' dropdown menu.
Last step is to add an LLDB Post Attach Commands, corresponding to a breakpoint onto the library function of your choice :
```
break set --name Java_anative_hackit2017_com_apiclient_MainActivity_signature
```
![debug config]({{site.baseurl}}/images/2017-08-28-apk-native-library-debbuging/debug_config.png)

### 8. Ready to launch

Launch your app in debug mode : _Run -> Debug (maj + f9)_.
You're going to need an virtual device if that's not already done, then you're good to go !

On the bottom of android studio screen you should be able to interact with a lldb console, that just breaked into the breakpoint you set up early on.
![lldb interface]({{site.baseurl}}/images/2017-08-28-apk-native-library-debbuging/lldb.png)


***Congratulations, the ball is on your side now. Happy reversing !***

## Useful link :

* [https://github.com/maaaaz/jnianalyzer](https://github.com/maaaaz/jnianalyzer)
* [https://lldb.llvm.org/lldb-gdb.html](https://lldb.llvm.org/lldb-gdb.html)
