            
[jni](http://baike.baidu.com/view/1272329.htm)

[jni wiki](http://en.wikipedia.org/wiki/Java_Native_Interface)

jni不是android研究出来的技术，他是java和其他非java写的库连接的中间层。在android中就是java和底层c/c++写成的so（shared libary 共享库）通讯的。java和c/c++的不同，在jni中屏蔽，转换。

Java <-> jni <-> c/c++ native

Java是在虚拟机中运行得，而jni作为中间层，是用c/c++写的，那jni中怎样和java联系呢？通过JNIEnv、JavaVM这些结构体或类（c中是结构体，c++是类，特别是JNIEnv这个的实现），和java联系。实际上这几个只是在c/c++中的一个reference，所以他们的生命周期还是靠JVM管理的（更多可以查看NewGobalRef这类函数）。而c/c++ native这层则不受JVM控制了，是独立的c/c++。

jni做的就是java转c/c++，c/c++转java。但要注意，java是访问不到c++对象，反之可行。

代码建议：jni层独立一个jni.c或jni.cpp。

####包括数据类型的不同：


<table class="wikitable">
<tbody><tr>
<th>Native Type</th>
<th>Java Language Type</th>
<th>Description</th>
<th>Type signature</th>
</tr>
<tr>
<td>unsigned char</td>
<td>jboolean</td>
<td>unsigned 8 bits</td>
<td>Z</td>
</tr>
<tr>
<td>signed char</td>
<td>jbyte</td>
<td>signed 8 bits</td>
<td>B</td>
</tr>
<tr>
<td>unsigned short</td>
<td>jchar</td>
<td>unsigned 16 bits</td>
<td>C</td>
</tr>
<tr>
<td>short</td>
<td>jshort</td>
<td>signed 16 bits</td>
<td>S</td>
</tr>
<tr>
<td>long</td>
<td>jint</td>
<td>signed 32 bits</td>
<td>I</td>
</tr>
<tr>
<td>
<p>long long<br>
__int64</p>
</td>
<td>jlong</td>
<td>signed 64 bits</td>
<td>J</td>
</tr>
<tr>
<td>float</td>
<td>jfloat</td>
<td>32 bits</td>
<td>F</td>
</tr>
<tr>
<td>double</td>
<td>jdouble</td>
<td>64 bits</td>
<td>D</td>
</tr>
<tr>
<td>void</td>
<td></td>
<td></td>
<td>V</td>
</tr>
</tbody></table>
	
	#ifdef HAVE_INTTYPES_H
	# include <inttypes.h>      /* C99 */
	typedef uint8_t         jboolean;       /* unsigned 8 bits */
	typedef int8_t          jbyte;          /* signed 8 bits */
	typedef uint16_t        jchar;          /* unsigned 16 bits */
	typedef int16_t         jshort;         /* signed 16 bits */
	typedef int32_t         jint;           /* signed 32 bits */
	typedef int64_t         jlong;          /* signed 64 bits */
	typedef float           jfloat;         /* 32-bit IEEE 754 */
	typedef double          jdouble;        /* 64-bit IEEE 754 */
	#else
	typedef unsigned char   jboolean;       /* unsigned 8 bits */
	typedef signed char     jbyte;          /* signed 8 bits */
	typedef unsigned short  jchar;          /* unsigned 16 bits */
	typedef short           jshort;         /* signed 16 bits */
	typedef int             jint;           /* signed 32 bits */
	typedef long long       jlong;          /* signed 64 bits */
	typedef float           jfloat;         /* 32-bit IEEE 754 */
	typedef double          jdouble;        /* 64-bit IEEE 754 */

android jni 暂时参考1.6的版本


####build
可以通过android.mk或application.mk

android.mk
An Android.mk file is a small build script that you write to describe your sources to the NDK build system.
这个script是用来把你写的源码分成模块的（the NDK groups your sources into "modules"）
详细可以看 `<ndk>/docs/ANDROID-MK.html`


####调试
`<ndk>/docs/NDK-GDB.html`


####方法注册
#####静态
java中的方法

	package test;

	public class Jni {

	static{
		System.loadLibrary("kl-test");
	}
	
	public native static String getKLTest();

}

在Jni的c中就要

	#include <jni.h>

	jstring
	Java_test_Jni_getKLTest(JNIEnv* env,jclass clazz)

* Java加方法路径（把`.`换成`_`）,当然，java的方法本来就有下划线，也是可以的

* 这些和java中native对应的函数，我们叫做jni函数，返回值和参数值是要转换的。首先看参数：
	1. 必须加上两个参数，JNIEnv，和jclass或jobject，如果对应的java中native那个函数是static的，用jclass，否则用jobject
	2. 类型转换，由于java和c/c++的数据类型不一样，所以要进行转换。

####动态注册

在`JNI_OnLoad`这个方法中注册，这个函数可以放在你模块的任意文件的任意地方
如：

###JavaVM
jni中代表java虚拟机

JavaVM实际上是JNIInvokeInterface的别名

	struct JNIInvokeInterface {
    void*       reserved0;
    void*       reserved1;
    void*       reserved2;

    jint        (*DestroyJavaVM)(JavaVM*);
    jint        (*AttachCurrentThread)(JavaVM*, JNIEnv**, void*);
    jint        (*DetachCurrentThread)(JavaVM*);
    jint        (*GetEnv)(JavaVM*, void**, jint);
    jint        (*AttachCurrentThreadAsDaemon)(JavaVM*, JNIEnv**, void*);
	};

###JNIEnv
JNIEnv是一个与线程相关的代表JNI环境的结构体

而JNIEnv实际上是JNINativeInterface这个的别名

	struct JNINativeInterface {
    void*       reserved0;
    void*       reserved1;
    void*       reserved2;
    void*       reserved3;

    jint        (*GetVersion)(JNIEnv *);

    jclass      (*DefineClass)(JNIEnv*, const char*, jobject, const jbyte*,
                        jsize);
    jclass      (*FindClass)(JNIEnv*, const char*);

    jmethodID   (*FromReflectedMethod)(JNIEnv*, jobject);
    jfieldID    (*FromReflectedField)(JNIEnv*, jobject);
    /* spec doesn't show jboolean parameter */
    jobject     (*ToReflectedMethod)(JNIEnv*, jclass, jmethodID, jboolean);

    jclass      (*GetSuperclass)(JNIEnv*, jclass);
    jboolean    (*IsAssignableFrom)(JNIEnv*, jclass, jclass);

    /* spec doesn't show jboolean parameter */
    jobject     (*ToReflectedField)(JNIEnv*, jclass, jfieldID, jboolean);

    jint        (*Throw)(JNIEnv*, jthrowable);
    jint        (*ThrowNew)(JNIEnv *, jclass, const char *);
    jthrowable  (*ExceptionOccurred)(JNIEnv*);
    void        (*ExceptionDescribe)(JNIEnv*);
    void        (*ExceptionClear)(JNIEnv*);
    void        (*FatalError)(JNIEnv*, const char*);

    jint        (*PushLocalFrame)(JNIEnv*, jint);
    jobject     (*PopLocalFrame)(JNIEnv*, jobject);

    jobject     (*NewGlobalRef)(JNIEnv*, jobject);
    void        (*DeleteGlobalRef)(JNIEnv*, jobject);
    void        (*DeleteLocalRef)(JNIEnv*, jobject);
    jboolean    (*IsSameObject)(JNIEnv*, jobject, jobject);

    jobject     (*NewLocalRef)(JNIEnv*, jobject);
    jint        (*EnsureLocalCapacity)(JNIEnv*, jint);

    jobject     (*AllocObject)(JNIEnv*, jclass);
    jobject     (*NewObject)(JNIEnv*, jclass, jmethodID, ...);
    jobject     (*NewObjectV)(JNIEnv*, jclass, jmethodID, va_list);
    jobject     (*NewObjectA)(JNIEnv*, jclass, jmethodID, jvalue*);

    jclass      (*GetObjectClass)(JNIEnv*, jobject);
    jboolean    (*IsInstanceOf)(JNIEnv*, jobject, jclass);
    jmethodID   (*GetMethodID)(JNIEnv*, jclass, const char*, const char*);

    jobject     (*CallObjectMethod)(JNIEnv*, jobject, jmethodID, ...);
    jobject     (*CallObjectMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jobject     (*CallObjectMethodA)(JNIEnv*, jobject, jmethodID, jvalue*);
    jboolean    (*CallBooleanMethod)(JNIEnv*, jobject, jmethodID, ...);
    jboolean    (*CallBooleanMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jboolean    (*CallBooleanMethodA)(JNIEnv*, jobject, jmethodID, jvalue*);
    jbyte       (*CallByteMethod)(JNIEnv*, jobject, jmethodID, ...);
    jbyte       (*CallByteMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jbyte       (*CallByteMethodA)(JNIEnv*, jobject, jmethodID, jvalue*);
    jchar       (*CallCharMethod)(JNIEnv*, jobject, jmethodID, ...);
    jchar       (*CallCharMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jchar       (*CallCharMethodA)(JNIEnv*, jobject, jmethodID, jvalue*);
    jshort      (*CallShortMethod)(JNIEnv*, jobject, jmethodID, ...);
    jshort      (*CallShortMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jshort      (*CallShortMethodA)(JNIEnv*, jobject, jmethodID, jvalue*);
    jint        (*CallIntMethod)(JNIEnv*, jobject, jmethodID, ...);
    jint        (*CallIntMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jint        (*CallIntMethodA)(JNIEnv*, jobject, jmethodID, jvalue*);
    jlong       (*CallLongMethod)(JNIEnv*, jobject, jmethodID, ...);
    jlong       (*CallLongMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    jlong       (*CallLongMethodA)(JNIEnv*, jobject, jmethodID, jvalue*);
    jfloat      (*CallFloatMethod)(JNIEnv*, jobject, jmethodID, ...) __NDK_FPABI__;
    jfloat      (*CallFloatMethodV)(JNIEnv*, jobject, jmethodID, va_list) __NDK_FPABI__;
    jfloat      (*CallFloatMethodA)(JNIEnv*, jobject, jmethodID, jvalue*) __NDK_FPABI__;
    jdouble     (*CallDoubleMethod)(JNIEnv*, jobject, jmethodID, ...) __NDK_FPABI__;
    jdouble     (*CallDoubleMethodV)(JNIEnv*, jobject, jmethodID, va_list) __NDK_FPABI__;
    jdouble     (*CallDoubleMethodA)(JNIEnv*, jobject, jmethodID, jvalue*) __NDK_FPABI__;
    void        (*CallVoidMethod)(JNIEnv*, jobject, jmethodID, ...);
    void        (*CallVoidMethodV)(JNIEnv*, jobject, jmethodID, va_list);
    void        (*CallVoidMethodA)(JNIEnv*, jobject, jmethodID, jvalue*);

    jobject     (*CallNonvirtualObjectMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jobject     (*CallNonvirtualObjectMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    jobject     (*CallNonvirtualObjectMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, jvalue*);
    jboolean    (*CallNonvirtualBooleanMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jboolean    (*CallNonvirtualBooleanMethodV)(JNIEnv*, jobject, jclass,
                         jmethodID, va_list);
    jboolean    (*CallNonvirtualBooleanMethodA)(JNIEnv*, jobject, jclass,
                         jmethodID, jvalue*);
    jbyte       (*CallNonvirtualByteMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jbyte       (*CallNonvirtualByteMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    jbyte       (*CallNonvirtualByteMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, jvalue*);
    jchar       (*CallNonvirtualCharMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jchar       (*CallNonvirtualCharMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    jchar       (*CallNonvirtualCharMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, jvalue*);
    jshort      (*CallNonvirtualShortMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jshort      (*CallNonvirtualShortMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    jshort      (*CallNonvirtualShortMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, jvalue*);
    jint        (*CallNonvirtualIntMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jint        (*CallNonvirtualIntMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    jint        (*CallNonvirtualIntMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, jvalue*);
    jlong       (*CallNonvirtualLongMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    jlong       (*CallNonvirtualLongMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    jlong       (*CallNonvirtualLongMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, jvalue*);
    jfloat      (*CallNonvirtualFloatMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...) __NDK_FPABI__;
    jfloat      (*CallNonvirtualFloatMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list) __NDK_FPABI__;
    jfloat      (*CallNonvirtualFloatMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, jvalue*) __NDK_FPABI__;
    jdouble     (*CallNonvirtualDoubleMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...) __NDK_FPABI__;
    jdouble     (*CallNonvirtualDoubleMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list) __NDK_FPABI__;
    jdouble     (*CallNonvirtualDoubleMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, jvalue*) __NDK_FPABI__;
    void        (*CallNonvirtualVoidMethod)(JNIEnv*, jobject, jclass,
                        jmethodID, ...);
    void        (*CallNonvirtualVoidMethodV)(JNIEnv*, jobject, jclass,
                        jmethodID, va_list);
    void        (*CallNonvirtualVoidMethodA)(JNIEnv*, jobject, jclass,
                        jmethodID, jvalue*);

    jfieldID    (*GetFieldID)(JNIEnv*, jclass, const char*, const char*);

    jobject     (*GetObjectField)(JNIEnv*, jobject, jfieldID);
    jboolean    (*GetBooleanField)(JNIEnv*, jobject, jfieldID);
    jbyte       (*GetByteField)(JNIEnv*, jobject, jfieldID);
    jchar       (*GetCharField)(JNIEnv*, jobject, jfieldID);
    jshort      (*GetShortField)(JNIEnv*, jobject, jfieldID);
    jint        (*GetIntField)(JNIEnv*, jobject, jfieldID);
    jlong       (*GetLongField)(JNIEnv*, jobject, jfieldID);
    jfloat      (*GetFloatField)(JNIEnv*, jobject, jfieldID) __NDK_FPABI__;
    jdouble     (*GetDoubleField)(JNIEnv*, jobject, jfieldID) __NDK_FPABI__;

    void        (*SetObjectField)(JNIEnv*, jobject, jfieldID, jobject);
    void        (*SetBooleanField)(JNIEnv*, jobject, jfieldID, jboolean);
    void        (*SetByteField)(JNIEnv*, jobject, jfieldID, jbyte);
    void        (*SetCharField)(JNIEnv*, jobject, jfieldID, jchar);
    void        (*SetShortField)(JNIEnv*, jobject, jfieldID, jshort);
    void        (*SetIntField)(JNIEnv*, jobject, jfieldID, jint);
    void        (*SetLongField)(JNIEnv*, jobject, jfieldID, jlong);
    void        (*SetFloatField)(JNIEnv*, jobject, jfieldID, jfloat) __NDK_FPABI__;
    void        (*SetDoubleField)(JNIEnv*, jobject, jfieldID, jdouble) __NDK_FPABI__;

    jmethodID   (*GetStaticMethodID)(JNIEnv*, jclass, const char*, const char*);

    jobject     (*CallStaticObjectMethod)(JNIEnv*, jclass, jmethodID, ...);
    jobject     (*CallStaticObjectMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    jobject     (*CallStaticObjectMethodA)(JNIEnv*, jclass, jmethodID, jvalue*);
    jboolean    (*CallStaticBooleanMethod)(JNIEnv*, jclass, jmethodID, ...);
    jboolean    (*CallStaticBooleanMethodV)(JNIEnv*, jclass, jmethodID,
                        va_list);
    jboolean    (*CallStaticBooleanMethodA)(JNIEnv*, jclass, jmethodID,
                        jvalue*);
    jbyte       (*CallStaticByteMethod)(JNIEnv*, jclass, jmethodID, ...);
    jbyte       (*CallStaticByteMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    jbyte       (*CallStaticByteMethodA)(JNIEnv*, jclass, jmethodID, jvalue*);
    jchar       (*CallStaticCharMethod)(JNIEnv*, jclass, jmethodID, ...);
    jchar       (*CallStaticCharMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    jchar       (*CallStaticCharMethodA)(JNIEnv*, jclass, jmethodID, jvalue*);
    jshort      (*CallStaticShortMethod)(JNIEnv*, jclass, jmethodID, ...);
    jshort      (*CallStaticShortMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    jshort      (*CallStaticShortMethodA)(JNIEnv*, jclass, jmethodID, jvalue*);
    jint        (*CallStaticIntMethod)(JNIEnv*, jclass, jmethodID, ...);
    jint        (*CallStaticIntMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    jint        (*CallStaticIntMethodA)(JNIEnv*, jclass, jmethodID, jvalue*);
    jlong       (*CallStaticLongMethod)(JNIEnv*, jclass, jmethodID, ...);
    jlong       (*CallStaticLongMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    jlong       (*CallStaticLongMethodA)(JNIEnv*, jclass, jmethodID, jvalue*);
    jfloat      (*CallStaticFloatMethod)(JNIEnv*, jclass, jmethodID, ...) __NDK_FPABI__;
    jfloat      (*CallStaticFloatMethodV)(JNIEnv*, jclass, jmethodID, va_list) __NDK_FPABI__;
    jfloat      (*CallStaticFloatMethodA)(JNIEnv*, jclass, jmethodID, jvalue*) __NDK_FPABI__;
    jdouble     (*CallStaticDoubleMethod)(JNIEnv*, jclass, jmethodID, ...) __NDK_FPABI__;
    jdouble     (*CallStaticDoubleMethodV)(JNIEnv*, jclass, jmethodID, va_list) __NDK_FPABI__;
    jdouble     (*CallStaticDoubleMethodA)(JNIEnv*, jclass, jmethodID, jvalue*) __NDK_FPABI__;
    void        (*CallStaticVoidMethod)(JNIEnv*, jclass, jmethodID, ...);
    void        (*CallStaticVoidMethodV)(JNIEnv*, jclass, jmethodID, va_list);
    void        (*CallStaticVoidMethodA)(JNIEnv*, jclass, jmethodID, jvalue*);

    jfieldID    (*GetStaticFieldID)(JNIEnv*, jclass, const char*,
                        const char*);

    jobject     (*GetStaticObjectField)(JNIEnv*, jclass, jfieldID);
    jboolean    (*GetStaticBooleanField)(JNIEnv*, jclass, jfieldID);
    jbyte       (*GetStaticByteField)(JNIEnv*, jclass, jfieldID);
    jchar       (*GetStaticCharField)(JNIEnv*, jclass, jfieldID);
    jshort      (*GetStaticShortField)(JNIEnv*, jclass, jfieldID);
    jint        (*GetStaticIntField)(JNIEnv*, jclass, jfieldID);
    jlong       (*GetStaticLongField)(JNIEnv*, jclass, jfieldID);
    jfloat      (*GetStaticFloatField)(JNIEnv*, jclass, jfieldID) __NDK_FPABI__;
    jdouble     (*GetStaticDoubleField)(JNIEnv*, jclass, jfieldID) __NDK_FPABI__;

    void        (*SetStaticObjectField)(JNIEnv*, jclass, jfieldID, jobject);
    void        (*SetStaticBooleanField)(JNIEnv*, jclass, jfieldID, jboolean);
    void        (*SetStaticByteField)(JNIEnv*, jclass, jfieldID, jbyte);
    void        (*SetStaticCharField)(JNIEnv*, jclass, jfieldID, jchar);
    void        (*SetStaticShortField)(JNIEnv*, jclass, jfieldID, jshort);
    void        (*SetStaticIntField)(JNIEnv*, jclass, jfieldID, jint);
    void        (*SetStaticLongField)(JNIEnv*, jclass, jfieldID, jlong);
    void        (*SetStaticFloatField)(JNIEnv*, jclass, jfieldID, jfloat) __NDK_FPABI__;
    void        (*SetStaticDoubleField)(JNIEnv*, jclass, jfieldID, jdouble) __NDK_FPABI__;

    jstring     (*NewString)(JNIEnv*, const jchar*, jsize);
    jsize       (*GetStringLength)(JNIEnv*, jstring);
    const jchar* (*GetStringChars)(JNIEnv*, jstring, jboolean*);
    void        (*ReleaseStringChars)(JNIEnv*, jstring, const jchar*);
    jstring     (*NewStringUTF)(JNIEnv*, const char*);
    jsize       (*GetStringUTFLength)(JNIEnv*, jstring);
    /* JNI spec says this returns const jbyte*, but that's inconsistent */
    const char* (*GetStringUTFChars)(JNIEnv*, jstring, jboolean*);
    void        (*ReleaseStringUTFChars)(JNIEnv*, jstring, const char*);
    jsize       (*GetArrayLength)(JNIEnv*, jarray);
    jobjectArray (*NewObjectArray)(JNIEnv*, jsize, jclass, jobject);
    jobject     (*GetObjectArrayElement)(JNIEnv*, jobjectArray, jsize);
    void        (*SetObjectArrayElement)(JNIEnv*, jobjectArray, jsize, jobject);

    jbooleanArray (*NewBooleanArray)(JNIEnv*, jsize);
    jbyteArray    (*NewByteArray)(JNIEnv*, jsize);
    jcharArray    (*NewCharArray)(JNIEnv*, jsize);
    jshortArray   (*NewShortArray)(JNIEnv*, jsize);
    jintArray     (*NewIntArray)(JNIEnv*, jsize);
    jlongArray    (*NewLongArray)(JNIEnv*, jsize);
    jfloatArray   (*NewFloatArray)(JNIEnv*, jsize);
    jdoubleArray  (*NewDoubleArray)(JNIEnv*, jsize);

    jboolean*   (*GetBooleanArrayElements)(JNIEnv*, jbooleanArray, jboolean*);
    jbyte*      (*GetByteArrayElements)(JNIEnv*, jbyteArray, jboolean*);
    jchar*      (*GetCharArrayElements)(JNIEnv*, jcharArray, jboolean*);
    jshort*     (*GetShortArrayElements)(JNIEnv*, jshortArray, jboolean*);
    jint*       (*GetIntArrayElements)(JNIEnv*, jintArray, jboolean*);
    jlong*      (*GetLongArrayElements)(JNIEnv*, jlongArray, jboolean*);
    jfloat*     (*GetFloatArrayElements)(JNIEnv*, jfloatArray, jboolean*);
    jdouble*    (*GetDoubleArrayElements)(JNIEnv*, jdoubleArray, jboolean*);

    void        (*ReleaseBooleanArrayElements)(JNIEnv*, jbooleanArray,
                        jboolean*, jint);
    void        (*ReleaseByteArrayElements)(JNIEnv*, jbyteArray,
                        jbyte*, jint);
    void        (*ReleaseCharArrayElements)(JNIEnv*, jcharArray,
                        jchar*, jint);
    void        (*ReleaseShortArrayElements)(JNIEnv*, jshortArray,
                        jshort*, jint);
    void        (*ReleaseIntArrayElements)(JNIEnv*, jintArray,
                        jint*, jint);
    void        (*ReleaseLongArrayElements)(JNIEnv*, jlongArray,
                        jlong*, jint);
    void        (*ReleaseFloatArrayElements)(JNIEnv*, jfloatArray,
                        jfloat*, jint);
    void        (*ReleaseDoubleArrayElements)(JNIEnv*, jdoubleArray,
                        jdouble*, jint);

    void        (*GetBooleanArrayRegion)(JNIEnv*, jbooleanArray,
                        jsize, jsize, jboolean*);
    void        (*GetByteArrayRegion)(JNIEnv*, jbyteArray,
                        jsize, jsize, jbyte*);
    void        (*GetCharArrayRegion)(JNIEnv*, jcharArray,
                        jsize, jsize, jchar*);
    void        (*GetShortArrayRegion)(JNIEnv*, jshortArray,
                        jsize, jsize, jshort*);
    void        (*GetIntArrayRegion)(JNIEnv*, jintArray,
                        jsize, jsize, jint*);
    void        (*GetLongArrayRegion)(JNIEnv*, jlongArray,
                        jsize, jsize, jlong*);
    void        (*GetFloatArrayRegion)(JNIEnv*, jfloatArray,
                        jsize, jsize, jfloat*);
    void        (*GetDoubleArrayRegion)(JNIEnv*, jdoubleArray,
                        jsize, jsize, jdouble*);

    /* spec shows these without const; some jni.h do, some don't */
    void        (*SetBooleanArrayRegion)(JNIEnv*, jbooleanArray,
                        jsize, jsize, const jboolean*);
    void        (*SetByteArrayRegion)(JNIEnv*, jbyteArray,
                        jsize, jsize, const jbyte*);
    void        (*SetCharArrayRegion)(JNIEnv*, jcharArray,
                        jsize, jsize, const jchar*);
    void        (*SetShortArrayRegion)(JNIEnv*, jshortArray,
                        jsize, jsize, const jshort*);
    void        (*SetIntArrayRegion)(JNIEnv*, jintArray,
                        jsize, jsize, const jint*);
    void        (*SetLongArrayRegion)(JNIEnv*, jlongArray,
                        jsize, jsize, const jlong*);
    void        (*SetFloatArrayRegion)(JNIEnv*, jfloatArray,
                        jsize, jsize, const jfloat*);
    void        (*SetDoubleArrayRegion)(JNIEnv*, jdoubleArray,
                        jsize, jsize, const jdouble*);

    jint        (*RegisterNatives)(JNIEnv*, jclass, const JNINativeMethod*,
                        jint);
    jint        (*UnregisterNatives)(JNIEnv*, jclass);
    jint        (*MonitorEnter)(JNIEnv*, jobject);
    jint        (*MonitorExit)(JNIEnv*, jobject);
    jint        (*GetJavaVM)(JNIEnv*, JavaVM**);

    void        (*GetStringRegion)(JNIEnv*, jstring, jsize, jsize, jchar*);
    void        (*GetStringUTFRegion)(JNIEnv*, jstring, jsize, jsize, char*);

    void*       (*GetPrimitiveArrayCritical)(JNIEnv*, jarray, jboolean*);
    void        (*ReleasePrimitiveArrayCritical)(JNIEnv*, jarray, void*, jint);

    const jchar* (*GetStringCritical)(JNIEnv*, jstring, jboolean*);
    void        (*ReleaseStringCritical)(JNIEnv*, jstring, const jchar*);

    jweak       (*NewWeakGlobalRef)(JNIEnv*, jobject);
    void        (*DeleteWeakGlobalRef)(JNIEnv*, jweak);

    jboolean    (*ExceptionCheck)(JNIEnv*);

    jobject     (*NewDirectByteBuffer)(JNIEnv*, void*, jlong);
    void*       (*GetDirectBufferAddress)(JNIEnv*, jobject);
    jlong       (*GetDirectBufferCapacity)(JNIEnv*, jobject);

    /* added in JNI 1.6 */
    jobjectRefType (*GetObjectRefType)(JNIEnv*, jobject);
	};

####JNI方法doc
[Go -->](http://docs.oracle.com/javase/6/docs/technotes/guides/jni/spec/jniTOC.html)
	
####JNIEnv操作jobject
jobject是java的对象，在jni中如何操作这个对象呢？操作对象就是调用函数和属性。

通过`JNIEnv->Call<type>Method(JNIEnv*,jobject,jmethodID,...)`这一类方法来调用对象的方法

而如果是静态方法则通过`CallStatic<Type>Method`这些方法来调用

对于构建函数，用`jobject (*NewObject)(JNIEnv*,jclass, jmethodID, ...);`

对于field呢？通过`Get<Type>Field(JNIEnv*,jobject,jfieldID)`和`Set<Type>Field(JNIEnv*,jobject,jfieldID,NativeType)`来操作field

这里我们可以看到两个关键的参数MethodID 和FieldID
，怎样这两个又获得？

	JNIEnv中有如下两个方法
	jfieldID GetFieldID(jclass clazz,const char* name,const char* sig);
	jmethodID GetMethodID(jclass clazz,const char* name,const char* sig);//如果是构造函数，name = "<init>",其他的则是方法名

好吧，jclass又怎么获得呢？
	jclass FindClass(const char* name)
	jclass FindClass(JNIEnv*, const char*);
	//例子：
	jclass mediaScannerClientInterface = env->FindClass("android/media/MediaScannerClient");
	//内部类用 $
name是成员的名字，sig是函数和变量的签名
sig是怎样的呢？可以百度一下。
为了方便写signature，可以定义宏，或用javap -s -p xxx来获取签名，xxx是你的类
内部类也可以，如 javap -s -p A.B

	public class Jni {
		int ii;
		static int si;
		final static int fsi = 10;
		static{
			System.loadLibrary("kl-test");
		}
		
		public native static String getKLTest();
		
		public void getUs(String s,String[] ss){
			
		}
		
		public int getNN(String s,float f){
			return 1;
		}
	}

javap后

	Compiled from "Jni.java"
	public class test.Jni {
	  int ii;
	    Signature: I
	  static int si;
	    Signature: I
	  static final int fsi;
	    Signature: I
	  static {};
	    Signature: ()V
	
	  public test.Jni();
	    Signature: ()V
	
	  public static native java.lang.String getKLTest();
	    Signature: ()Ljava/lang/String;
	
	  public void getUs(java.lang.String, java.lang.String[]);
	    Signature: (Ljava/lang/String;[Ljava/lang/String;)V
	
	  public int getNN(java.lang.String, float);
	    Signature: (Ljava/lang/String;F)I
	}

	javap -s -p Jni.Jessica
	Warning: Binary file Jni.Jessica contains test.Jni$Jessica
	Compiled from "Jni.java"
	public class test.Jni$Jessica {
	  public int id;
	    Signature: I
	  public test.Jni$Jessica();
	    Signature: ()V
	
	  public test.Jni$Jessica(int);
	    Signature: (I)V
	
	  public int getId();
	    Signature: ()I
	
	  public void setId(int);
	    Signature: (I)V
	}

		

####jni返回java


####例子：

	java:
	package test;

	public class Jni {
		
		static{
			System.loadLibrary("kl-test");
		}
		
		public native static String callJessica(Jessica jessica);
		public native Jessica giveMeJessica();

		public static class Jessica{
			public Jessica(){
				
			}
			
			public Jessica(int i){
				this.id = i;
		    }
			public int id = 1;
			
			public int getId(){
				return id;
			}
			
			public void setId(int id){
				this.id = id;
			}
		}
	
	}

	C:
	jstring
	Java_test_Jni_callJessica( JNIEnv* env,jclass clazz,jobject jobj){
		jclass jessciaz = (*env)->FindClass(env,"test/Jni$Jessica");
		jfieldID fid =  (*env)->GetFieldID(env,jessciaz,"id","I");
		jmethodID mID =  (*env)->GetMethodID(env,jessciaz,"setId","(I)V");
		if(1 == (*env)->GetIntField(env,jobj,fid)){
			logDebug("jesscia`s id is 1");
		}
		(*env)->CallVoidMethod(env,jobj,mID,2);
		if(2 == (*env)->GetIntField(env,jobj,fid)){
			logDebug("jesscia`s id has been changed to 2");
		}
		return (*env)->NewStringUTF(env,"123");
	}

	jobject Java_test_Jni_giveMeJessica( JNIEnv* env,jobject thiz ){
	jclass cls = (*env)->FindClass(env, "test/Jni$Jessica");
	jmethodID methodId = (*env)->GetMethodID(env,cls,"<init>","(I)V");
	return (*env)->NewObject(env,cls,methodId,52);
}
	
	log的结果：
	jesscia`s id is 1
	jesscia`s id has been changed to 2

###调试
1、使用ndk-build编译时，加上如下参数NDK_DEBUG=1，之后生成so文件之外，还会生成gdbserver,gdb.setup调式文件

2、在项目的Debug Configuration中选择Android Native Apllication，点击下方Debug。或者右击项目，debug as->Android Native Apllication

3、Enjoy your Debugging！

native 的log调试
首先在android.mk
的log的MODULE下，在include $(CLEAR_VARS)后，加上
LOCAL_LDLIBS := -llog

在(platforms/android-3/arch-arm/usr/include/android/log.h）可以查找到如下代码

	typedef enum android_LogPriority {
    ANDROID_LOG_UNKNOWN = 0,
    ANDROID_LOG_DEFAULT,    /* only for SetMinPriority() */
    ANDROID_LOG_VERBOSE,
    ANDROID_LOG_DEBUG,
    ANDROID_LOG_INFO,
    ANDROID_LOG_WARN,
    ANDROID_LOG_ERROR,
    ANDROID_LOG_FATAL,
    ANDROID_LOG_SILENT,     /* only for SetMinPriority(); must be last */
	} android_LogPriority;

	/*
	 * Send a simple string to the log.
	 */
	int __android_log_write(int prio, const char *tag, const char *text);

所以我们可以用`__android_log_write(int prio, const char *tag, const char *text);`这个函数来log，我们在java中`Log.d("dubeg","hello world");`

在native中，要这样才能达到相同效果：

	char * d = "debug";
	char * text = "hello world printf in native";
	__android_log_write(3, d, text);

打开LogCat，就能开到hello world printf in native了；
打印数字的`__android_log_print(3, "debug", "Need to print : %d %d",height, width);`

记得不要`#include <lib.h>`用`#include <android/log.h>`

