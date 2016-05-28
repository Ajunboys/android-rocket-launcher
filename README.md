# Android Rocket Launcher
Gradle plugin that adds tasks to your android modules for installing and launching all variants.

<p align="center">
<img src="extras/screenshot.png" />
</p>

## How to use
A *one liner* for your module's `build.gradle`
```groovy
plugins {
    id 'android-rocket-launcher' version '0.2.1'
}
```
Now, when you run `./gradlew tasks`, you'll see something like this:

```
openDemoDebug - Installs and opens DebugDemo build
openFreeDebug - Installs and opens DebugFree build
openPaidDebug - Installs and opens DebugPaid build
```

### Alternative
Copy-pasting this gradle task on every project

```groovy
// Running the APK on your Android Device
android.applicationVariants.all { variant ->
    if (variant.install) {
        tasks.create(name: "run${variant.name.capitalize()}", type: Exec,
                dependsOn: variant.install) {
            group = 'Run'
            description "Installs and Runs the APK for ${variant.description}."
            def getMainActivity = { file ->
                new XmlSlurper().parse(file).application.activity.find {
                    it.'intent-filter'.find { filter ->
                        return filter.action.find {
                            it.'@android:name'.text() == 'android.intent.action.MAIN'
                        } \
                                 && filter.category.find {
                            it.'@android:name'.text() == 'android.intent.category.LAUNCHER'
                        }
                    }
                }.'@android:name'
            }
            doFirst {
                def activityClass =
                        getMainActivity(variant.outputs.processManifest.manifestOutputFile)
                commandLine android.adbExe, 'shell', 'am', 'start', '-n',
                        "${variant.applicationId}/${activityClass}"
            }
        }
    }
}
```

It's a no brainer :smile:
