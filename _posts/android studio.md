###Dependencies

The Android Studio build system manages project dependencies and supports module dependencies, local binary dependencies, and remote binary dependencies.

####Module Dependencies
A project module can include in its build file a list of other modules it depends on. When you build this module, the build system assembles and includes the required modules.

####Local Dependencies
If you have binary archives in your local filesystem that a module depends on, such as JAR files, you can declare these dependencies in the build file for that module.

####Remote Dependencies
When some of your dependencies are available in a remote repository, you do not have to download them and copy them into your project. The Android Studio build system supports remote Maven dependencies. Maven is a popular software project management tool that helps organize project dependencies using repositories.

Many popular software libraries and tools are available in public Maven repositories. For these dependencies you only have to specify their Maven coordinates, which uniquely identify each element in a remote repository. The format for Maven coordinates used in the build system is **group:name:version**. For example, the Maven coordinates for version 16.0.1 of the Google Guava libraries are com.google.guava:guava:16.0.1.

The [Maven Central Repository](http://search.maven.org/) is widely used to distribute many libraries and tools.

KL:android默认是用[jcenter](http://jcenter.bintray.com/)的
[https://bintray.com/bintray/jcenter](https://bintray.com/bintray/jcenter)这个地址更加友好

####Remote Dependencies

	repositories {
    	mavenCentral()
	}
	dependencies {
		compile 'com.github.chrisbanes.actionbarpulltorefresh:extra-abc:+'
	}
####lib project
	dependencies {
    compile project(':shared')
	}

Declare dependencies

The app module in BuildSystemExample declares three dependencies:

...
dependencies {
    // Module dependency
    compile project(":lib")

    // Remote binary dependency
	//com.android.support is group,appcompat-v7 is name,19.0.1 is version
    compile 'com.android.support:appcompat-v7:19.0.1'

    // Local binary dependency
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
Each of these dependencies is described below. The build system adds all the compile dependencies to the compilation classpath and includes them in the final package.

Module dependencies

The app module depends on the lib module, because MainActivity launches LibActivity1 as described in Open an Activity from a Library Module.

compile project(":lib") declares a dependency on the lib module of BuildSystemExample. When you build the app module, the build system assembles and includes the lib module.

Remote binary dependencies

The app and lib modules both use the ActionBarActivity class from the Android Support Library, so these modules depend on it.

compile 'com.android.support:appcompat-v7:19.0.1' declares a dependency on version 19.0.1 of the Android Support Library by specifying its Maven coordinates. The Android Support Library is available in the Android Repository package of the Android SDK. If your SDK installation does not have this package, download and install it using the SDK Manager.

Android Studio configures projects to use the Maven Central Repository by default. (This configuration is included in the top-level build file for the project.)
Local binary dependencies

The modules in BuildSystemExample do not use any binary dependencies from the local file system. If you have modules that require local binary dependencies, copy the JAR files for these dependencies into <moduleName>/libs inside your project.

compile fileTree(dir: 'libs', include: ['*.jar']) tells the build system that any JAR file inside app/libs is a dependency and should be included in the compilation classpath and in the final package.

For more information about dependencies in Gradle, see Dependency Management Basics in the Gradle User Guide.

http://www.gradle.org/docs/1.2/userguide/dependency_management.html#sub:project_dependencies

考虑一种情况，我有一个project，是作为一个lib被引用的，而且要方便修改，怎样做呢？
建立这个project，这个project下建一个叫A的module
在要引用的project，file->import module，把A导入，如果在其他地方修改了，只需要sync...就能同步了
###Gradle
android studio中gradle的版本查看：
打开project（不是module）下的build.gradle，里面会有一句。

 	dependencies {
        classpath 'com.android.tools.build:gradle:0.12.2'
	}
0.12.2就是版本号

[gradle user guide](http://www.gradle.org/docs/1.2/userguide/userguide.html)

####Dependences

compile
The dependencies required to compile the production source of the project.

runtime
The dependencies required by the production classes at runtime. By default, also includes the compile time dependencies.


KL快捷键

修改注释doc shift + alt + d
xml中查看图片 ctrl + shift + I