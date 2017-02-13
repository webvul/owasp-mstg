## Android Platform Overview

(... TODO ...)

### Android Architecture and Security Mechanisms

-- TODO :IPC mechanism and Reference monitor, Binder, Discretionary - Mandatory Access Control / UID - GID / Filesystem, Applicative Architecture of an application : Permissions & Manifest, Application Signing. May be a part of Static / Dynamic Analysis chapter : each security mechanism efficiency can be checked at a given phase. --

Android is an open source platform that can be found nowadays on many devices:

* Mobile Phones and Tablets
* Wearables
* "Smart" devices in general like TVs

It also offers an application environment that supports not only pre-installed applications on the device, but also 3rd party applications that can be downloaded from marketplaces like Google Play.

The software stack of Android comprises of different layers, where each layer is defining certain behavior and offering specific services to the layer above.

![Android Software Stack](https://source.android.com/security/images/android_software_stack.png)

On the lowest level Android is using the Linux Kernel where the core operating system is built up on. The hardware abstraction layer defines a standard interface for hardware vendors. HAL implementations are packaged into shared library modules (.so files). These modules will be loaded by the Android system at the appropriate time. The Android Runtime consists of the core libraries and the Dalvik VM (Virtual Machine). Applications are most often implemented in Java and compiled in Java class files and then compiled again into the dex format. The dex files are then executed within the Dalvik VM.
In the next image we can see the differences between the normal process of compiling and running a typical project in Java vs the process in Android using Dalvik VM.

![Java vs Dalvik](/Document/images/Chapters/0x04a/java_vs_dalvik.png)

With Android 4.4 the successor of Dalvik VM was introduced, called Android Runtime (ART).

ART, a.k.a Android RunTime, was born with KitKat (Android 4.4). However, it has really been set for general use only in Lollipop (Android 5.0, API 21) in November 2014, where it replaced Dalvik : in KitKat, ART was only available in the 'Developer' menu to those who wanted to try it explicitely. When no user action was done to modify the normal behaviour of the mobile, Dalvik was used.

In Android, applications are executed into their own environnement in a Virtual Machine (VM), that was called Dalvik, located in the RunTime environnement. Each VM emulates the whole mobile and gives access to relevant resources from the Linux kernel while controlling this access: applications do not have direct access to hardware resources, and their execution environnements are therefore separate from each other. This allows fine-grained control over resources and applications: for instance, when an application crashes, it does not prevent other applications from working and only their environnement and the application itself have to be restarted. Also, the fact applications are not run directly on the mobile hardware allow the use of the same application (same bytecode) on different hardwares as the VM emulates a common hardware for the application. At the same time, the VM controls the maximum amount of resources provided to an application, preventing one application from using all resources while leaving only few resources to others.
In Android, applications are installed as bytecode (.dex files, see "Android Application Overview" section). In Dalvik, this bytecode was compiled at execution time into machine language suiting the current processor: such a mechanism is known as Just In Time (JIT). However, this means that such compiling is done everytime an application is executed on a given mobile. As a consequence, Dalvik has been improved to compile an application only once, at installation time (the principle is called AOT, a.k.a. Ahead  Of Time): ART was born, and compilation was required only once, saving precious time at execution time (the execution time of an application may be divided by 2). Another benefit was that ART was consuming less power than Dalvik, allowing the user to use its battery for more time..

#### Android Users and Groups

Android is a system based on Linux, however it does not deal with users the same way Linux does. It does not have a _/etc/password_ file describing a list of Linux users in the system. Instead Android contains a fixed set of users and groups and they are used to isolate processes and grant permissions.
File [system/core/include/private/android_filesystem_config.h](http://androidxref.com/7.1.1_r6/xref/system/core/include/private/android_filesystem_config.h) shows the complete list of the predefined users and groups mapped to numbers.
File below depicts some of the users defined for Android Nougat:

~~~
    #define AID_ROOT             0  /* traditional unix root user */

    #define AID_SYSTEM        1000  /* system server */
	...
    #define AID_SHELL         2000  /* adb and debug shell user */
	...
    #define AID_APP          10000  /* first app user */
	...
~~~




### Understanding Android Apps

#### Communication with the Operating System

In Android, applications are developed in Java, and the Operating System offers an API to interact with system resources: communication media (Wifi, Bluetooth, NFC, ...), files, cameras, geolocation (GPS), microphone, ... . System resources cannot be accessed directly, and APIs mediate the access for the user. At the time of writting this guide, the current version of Android API is 7.1.1 Nougat, API 25. 

APIs have evolved a lot since Android creation (the first release happened in September 2008). Early versions are not supported anymore; however, Android is a living project and new features and bug fixes are periodically made. 

Noteworthy recent API versions are:
- Android 4.2 Jelly Bean (API 16) in November 2012 (introduction of SELinux)
- Android 4.3 Jelly Bean (API 18) in July 2013 (SELinux becomes enabled by default)
- Android 4.4 KitKat (API 19) in October 2013 (several new APIs and ART is introduced)
- Android 5.0 Lollipop (API 21) in November 2014 (ART by default and many other new features)
- Android 6.0 Marshmallow (API 23) in October 2015 (many new features and improvements, including granting fine-grained permissions at run time and not all or nothing at installation time)
- Android 7.0 Nougat (API 24) in August 2016 (new JIT compiler on ART)

After being developed, applications can be installed on mobiles from a variety of sources: locally through USB, from Google official store (Google Play) or from alternate stores. 

#### App Folder Structure

Android applications installed (from Google Play Store or from external sources) are located at /data/app/. Since this folder cannot be listed without root, another way has to be used to get the exact name of the apk. To list all installed apks, the Android Debug Bridge (adb) can be used. ADB allows a tester to directly interact with the real phone, e.g., to gain access to a console on the device to issue further commands, list installed packages, start/stop processes, etc.
To do so, the device has to have USB-Debugging enabled (under developer settings) and has to be connected via USB.
Once USB-Debugging is enabled, the connected devices can be viewed with the command

```bash
$ adb devices
List of devices attached
BAZ5ORFARKOZYDFA	device
```

Then the following command lists all installed apps and their locations:

```bash
$ adb shell pm list packages -f
package:/system/priv-app/MiuiGallery/MiuiGallery.apk=com.miui.gallery
package:/system/priv-app/Calendar/Calendar.apk=com.android.calendar
package:/system/priv-app/BackupRestoreConfirmation/BackupRestoreConfirmation.apk=com.android.backupconfirm
```

To pull one of those apps from the phone, the following command can be used:

```bash
$ adb pull /data/app/com.google.android.youtube-1/base.apk
```

This file only contains the “installer” of the application, meaning this is the app the developer uploaded to the market.
The local data of the application is stored at /data/data/PACKAGE-NAME and has the following structure:

```bash
drwxrwx--x u0_a65   u0_a65            2016-01-06 03:26 cache
drwx------ u0_a65   u0_a65            2016-01-06 03:26 code_cache
drwxrwx--x u0_a65   u0_a65            2016-01-06 03:31 databases
drwxrwx--x u0_a65   u0_a65            2016-01-10 09:44 files
drwxr-xr-x system   system            2016-01-06 03:26 lib
drwxrwx--x u0_a65   u0_a65            2016-01-10 09:44 shared_prefs
```

* **cache**: This location used to cache application data on runtime including WebView caches.
* **code_cache**: TBD
* **databases**: This folder stores sqlite database files generated by the application at runtime, e.g. to store user data
* **files**: This folder is used to store files that are created in the App when using the internal storage.
* **lib**: This folder used to store native libraries written in C/C++. These libraries can have file extension as .so, .dll (x86 support). The folder contains subfolders for the platforms the app has native libraries for:
   * armeabi: compiled code for all ARM based processors only
   * armeabi-v7a: compiled code for all ARMv7 and above based processors only
   * arm64-v8a: compiled code for all ARMv8 arm64 and above based processors only
   * x86: compiled code for x86 processors only
   * x86_64: compiled code for x86_64 processors only
   * mips: compiled code for MIPS processors only
* **shared_prefs**: This folder is used to store the preference file generated by application on runtime to save current state of application including data, configuration, session, etc. The file format is XML.

#### APK Structure

An application on Android is a file with the extension .apk. This file is a signed zip-file which contains different files for the bytecode, assets, etc. When unzipped the following directory structure can be identified:

```bash
$ unzip base.apk
$ ls -lah
-rw-r--r--   1 sven  staff    11K Dec  5 14:45 AndroidManifest.xml
drwxr-xr-x   5 sven  staff   170B Dec  5 16:18 META-INF
drwxr-xr-x   6 sven  staff   204B Dec  5 16:17 assets
-rw-r--r--   1 sven  staff   3.5M Dec  5 14:41 classes.dex
drwxr-xr-x   3 sven  staff   102B Dec  5 16:18 lib
drwxr-xr-x  27 sven  staff   918B Dec  5 16:17 res
-rw-r--r--   1 sven  staff   241K Dec  5 14:45 resources.arsc
```

* **AndroidManifest.xml**: Contains the definition of application’s package name, target and min API version, application configuration, application components, user-granted permissions, etc.
* **META-INF**: This folder contains metadata of application:
   * MANIFEST.MF: stores hashes of application resources.
   * CERT.RSA: The certificate(s) of the application.
   * CERT.SF: The list of resources and SHA-1 digest of the corresponding lines in the MANIFEST.MF file.
* **assets**: A directory containing applications assets (files used within the Android App like XML, Java Script or pictures) which can be retrieved by the AssetManager.
* **classes.dex**: The classes compiled in the DEX file format understandable by the Dalvik virtual machine/Android Runtime. DEX is Java Byte Code for Dalvik Virtual Machine. It is optimized for running on small devices.
* **lib**: A directory containting libraries that are part of the APK, for example 3rd party libraries that are not part of the Android SDK.
* **res**: A directory containing resources not compiled into resources.arsc.
* **resources.arsc**: A file containing precompiled resources, such as XML files for the layout.

Since some resources inside the APK are compressed using non-standard algorithms (e.g. the AndroidManifest.xml), simply unzipping the file does not reveal all information. A better way is to use the tool apktool to unpack and uncompress the files. The following is a listing of the the files contained in the apk:

```bash
$ apktool d base.apk
I: Using Apktool 2.1.0 on base.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: /Users/sven/Library/apktool/framework/1.apk
I: Regular manifest package...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
$ cd base
$ ls -alh
total 32
drwxr-xr-x    9 sven  staff   306B Dec  5 16:29 .
drwxr-xr-x    5 sven  staff   170B Dec  5 16:29 ..
-rw-r--r--    1 sven  staff    10K Dec  5 16:29 AndroidManifest.xml
-rw-r--r--    1 sven  staff   401B Dec  5 16:29 apktool.yml
drwxr-xr-x    6 sven  staff   204B Dec  5 16:29 assets
drwxr-xr-x    3 sven  staff   102B Dec  5 16:29 lib
drwxr-xr-x    4 sven  staff   136B Dec  5 16:29 original
drwxr-xr-x  131 sven  staff   4.3K Dec  5 16:29 res
drwxr-xr-x    9 sven  staff   306B Dec  5 16:29 smali
```

* **AndroidManifest.xml**: This file is not compressed anymore and can be openend in a text editor.
* **apktool.yml** : This file contains information about the output of apktool.
* **assets**: A directory containing applications assets (files used within the Android App like XML, Java Script or pictures) which can be retrieved by the AssetManager.
* **lib**: A directory containting libraries that are part of the APK, for example 3rd party libraries that are not part of the Android SDK.
* **original**: This folder contains the MANIFEST.MF file which stores meta data about the contents of the JAR and signature of the APK. The folder is also named as META-INF.
* **res**: A directory containing resources not compiled into resources.arsc.
* **smali**: A directory containing the disassembled Dalvik Bytecode in Smali. Smali is a human readable representation of the Dalvik executable.

#### Linux UID/GID of Normal Applications

When a new application gets installed on Android a new UID is assigned to it. Generally apps are assigned UIDs in the range of 10000 (_AID_APP_) and 99999. Android apps also receive a user name based on its UID. As an example, application with UID 10188 receives the user name _u0_a188_. 
If an app requested some permissions and they are granted, the corresponding group ID is added to the process of the application.
For example, the user ID of the application below is 10188, and it also belongs to group ID 3003 (_inet_) that is the group related to _android.permission.INTERNET_ permission. The result of the `id` command is shown below:

```
$ id
uid=10188(u0_a188) gid=10188(u0_a188) groups=10188(u0_a188),3003(inet),9997(everybody),50188(all_a188) context=u:r:untrusted_app:s0:c512,c768
```

The relationship between group IDs and permissions are defined in the file [frameworks/base/data/etc/platform.xml](http://androidxref.com/7.1.1_r6/xref/frameworks/base/data/etc/platform.xml)
```
<permission name="android.permission.INTERNET" >
	<group gid="inet" />
</permission>

<permission name="android.permission.READ_LOGS" >
	<group gid="log" />
</permission>

<permission name="android.permission.WRITE_MEDIA_STORAGE" >
	<group gid="media_rw" />
	<group gid="sdcard_rw" />
</permission>
```

An important element to understand Android security is that all applications have the same level of privileges: both native and third-party applications are built on the same APIs and are run in similar environnements. Also, all applications are executed not as 'root', but with the user level of privileges. That means that, basically, applications cannot perform some actions or access some parts of the file system. In order to be able to execute an application with 'root' privileges (inject packets in a network, run interpreters like for Python, ...), mobiles need to be rooted.

#### The App Sandbox

Applications are executed in the Android Application Sandbox that enforces isolation of application data and code execution from other applications on the device, that adds an additional layer of security.

When installing new applications (From Google Play or External Sources), a new folder is created in the filesystem in the path /data/data/\<package name>. This folder is going to be the private data folder for that particular application.

Since every application has its own unique Id, Android separates application data folders configuring the mode to _read_ and _write_ only to the owner of the application. 

![Sandbox](/Document/images/Chapters/0x04a/Selection_003.png)

In this example, the Chrome and Calendar app are completly segmented with different UID and different folder permissions.

We can confirm this my looking at the filesystem permissions created for each folder:
```
drwx------  4 u0_a97              u0_a97              4096 2017-01-18 14:27 com.android.calendar
drwx------  6 u0_a120             u0_a120             4096 2017-01-19 12:54 com.android.chrome
```
However, if two applications are signed with the same certificate and explicitly share the same user ID (by including the _sharedUserId_ in their _AndroidManifest.xml_) they can access each other data directory.
An example how this is achieved in Nfc application:
```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	package="com.android.nfc"
	android:sharedUserId="android.uid.nfc">
```

The Android Framework is creating an abstraction layer for all the layers below, so developers can implement Android Apps and can utilize the capabilities of Android without deeper knowledge of the layers below. It also offers a robust implementation that offers common security functions like secure IPC or cryptography.

#### App Components

Android applications are made of several high-level components that make up their architectures. The main components are activities, fragments, intents, broadcast receivers, content providers and services. All these elements are provided by the Android operating system in the form of predefined classes available through APIs. 

##### Application lifecycle

Android applications have their own lifecycles, that is under the control of the operating system. Therefore, applications need to listen to state changes and must be able to react accordingly. For instance, when the system needs resources, applications may be killed. The system selects the ones that will be killed according to the application priority: active applications have the highest priority (actually the same as Broadcast Receivers), followed by visible ones, running services, background services, and last useless processes (for instance applications that are still open but not in use since a significant time). 

Applications implement several event managers to handle events: for example, the onCreate handler implements what is to be done on application creation and will be called on that event. Other managers include onLowMemory, onTrimMemory and onConfigurationChanged.

##### Manifest files

Every application must have a manifest file, which embeds content in the XML format. The name of this file is standardized as AndroidManifest.xml and is the same for every application. It is located in the root tree of the .apk file in which the application is published. 

A Manifest file describes the application structure as well as its exposed components (activities, services, content providers and intent receivers) and the rights required by the application (required permissions are listed, and filters can be implemented to refine the way the application will interact with the outside world). It also contains general metadata about the application, like its icon, its version number and the theme is uses for User Experience (UX). It may also list other information like the APIs it is compatible with (minimal, targeted and maximal SDK version) and the kind of memory is can be installed in (external or internal). 

Here is an example of a manifest file, including the package name (the convention is to use a url in reverse order, but any string can be used), the application version, relevant SDKs, required permissions, exposed content providers, used broadcast receivers with intent filters, and the description of the application itself with its activities:
```
<manifest 
	package="com.owasp.myapplication"
	android:versionCode="0.1" >
	
	<uses-sdk android:minSdkVersion="12"
		android:targetSdkVersion="22"
		android:maxSdkVersion="25" />
	
	<uses-permission android:name="android.permission.INTERNET" />
	
	<provider
		android:name="com.owasp.myapplication.myProvider"
		android:exported="false" />

	<receiver android:name=".myReceiver" >
		<intent-filter>
			<action android:name="com.owasp.myapplication.myaction" />
		</intent-filter>
	</receiver>
	
	<application
		android:icon="@drawable/ic_launcher"
		android:label="@string/app_name"
		android:theme="@style/Theme.Material.Light" >
		<activity
			android:name="com.owasp.myapplication.MainActivity" >
			<intent-filter>
				<action android:name="android.intent.action.MAIN" />
			</intent-filter>
		</activity>
	</application>
</manifest>
```

Manifests are text files and can be edited with any text editor. However, Android Studio, Google prefered IDE for Android development, embeds a graphical editor.

A lot more useful options can be added to manifest files. The reader is invited to refer to Android official documentation available at https://developer.android.com/index.html for more details: even if numerous examples are given in this section, in no way this guide should be considered as a reference on the topic.

##### Activities

Activities make up the visible part of any application. More specifically, one activity exists per screen (e.g. user interface) in an application: for instance, applications that have 3 different screens implement 3 different activities, where the user can interact with the system (get and enter information). Activities are declared by extending the Activity class; they contain all user interface elements: fragments, views and layouts.

Activities implement manifest files. Each activity needs to be declared in the application manifest with the following syntax:
```
<activity android:name=".ActivityName>
</activity>
```
When activities are not declared in manifests, they cannot be displayed and would raise an exception.

In the same way as applications do, activities also have their own lifecycles and need to listen to system changes to be able to handle them accordingly. Activities can have the following states: active, paused, stopped and inactive. These states are managed by Android operating systems. Accordingly, activities can implement the following event managers:
- onCreate
- onSaveInstanceState
- onStart
- onResume
- onRestoreInstanceState
- onPause
- onStop
- onRestart
- onDestroy
An application may not explicitely implement all event managers; in that situation, default actions are taken. However, usually at least the onCreate manager is overriden by application developers, as this is the place where most user interface components are declared and initialised. onDestroy may be overridden as well in case some resources need to be explicitely released (like network connections or connections to databases) or specific actions need to take place at the end of the application. 

##### Fragments

A Fragment represents a behavior or a portion of user interface in an Activity.

##### Intents

Intents are messaging components used between applications and components. They can be used by an application to send information to its own components (for instance, start inside the application a new activity) or to other applications, and may be received from other applications or from the operating system. Intents can be used to start activities or services, run an action on a given set of data, or broadcast a message to the whole system. They are a convenient way to decouple components.

There are two kinds of intents : explicit and implicit. 
- explicit intents exactly name the activity class to be used. For instance:
```
	Intent intent = new Intent(this, myActivity.myClass);
```
- implicit intents are sent to the system with a given action to perform on a given set of data ("http://www.example.com" in our example below). It is up to the system to decide which application or class will perform the corresponding service. For instance:
```
	Intent intent = new Intent(Intent.MY_ACTION, Uri.parse("http://www.example.com"));
```

Android uses intents to broadcast messages to applications, like an incoming call or SMS, important information on power supply (low battery for example) or network changes (loss of connection for instance). Extra data may be added to intents (through putExtra / getExtras). 

Here is a short list of intents from the operating system. All constants are defined in the Intent class, and the whole list can be found in Android official documentation:
- ACTION_CAMERA_BUTTON
- ACTION_MEDIA_EJECT
- ACTION_NEW_OUTGOING_CALL
- ACTION_TIMEZONE_CHANGED

In order to improve security and privacy, a Local Broadcast Manager exists and is used to send and receive intents inside an application, without having them sent to the outside world (other applications or operating system). This is very useful to guarantee sensitive or private data do not leave the application perimeter (geolocation data for instance). 

##### Broadcast Receivers

Broadcast Receivers are components that allow to receive notifications sent from other applications and from the system itself. This way, applications can react to events (either internal, from other applications or from the operating system). They are generally used to update a user interface, start services, update content or create user notifications. 

Broadcast Receivers need to be declared in the Manifest file of the application. Any Broadcast Receiver must be associated to an intent filter in the manifest to specify which actions it is meant to listen with which kind of data. If they are not declared, the application will not listen to broadcasted messages. However, applications do not need to be started to receive intents: they are automatically started by the system when a relevant intent is raised. 

An example of declaring a Broadcast Receiver with an Intent Filter in a manifest is:
```
	<receiver android:name=".myReceiver" >
		<intent-filter>
			<action android:name="com.owasp.myapplication.MY_ACTION" />
		</intent-filter>
	</receiver>
```

When receiving an implicit intent, Android will list all applications that have registered a given action in their filters. If more than one application is matching, then Android will list all those applications and will require the user to make a selection.

An interesting feature concerning Broadcast Receivers is that they be affected a priority; this way, an intent will be delivered to all receivers authorized to get them according to their priority. 

A Local Broadcast Manager can be used to make sure intents are received only from the internal application, and that any intent from any other application will be discarded. This is very useful to improve security.

##### Content Providers

Android is using SQLite to store data permanently: as it is in Linux, data is stored in files. SQLite is an open-source, light and efficient technology for relational data storage that does not require much processing power, making it ideal for use in the mobile world. An entire API is available to the developer with specific classes (Cursor, ContentValues, SQLiteOpenHelper, ContentProvider, ContentResolver, ...). 
SQLite is not run in a separate process from a given application, but it is part of it. 
By default, a database belonging to a given application is only accessible to this application. However, Content Providers offer a great mechanism to abstract data sources (including databases, but also flat files) for a more easy use in an application; they also provide a standard and efficient mechanism to share data between applications, including native ones. In order to be accessible to other applications, content providers need to be explicitely declared in the Manifest file of the application that will share it. As long as Content Providers are not declared, they are not exported and can only be called by the application that creates them.

Content Providers are implemented through a URI addressing scheme: they all use the content:// model. Whatever the nature of sources is (SQLite database, flat file, ...), the addressing scheme is always the same, abstracting what sources are and offering a unique scheme to the developer. Content providers offer all regular operations on databases: create, read, update, delete. That means that any application with proper rights in its manifest file can manipulate the data from other applications.

##### Services

Services are components provided by Android operating system (in the form of the Service class) that will perform tasks in the background (data processing, start intents and notifications, ...), without presenting any kind of user interface. Services are meant to run processing on the long term. Their system priorities are lower than the ones active applications have, but are higher than inactive ones. As such, they are less likely to be killed when the system needs resources; they can also be configured to start again automatically when enough resources become available in case they get killed. Activities are executed in the main application thread. They are great candidates to run asynchronous tasks. 

##### Permissions
Because Android applications are installed in a sandbox and initially it does not have access to neither user information nor access to system components (such as using the camera or the microphone), it provides a system based on permissions where the system has a predefined set of permissions for certain tasks that the application can request.
As an example, if you want your application to use the camera on the phone you have to request the camera permission. 
On Android versions before Marshmallow (API 23) all permissions requested by an application were granted at installation time. From Android Marshmallow onwards the user have to approve some permissions during application execution.

###### Protection Levels
Android permissions are classified in four different categories based on the protection level it offers.
- *Normal*: Is the lower level of protection, it gives applications access to isolated application-level feature, with minimal risk to other applications, the user or the system. It is granted during the installation of the App. If no protection level is specified, normal is the default value.
Example: `android.permission.INTERNET`
- *Dangerous*: This permission usually gives the application control over user data or control over the device that impacts the user. This type of permissoin may not be granted at installation time, leaving to the user decide whether the application should have the permission or not.
Example: `android.permission.RECORD_AUDIO`
- *Signature*: This permission is granted only if the requesting app was signed with the same certificate as the application that declared the permission. If the signature matches, the permission is automatically granted.
Example: `android.permission.ACCESS_MOCK_LOCATION`
- *SystemOrSignature*: Permission only granted to applications embedded in the system image or that were signed using the same certificated as the application that declared the permission.
Example: `android.permission.ACCESS_DOWNLOAD_MANAGER`

###### Requesting Permissions
Applications can request permissions of protection level Normal, Dangerous and Signature by inserting the XML tag `<uses-permission />` to its Android Manifest file.
The example below shows an AndroidManifes.xml sample requesting permission to read SMS messages:
```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.permissions.sample" ...>

    <uses-permission android:name="android.permission.RECEIVE_SMS" />
    <application>...</application>
</manifest>
```
This will enable the application to read SMS messages at install time (before Android Marshmallow - 23) or will enable the application to ask the user to allow the permission at runtime (Android M onwards).

###### Declaring Permissions
Any application is able to expose its features or content to other applications installed on the system. It can expose the information openly or restrict it some applications by declaring a permission.
The example below shows an application declaring a permission of protection level *signature*.
```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.permissions.sample" ...>

    <permission
    android:name="com.permissions.sample.ACCESS_USER_INFO"
    android:protectionLevel="signature" />
    <application>...</application>
</manifest>
```
Only applications signed with the same developer certificate can use this permission.

###### Enforcing Permissions on Android Components
It is possible to protect Android components using permissions. Activities, Services, Content Providers and Broadcast Receivers all can use the permission mechanism to protect its interfaces.
*Activities*, *Services* and *Broadcast Receivers* can enforce a permission by entering the attribute *android:permission* inside each tag in AndroidManifest.xml:
```
<receiver
    android:name="com.permissions.sample.AnalyticsReceiver"
    android:enabled="true"
    android:permission="com.permissions.sample.ACCESS_USER_INFO">
    ...
</receiver>
```
*Content Providers* are a little bit different. They allow separate permissions for read, write or access the Content Provider using a content URI.
- `android:writePermission`, `android:readPermission`: The developer can set separate permissions to read or write.
- `android:permission`: General permission that will control read and write to the Content Provider.
- `android:grantUriPermissions`: True if the Content Provider can be accessed using a content URI, temporarily overcoming the restriction of other permissions and False, if not.

### Android IPC

As we know, every process on Android has its own sandboxed address space. Inter-process communication (IPC) facilities enable apps to exchange signals and data in a (hopefully) secure way. Instead of relying on the default Linux IPC facilities, IPC on Android is done through Binder, a custom implementation of OpenBinder. A lot of Android system services, as well as all high-level IPC services, depend on Binder.

In the Binder framework, a client-server communication model is used. IPC clients communicate through a client-side proxy. This proxy connects to the Binder server, which is implemented as a character driver (/dev/binder).The server holds a thread pool for handling incoming requests, and is responsible for delivering messages to the destination object. Developers  write interfaces for remote services using the Android Interface Descriptor Language (AIDL).

![Binder Overview](/Document/Images/Chapters/0x04a/binder.jpg)
*Binder Overview. Image source: [Android Binder by Thorsten Schreiber](https://www.nds.rub.de/media/attachments/files/2011/10/main.pdf)*

#### High-Level Abstractions

*Intent messaging* is a framework for asynchronous communication built on top of binder. This framework enables both point-to-point and publish-subscribe messaging. An *Intent* is a messaging object that can be used to request an action from another app component. Although intents facilitate communication between components in several ways, there are three fundamental use cases:

- Starting an activity
	- An Activity represents a single screen in an app. You can start a new instance of an Activity by passing an Intent to startActivity(). The Intent describes the activity to start and carries any necessary data.
- Starting an Service
	- A Service is a component that performs operations in the background without a user interface. With Android 5.0 (API level 21) and later, you can start a service with JobScheduler. 
- Delivering a broadcast
	- A broadcast is a message that any app can receive. The system delivers various broadcasts for system events, such as when the system boots up or the device starts charging. You can deliver a broadcast to other apps by passing an Intent to sendBroadcast() or sendOrderedBroadcast().

There are two types of Intents:

- Explicit intents specify the component to start by name (the fully-qualified class name).

- Implicit intents do not name a specific component, but instead declare a general action to perform, which allows a component from another app to handle it. When you create an implicit intent, the Android system finds the appropriate component to start by comparing the contents of the intent to the intent filters declared in the manifest file of other apps on the device.

An *intent filter* is an expression in an app's manifest file that specifies the type of intents that the component would like to receive. For instance, by declaring an intent filter for an activity, you make it possible for other apps to directly start your activity with a certain kind of intent. Likewise, if you do not declare any intent filters for an activity, then it can be started only with an explicit intent.

For activities and broadcast receivers, intents are the preferred mechanism for asynchronous IPC in Android. Depending on your application requirements, you might use sendBroadcast(), sendOrderedBroadcast(), or an explicit intent to a specific application component.

A BroadcastReceiver handles asynchronous requests initiated by an Intent.

Using Binder or Messenger is the preferred mechanism for RPC-style IPC in Android. They provide a well-defined interface that enables mutual authentication of the endpoints, if required.


(... TODO ... briefly on security implications)

Android’s Messenger represents a reference to a Handler that can be sent to a remote process via an Intent

A reference to the Messenger can be sent via an Intent using the previously mentioned IPC mechanism

Messages sent by the remote process via the messenger are delivered to the local handler. Great for efficient call-backs from the service to the client


### References

+ [Android Security](https://source.android.com/security/)
+ [Android Developer: App Components](https://developer.android.com/guide/components/index.html)
+ [HAL](https://source.android.com/devices/)
+ "Android Security: Attacks and Defenses" By Anmol Misra, Abhishek Dubey
+ [AProgrammer Blog](https://pierrchen.blogspot.com.br)
+ [keesj Android internals](https://github.com/keesj/gomo)
+ [Android Versions] (https://en.wikipedia.org/wiki/Android_version_history)