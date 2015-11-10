title: C++中线程级资源自动释放
tags: [C++,Jni,Qt]
date: 2015-11-06 15:31:01
description: 如何让一个资源随着线程结束而释放
---

我们常常在代码通过类的生命周期来管理资源释放，构造函数的时候申请资源，析构函数的时候释放资源，这样可以减少程序员自动释放资源的工作。而有些场景，一个资源是线程级别共享的，线程结束后才释放，对于这种场景，可以利用Thread Local Storage机制，因为

> 在C++中，线程的局部变量的析构函数会随着线程的结束而调用

# 函数级
先复习一下函数级资源释，这里面应用的最多的就是局部锁了，代码如下，在构造函数获取锁，在析构的时候释放锁
```c++
template <class Lockable>
class LockScope
{
public:
    LockScope(Lockable & m) : mtx(m)
    {
        mtx.lock();
    }
    ~LockScope()
    {
        mtx.unlock();
    }
private:
    Lockable & mtx;
};
```
调用代码如下：
```c++
void func()
{
    LockScope(lock);
    //...
}
```
# 线程级
线程级别的资源应用场景不多，通常是一个全局静态变量，希望它在每个线程中有独立的拷贝，Android里面的Looper实现算是一个，这样每个线程都有自己的事件队列，互不干扰。
## Qt中全局JniEnv实现
在Jni中，获取JNIEnv时，在使用前要调用AttachCurrentThread，使用后要调用DetachCurrentThread，否则线程无法正常退出，这就给程序员比较大压力，我们需要想一种方法来封装这个操作，Qt中的全局JniEnv实现给了我们一种思路
如下面代码所示，有一个全局静态变量QJNIEnvironmentPrivateTLS，存放在QThreadStorage中，表示是线程局部变量，它的析构函数是执行DetachCurrentThread操作，表示在线程退出时会调用这个操作。
```c++
class QJNIEnvironmentPrivateTLS
{
public:
    inline ~QJNIEnvironmentPrivateTLS()
    {
        QtAndroidPrivate::javaVM()->DetachCurrentThread();
    }
};

Q_GLOBAL_STATIC(QThreadStorage<QJNIEnvironmentPrivateTLS*>, jniEnvTLS)
```
QJNIEnvironmentPrivate在构造函数中会获取JNIEnv，如果发现没有调用过AttachCurrentThread则马上Attach一下，
> 每个线程在AttachCurrentThread时会在线程局部变量中初始化一个QJNIEnvironmentPrivateTLS实例，这样可以保证在线程退出时会调用相应的DetachCurrentThread操作

```c++
static const char qJniThreadName[] = "QtThread";

QJNIEnvironmentPrivate::QJNIEnvironmentPrivate()
    : jniEnv(0)
{
    JavaVM* vm = QtAndroidPrivate::javaVM();
    const jint ret = vm->GetEnv((void**)&jniEnv, JNI_VERSION_1_6);
    if (ret == JNI_OK) // Already attached
        return;

    if (ret == JNI_EDETACHED) { // We need to (re-)attach
        JavaVMAttachArgs args = { JNI_VERSION_1_6, qJniThreadName, Q_NULLPTR };
        if (vm->AttachCurrentThread(&jniEnv, &args) != JNI_OK)
            return;

        if (!jniEnvTLS->hasLocalData()) // If we attached the thread we own it.
            jniEnvTLS->setLocalData(new QJNIEnvironmentPrivateTLS);
    }
}
```
QJNIEnvironmentPrivate还重载了->方法，这样使用者就可以通过QJNIEnvironmentPrivate来使用JniEnv，而不用考虑AttachCurrentThread和DetachCurrentThread操作
```c++
JNIEnv *QJNIEnvironmentPrivate::operator->()
{
    return jniEnv;
}
```
## JniEnv工具模块
在[C++中如何实现一个完整的Jni模块](http://peter517.github.io/2015/11/05/C++%E4%B8%AD%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AA%E5%AE%8C%E6%95%B4%E7%9A%84Jni%E6%A8%A1%E5%9D%97/#JNI_OnLoad)文章中有提到，可以有一个工具模块，来管理全局的JavaVM，其他Jni模块依赖这个工具模块来使用JNIEnv。上面的QJNIEnvironmentPrivate就是这个模块，代码如下，在JNI_OnLoad中在QtAndroidPrivate中保存JavaVM，其他模块中通过QtAndroidPrivate::javaVM()来获取JavaVM
```c++
Q_CORE_EXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved)
{
    //...

    if (vm->GetEnv(&uenv.venv, JNI_VERSION_1_6) != JNI_OK)
        return JNI_ERR;

    JNIEnv* env = uenv.nenv;
    //save the JavaVM , get it by QtAndroidPrivate::javaVM()
    const jint ret = QT_PREPEND_NAMESPACE(QtAndroidPrivate::initJNI(vm, env));
    if (ret != 0)
        return ret;

    return JNI_VERSION_1_6;
}
```
调用代码如下：
```c++
//other Jni module
void onTest(long value)
{
    QJNIEnvironmentPrivate env;
    jlong j_value = (jlong) value;
    jmethodID jmethod =env->GetMethodID(jclass_callback_global_, "onTest", "(J)V");
    env->CallVoidMethod(jobject_callback_global_, jmethod,j_value);
}
```
# 结论
如果不采用线程级资源管理的方式，就会在代码中布满了一对对AttachCurrentThread方法DetachCurrentThread，十分容易出Bug，对JNIEnv的封装也让其他Jni模块可以方便的使用Jni功能。每次写代码时，都考虑一下是否有线程级资源，是否需要在线程退出时自动释放相应资源



<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>
