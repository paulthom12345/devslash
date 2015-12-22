---
layout: post
title: Testing With Gradle
date: '2014-12-22 08:51:00'
---



I love Android.
I also think that testing is super duper.

For this reason I've always been a big fan of attempting to find ways to perform better testing. Whether this be through use of continuous integration, mocking or even _sweet as_ dependency injection coupled with Gradle.

## Simple Gradle Example
Gradle allows very easily to allow for different flavours or build configurations. A very small example that shows a small change in builds is below. It is an exerpt from a larger Android `build.gradle` that will set various build configurations based on the build type.

~~~
buildTypes {
    release {
        signingConfig signingConfigs.release
        minifyEnabled false
    }
    debug {
        minifyEnabled false
        debuggable true
}
~~~
{: .language-groovy}

A quick look at Gradle will make it fairly obvious that we're using a whole lot less boilerplate than the standard Maven POM. We are managing to leverage the Groovy DSL to be more expressive in fewer characters. Surprisingly XML isn't the best thing for all things configuration based. 

The above buildTypes will set Android debugging flags. Based on the type of release that is being done, it will switch on debugging and sign the app correctly.

## Adding some Android flair
Gradle is a powerful language that means that you're able to very easily perform some simple actions that will affect the global state of your program. The following code example shows off a ways to turn on and off the high logging levels. 

~~~
release {
    buildConfigField "boolean", "HIGH_LOGGING", "false"
}
debug {
    buildConfigField "boolean", "HIGH_LOGGING", "true"
}
~~~
{: .language-groovy}

This build config field is created in the form

>buildConfigField &lt;type&gt;, &lt;name&gt;, &lt;value&gt;

We will then be able to use this at any point as a static field in our program via <i>BuildConfig.VARIABLE_NAME</i>. This makes it suddenly incredibly easy to create a logging stub that will allow for easy debug logging. *Note that this part of the Android Gradle plugin and does not work out of the box for a Java build*. 


~~~
public class Logger{

    public static void log( Level logLevel, String message ){
        if( logLevel >= Level.WARN ){
            log.log( logLevel, message );
        }
        else if( BuildConfig.HIGH_LOGGING )
            log.log( logLevel, message );
        }
    }
}
~~~
{: .language-java}

Suddenly we will have a call to a logger that will always be able to choose how much to log based on the build type. This means whenever you release your apk you will never have a problem of creating a bucket of log files that are over the top but there is ZERO extra configuration that is required when developers are attempting to debug on their own devices.

But we don't have to stop there. Gradle will also let us perform more advanced actions such as those that will include or exclude whole source packages automatically. This allows for you to perform fantastically simple mocking of objects that would otherwise be a pain to switch in and out.

## Controlling source packages with Gradle
Android isn't always easy to perform testing on. This is generally for the same reason that UI testing in the web isn't exactly simple either. For this people generally use products such as HTMLUNIT or other selenium test runners. If you're after something similar [Robolectric](http://robolectric.org/) allows you to perform Unit testing with the ability to test as though a UI existed.

Sometimes though that's not enough. When testing apps it's common that the only way that you can find exactly what you need to improve by continually attempting to use your app. This is known as dogfooding and is used extensively in software companies. We can leverage Gradle to allow us to create different APK files for the internal development team. We can combine build types and build flavours to allow us to create builds specifically for tablets, for smaller phones, for paid versions, for free versions. This without too much hair falling out. So lets demonstrate how it would be possible to create app versions that contained debug/release only code.

### Getting build specific control
Gradle is run through Groovy. This means that we can technically do anything that we can do in Groovy. Suddenly this means our build system can have logic rather than just a lot of configuration steps. Instead of a barrage of tags all over the place we can instead create dynamic builds, even including user input. This would be fantastic if you wanted to tag a build with a special name. We'd be able to do something like:

~~~
android {
    defaultConfig {
        applicationVariants.all { variant ->
            def file = variant.outputFile
            variant.outputFile = new File(file.parent, "supercoolfilename.apk")        
        }
    }
}
~~~
{: .language-groovy}

Which you can extend this to get the user input to do the following.

~~~
def console = System.console()

def newname = console.readLine("Set the APK name >")

android {
    defaultConfig {
        applicationVariants.all { variant ->
            def file = variant.outputFile
            variant.outputFile = new File(file.parent, newname + ".apk")
        }
    }
}
~~~
{: .language-groovy}

