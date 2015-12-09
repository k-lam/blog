#ProGuard

##关键名词
###Entry points

In order to determine which code has to be preserved and which code can be discarded or obfuscated, you have to specify one or more entry points to your code. These entry points are typically classes with main methods, applets, midlets, etc.

* In the shrinking step, ProGuard starts from these seeds and recursively determines which classes and class members are used. All other classes and class members are discarded.

* In the optimization step, ProGuard further optimizes the code. Among other optimizations, classes and methods that are not entry points can be made private, static, or final, unused parameters can be removed, and some methods may be inlined.
* In the obfuscation step, ProGuard renames classes and class members that are not entry points. In this entire process, keeping the entry points ensures that they can still be accessed by their original names.

* The preverification step is the only step that doesn't have to know the entry points.


##A complete Android application

These options shrink, optimize, and obfuscate all public activities, services, broadcast receivers, and content providers from the compiled classes and external libraries:

	-injars      bin/classes
	-injars      libs
	-outjars     bin/classes-processed.jar
	-libraryjars /usr/local/java/android-sdk/platforms/android-9/android.jar
	
	-dontpreverify
	-repackageclasses ''
	-allowaccessmodification
	-optimizations !code/simplification/arithmetic
	-keepattributes *Annotation*
	
	-keep public class * extends android.app.Activity
	-keep public class * extends android.app.Application
	-keep public class * extends android.app.Service
	-keep public class * extends android.content.BroadcastReceiver
	-keep public class * extends android.content.ContentProvider
	
	-keep public class * extends android.view.View {
	    public <init>(android.content.Context);
	    public <init>(android.content.Context, android.util.AttributeSet);
	    public <init>(android.content.Context, android.util.AttributeSet, int);
	    public void set*(...);
	}
	
	-keepclasseswithmembers class * {
	    public <init>(android.content.Context, android.util.AttributeSet);
	}
	
	-keepclasseswithmembers class * {
	    public <init>(android.content.Context, android.util.AttributeSet, int);
	}
	
	-keepclassmembers class * implements android.os.Parcelable {
	    static android.os.Parcelable$Creator CREATOR;
	}
	
	-keepclassmembers class **.R$* {
	    public static <fields>;
	}
Most importantly, we're keeping all fundamental classes that may be referenced by the AndroidManifest.xml file of the application. If your manifest file contains other classes and methods, you may have to specify those as well.

We're keeping annotations, since they might be used by custom RemoteViews.

We're keeping any custom View extensions and other classes with typical constructors, since they might be referenced from XML layout files.

We're also keeping the required static fields in Parcelable implementations, since they are accessed by introspection.

Finally, we're keeping the static fields of referenced inner classes of auto-generated R classes, just in case your code is accessing those fields by introspection. Note that the compiler already inlines primitive fields, so ProGuard can generally remove all these classes entirely anyway (because the classes are not referenced and therefore not required).

If you're using additional Google APIs, you'll have to specify those as well, for instance:

-libraryjars /usr/local/android-sdk/add-ons/google_apis-7_r01/libs/maps.jar
If you're using Google's optional License Verification Library, you can obfuscate its code along with your own code. You do have to preserve its ILicensingService interface for the library to work:

-keep public interface com.android.vending.licensing.ILicensingService
If you're using the Android Compatibility library, you should add the following line, to let ProGuard know it's ok that the library references some classes that are not available in all versions of the API:

-dontwarn android.support.**
If applicable, you should add options for processing native methods, callback methods, enumerations, and resource files. You may also want to add options for producing useful stack traces. You can find a complete sample configuration in examples/android.pro in the ProGuard distribution.

The build process of the Android SDK (version 2.3 and higher) already integrates ProGuard by default. You only need to enable it (for release builds) by adding proguard.config=proguard.cfg to the file build.properties. In case of problems, you may want to check if the automatically generated file proguard.cfg contains the settings discussed above. The generated Ant build file already sets the input and output files for you.

For more information, you can consult the official Developer Guide in the Android SDK.

参考：

[Android官网介绍](http://developer.android.com/tools/help/proguard.html)

[ProGuard官网](http://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/introduction.html)

[中文入门](http://lenciel.cn/2012/02/integration-of-proguard-and-maven-in-android-projects/)