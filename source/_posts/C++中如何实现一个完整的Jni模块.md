title: C++中如何实现一个完整的Jni模块
tags: [Jni,C++,Java]
date: 2015-11-05 10:27:41
description: 实现一个Jni模块的基本思路
---

在[开发中容易忽略的事情](http://peter517.github.io/2015/08/20/%E5%BC%80%E5%8F%91%E4%B8%AD%E5%AE%B9%E6%98%93%E5%BF%BD%E7%95%A5%E7%9A%84%E4%BA%8B%E6%83%85/#操作系统原生语言)文章中有提到，对于客户端开发，复用程度最高的还是C语言，但C语言实现的功能在Java语言环境中要通过Jni来调用
> Jni基本实现比较简单，但是要做稳定还要考虑很多因素，比如资源的释放、多线程处理、实例管理

# 基本流程
## JNI_OnLoad
C层加载动态库时调用的第一个函数就是JNI\_OnLoad，JNI\_OnLoad可以获得JavaVM是全局变量，通过JavaVM可以拿到JNIEnv来进行Jni常见操作（比如FindClass），不同的Jni模块共享同一个JavaVM，所以可以有一个工具模块，来管理JavaVM，同时在不同线程使用JNIEnv时封装调用AttachCurrentThread和DetachCurrentThread的逻辑，其他Jni模块依赖这个工具模块来使用JNIEnv。
一般在在JNI\_OnLoad时进行RegisterNatives操作，具体代码如下，NativeTest里面的initNativeMethods方法会调用RegisterNatives
```c++
extern “C” JNIEXPORT jint JNI_OnLoad(JavaVM* vm, void* reserved)
{
    if (!NativeTest::initNativeMethods()) {
        return JNI_ERR;
    }
    return JNI_VERSION_1_6;
}
```
## RegisterNatives
通过RegisterNatives方法可以把Java层对应类的方法和C层对应的函数绑定起来，java_class代表Java层的对应类的class信息，所以这个绑定是函数元信息跟函数元信息的绑定，不涉及到具体实例
```c++
jint RegisterNatives(jclass java_class, JNINativeMethod method, jint methodLen)
```
具体代码实例如下：
```c++
void NativeTest::initNativeMethods()
{
    JNINativeMethod nativeFunctions[] =
    {
        {
            "createNativeTest",
            "(Lcom/test/TestCallback;)J",
            (void*) &NativeTest::createNativeTest
        },
        {
            "destroyNativeTest",
            "(J)I",
            (void*) &NativeTest::destroyNativeTest
        },
        {
            "start",
            "(J)I",
            (void*) &NativeTest::terminate
        },
    };
    env->RegisterNatives("com/test/NativeTest",
                    nativeFunctions,
                    sizeof(nativeFunctions)/sizeof(nativeFunctions[0]));
}
```
JNINativeMethod类型主要包括三个元素，依次为：Java层对应的方法名、Java层方法名的签名，C层对应的方法，比如 "createNativeTest"、"(Lcom/test/TestCallback;)J"、(void*) &NativeTest::createNativeTest

## 创建C层实例
JNI功能是Java层的一个函数到C层的一个函数的映射，不是一个Java层的类实例和C层类实例的一一对应
，所以Java类和C层的类都应该是static的，当需要使用C层的一个实例时，需要在Java层保持C层的一个实例地址，所有操作都是基于这个实例，具体代码如下：
```c++
jlong NativeTest::createNativeTest(JNIEnv* env, jclass cls,jobject jobject_test_callback)
{
    TestCb* test_cb = new TestCb(env,jobject_test_callback);
    Test* test = new Test(test_cb);
    if(test == NULL){
        return -1;
    }
    return reinterpret_cast<intptr_t>(test);
}
jint NativeTest::destroyNativeTest(JNIEnv* env, jclass cls,jlong test_address)
{
    Test* test = reinterpret_cast<Test*>(test_address);
    if (test == NULL) {
        return -1;
    }
    delete test;
    return 0;
}
```
通过createNativeTest来创建一个实例，同时传入了一个回调类TestCb，用来在Test方法完成某项功能后进行回调。把实例地址返回给Java层，Java层在调用其他方法时，比如start方法，需要把这个作为参数传入，通过reinterpret_cast强转为对应的类型，再进行相应的调用，如下：
```c++
jint NativeTest::start(JNIEnv* env, jclass cls,jlong test_address)
{
    Test* test = reinterpret_cast<Test*>(test_address);
    if (test == NULL) {
        return -1;
    }
    return test->start();
}
```
## C层回调
Java层调用了一个C的接口，C里面给Java一个回调是很常见的业务，具体代码如下：
```c++
TestCb::TestCb(JNIEnv* env, jobject jobject_callback)
{
    jobject_callback_global_ = NULL;
    if (NULL != jobject_callback)
    {
        jclass jclass_callback = env->GetObjectClass(jobject_callback);
        if (NULL != jclass_callback)
        {
            jobject_callback_global_ =  env->NewGlobalRef(jobject_callback);
            jclass_callback_global_ = (jclass) env->NewGlobalRef(jclass_callback);;
            env->DeleteLocalRef(jclass_callback);
        }
    }
}

TestCb::~TestCb()
{
    if (jobject_callback_global_)
    {
        env->DeleteGlobalRef(jobject_callback_global_);
        jobject_callback_global_ = NULL;
    }
    if (jclass_callback_global_)
    {
        env->DeleteGlobalRef(jclass_callback_global_);
        jclass_callback_global_ = NULL;
    }
}
```
TestCb类里面保存了从Java层传入的对象指针jobject\_callback，在构造函数中将其转化成全局的指针jobject\_callback\_global\_和对应的类信息对象jclass\_callback\_global\_，在Test类中进行回调时，通过jobject\_callback\_global\_获取jmethodID，通过jclass\_callback\_global\_找到对应的对象完成回调，如下：
```c++
void TestCb::onTest(long value)
{
    jlong j_value = (jlong) value;
    jmethodID jmethod =env->GetMethodID(jclass_callback_global_, "onTest", "(J)V");
    env->CallVoidMethod(jobject_callback_global_, jmethod,j_value);
}
```
## Java层实现
Java层的类主要是对C层实例的封装，让使用者感觉不到是C的实现，代码如下，在构造函数的时候调用createNativeTest来保存C层的指针，调用start方法时把nativeAddress传到C层，通过terminate方法来释放对应的C层指针
```java
//NativeTest
public class NativeTest {
    private long nativeAddress = 0L;
    private TestCallback testCallback = new TestCallback();
    private boolean create() {
        if(0L != this.nativeAddress) {
            this.destroy();
        }
        this.nativeAddress = createNativeTest(this.testCallback);
        return 0L != this.nativeAddress;
    }
    public NativeTest() {
        create();
    }
    public int start() {
        return start(this.nativeAddress);
    }
    public int terminate() {
        int ret = terminate(this.nativeAddress);
        return destroy();
    }
    private void destroy() {
        long nativeAddressTemp = this.nativeAddress;
        this.nativeAddress = 0L;
        destroyNativeTest(nativeAddressTemp);
    }

    private static native long createNativeTest(TestCallback callback);
    private static native int destroyNativeTest(long nativeAddress);
    private static native int start(long nativeAddress);
}
```
回调类TestCallback是给C层调用的，在NativeTest初始化的时候把回调地址传给C层，TestCallback可以作为一个接口，这样更改回调实现时就不用更改C层代码
```java
public class TestCallback {
    public void onTest(long value){
        //TODO
    }
}
```

# 资源释放
Jni在本地局部或全局引用没有释放，超过内存句柄限制时会导致“local reference table overflow”，jobject、jstring、jclass是常见的需要释放的类型，通过命名方式可以提醒程序员来释放相应资源

- jobject类型本地局部命名以“jobject\_”开头，例如：jobject\_arraylist
- jstring类型本地局部命名以“jstring\_”开头，例如：jstring\_id
- jclass类型本地局部命名以“jclass\_”开头，例如：jclass\_test

# 结论


<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>