We will now get a console output that will ask us first to confirm the apk name before it actually will get built. But this is really not something that we'll want unless we're releasing (maybe like what Google does with android for mako and other build types). So you can easily extend this to make it totally based on the build version. ([Reference](http://stackoverflow.com/questions/25104323/how-to-customize-the-apk-file-name-for-product-flavors]))

~~~
def console = System.console()
android {
    applicationVariants.all { variant ->
        if (variant.buildType.name.equals("debug")) {
            def apk = variant.outputFile;
            variant.outputFile = new File(apk.parentFile, "debugAndroidApp.apk");
        } else if(variant.buildType.name.equals("release")) {
            def apk = variant.outputFile;
            variant.outputFile = new File(apk.parentFile, "AndroidApp.apk");
        }
    }
}
~~~
{: .language-groovy}

Suddenly we have a way to have build names for each of our releases all in the same number of lines that the standard Maven system would have managed to add two dependencies. 

### Using Gradle To Deploy Alternative Files

So up to this point we've done some fairly small things with the Gradle system. It will actually allow us to override class implementations so that we may go ahead and inject different requirements based on our test system. In the following example we're going to use it to inject a sidebar into a fake android app that will allow us to change what we inject into our app. Through this we're able to add things such as a Http Service that fails requests 50% of the time. 

To start off with, we'll have a look at the directory structure to create this example:

~~~
src  -\
     - main
     - release
     - debug
~~~

All code within the `main` folder will always be compiled for use. This is code that will exist in both our release and debug builds. The `release` and `debug` folders will each be compiled when the relevant gradle build is taking place. The generic way that this is used is **android.sourceSets.&lt;flavour&gt;&lt;ReleaseType&gt;**, so in our case we do not have a flavour so we just use the release type. Therefore **android.sourceSets.release** and **android.sourceSets.debug** are the two that we are using. This corresponds to `src/release` and `src/debug`. Every instance of these extends the standard **android.sourceSets.main** which will be in `src/main`.

To leverage this we will create, a <code>MainActivityExtender</code> which will be implemented in both release types. This will be used to add extra functionality to the <code>MainActivity</code> in the main source set.  A comparison of the files will show the small difference that lies between them.

***MainActivity.class***

~~~
class MainActivity extends Activity {

    public SideBarManager(Context ctx, View parentView){
        // Inflate the new View
        setContentView(R.layout.MainActivity);
        
        runDebugPostHook();
    }
    
    public void runPostHook(){
        // Nothing exists here in the MainActivity version
    }
}
~~~
{: .language-java}

***/src/debug/net/devlsash/MainActivityExtender.class***

~~~
class SidebarManagerExtender extends MainActivity {
    
    @Override
    public void runPostHook() {
        // Now we should create new details based on the 
        // fact that we're running in Debug mode
        
    }
    
}
~~~
{: .language-java}

***/src/release/net/devlsash/MainActivityExtender.class***

~~~
class SidebarManagerExtender extends MainActivity {
    // We will not override runPostHook here
}
~~~
{: .language-java}

***Android Manifest***

~~~
<activity
    android:name="net.devslash.testapp.RunningMainActivity"
    android:label="@string/app_name"
    android:windowSoftInputMode="adjustResize|stateHidden">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
~~~
{: .language-xml}

This may all seem a bit over the top. Couldn't we just do as we did before? Set a flag and perform an if statment to do the same thing? That's completely correct. This is merely what I'd call cleaner and less prone to errors or possible leaking of debug features to the final application. 

One thing not demonstrated here is the ability to override xml files. This provides extra utility due to the fact that instead of the following

~~~
Button someButton; 
if(Flag.DEBUG){
    setContentView(R.layout.debugMainActivityLayout);
    someButton = (Button) findViewById(R.id.debugButton);
} else {
    setContentView(R.id.MainActivityLayout);
}
// Then possibly later on
if(someButton != null) {
    // Do something here
}
~~~
{: .language-java}

With the Gradle source replacement/swapping in we're able to simplify it to

~~~
setContentView(R.layout.MainActivityLayout);
~~~
{: .language-java}

Then you can place different xml files in the directory *src/main/res/layout/MainActivity.xml* and *src/debug/res/layout/MainActivity.xml*. Based on the gradle build type we will override the *MainActivity.xml* with our debug version whenever it is built. 

Then leveraging what we did before in regards to having an extended `MainActivity` we would be able to access the debug only parts of the XML in our post hook. 

Overall Gradle gives a large amount of build configuration steps that otherwise would be a large pain or just take a really long time to do. Gradle's adoption for Android has progressed Gradle a long way as development has continued to move very quickly for a long time. For this reason it's entirely possible that what I've written will be out of date shortly. If you notice this please do leave a comment below and I'll fix up the inaccuracies. 

