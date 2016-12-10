title: Unity插件编写
tags: [Unity]
date: 2016-12-10 12:01:27
description: 介绍如何在Unity中调用平台原生方法
---

Unity作为跨平台的游戏引擎，支持很多跨平台的方法调用，但Unity重点还是在游戏开发，有些接口无法从Unity层面拿到，比如是设备全局音量控制接口、Android中的Activity跳转接口，本文以实现设备全局音量控制这个功能，来介绍如何编写Unity移动端插件，PC端插件类似，这里就不再赘述

# 工程结构
Unity插件都是在对应平台编译好，打包成jar、动态库、静态库后，放在工程Assets/Plugin/Android、Assets/Plugin/IOS下面，Unity会自动把插件加载起来

# 基础类定义
## Android
Unity调用Android接口可以是jar包，也可以试so文件，Unity提供调用Android接口方法很像Jni，下面通过GetUnityActivity()拿到当前App的Activity，这样插件可以基于这个Activity调用一些系统接口，GetJavaObject()方法相当于Jni中的JniEnv，用来调用对应插件的方法
```
public abstract class AndroidPluginBase {

	private const string UNITY_PLAYER_ANDROID_OBJECT_NAME = "com.unity3d.player.UnityPlayer";
	private const string UNITY_PLAYER_ACTIVITY_NAME = "currentActivity";

	protected abstract string getAndroidObjectEntryName ();

	private AndroidJavaObject javaObj = null;
	protected AndroidJavaObject GetJavaObject()
	{
		if (javaObj == null)
		{
			javaObj = new AndroidJavaObject(getAndroidObjectEntryName());
		}
		return javaObj;
	}

	protected AndroidJavaObject GetUnityActivity()
	{
		return new AndroidJavaClass(UNITY_PLAYER_ANDROID_OBJECT_NAME).GetStatic<AndroidJavaObject>(UNITY_PLAYER_ACTIVITY_NAME);
	}
}
```
## IOS
IOS插件一般是以源文件的方式集成进来，通过Unity生成的xcode工程一起编译。GetRootControllerAddress()类似于Android里面获取Activity引用，插件拿到这个后可以调用IOS的一些方法
```
public class IOSPluginBase{


	[DllImport("__Internal")]
	private static extern long _getRootControllerAddress ();
	protected long GetRootControllerAddress()
	{
		return _getRootControllerAddress ();
	}

}
```

# 插件调用模块
## 插件要实现的接口
各个平台需要实现的接口
```
public interface IVolumeCtrlDevice{
	 float GetVolume ();
	 float SetVolume (float volume);
}
```

## 对Unity其他模块的接口
统一接口更加当前运行平台选择对应的插件实现，其中EditorVolumeCtrlDevice是一个空实现，保证在Editor状态下面可以编译通过
```
public class VolumeCtrlNative{

	private const string TAG = "VolumeCtrlNative";
	private IVolumeCtrlDevice volumeCtrlDevice;
	public VolumeCtrlNative()
	{
		#if UNITY_EDITOR
			volumeCtrlDevice = new EditorVolumeCtrlDevice();
		#elif UNITY_IPHONE
			volumeCtrlDevice = new IOSVolumeCtrlDevice();
		#elif UNITY_ANDROID
			volumeCtrlDevice = new AndroidVolumeCtrlDevice();
		#endif
	}

	public static VolumeCtrlNative INSTANCE{
		get {
			if (instance == null) {
				instance = new VolumeCtrlNative();
			}
			return instance;
		}
	}

	private static VolumeCtrlNative instance = null;

	public float GetVolume(){
		return volumeCtrlDevice.GetVolume ();
	}

	public float SetVolume(float volume){
		if (volume > 1.0 || volume < 0.0) {
			return -1.0f;
		}
		return volumeCtrlDevice.SetVolume (volume);
	}

}

```
## Android
ANDROID_OBJECT_ENTRY_NAME表示插件的实现包名和类名，CallGetVolume()的实现跟Jni的调用很类似
```
public class AndroidVolumeCtrlDevice : AndroidPluginBase,IVolumeCtrlDevice
{
	private const string ANDROID_OBJECT_ENTRY_NAME = "com.yourcompany.VolumeCtrl";
	protected override string getAndroidObjectEntryName ()
	{
		return ANDROID_OBJECT_ENTRY_NAME;
	}

	public AndroidVolumeCtrlDevice()
	{
		CallSetUnityActivity();
	}

	public float GetVolume()
	{
		return CallGetVolume();
	}

	public float SetVolume(float volume)
	{
		return  CallSetVolume(volume);
	}

	private void CallSetUnityActivity()
	{
		GetJavaObject().Call("setUnityActivity", GetUnityActivity());
	}

	private float CallGetVolume()
	{
		return GetJavaObject().Call<float>("getVolume");
	}

	private float CallSetVolume(float volume)
	{
		return GetJavaObject().Call<float>("setVolume", volume);
	}
}
```
## IOS
IOS的接口就是C#调用c方法的实现方式
```
public class IOSVolumeCtrlDevice : IOSPluginBase,IVolumeCtrlDevice {

	public IOSVolumeCtrlDevice()
	{
		SetRootControllerAddress();
	}

	[DllImport("__Internal")]
	private static extern float _media_player_volume_get_volume ();
	[DllImport("__Internal")]
	private static extern float _media_player_volume_set_volume (float volume);
	[DllImport("__Internal")]
	private static extern float _media_player_set_root_control(long rootControllerAddress);

	public float GetVolume()
	{
		return _media_player_volume_get_volume ();
	}

	public float SetVolume(float volume)
	{
		return _media_player_volume_set_volume (volume);
	}

	private void SetRootControllerAddress()
	{
		_media_player_set_root_control(GetRootControllerAddress ());
	}
}

```

# 插件实现
## Android
### 基础类
```
public class BasePlugin {
    protected Activity activity;
    public void setUnityActivity(Activity activity) {
        this.activity = activity;
    }
}
```
### 具体实现
通过Unity传进来的Activity实例来调用系统方法获取全局音量
```
public class VolumeCtrl extends BasePlugin{

    private String TAG = this.getClass().getSimpleName();

    public float getVolume() {

        if (activity == null) {
            return;
        }

        AudioManager mgr = (AudioManager) activity.getSystemService(Context.AUDIO_SERVICE);
        final float MAX_STREAM_MUSIC_VOLUME = mgr.getStreamMaxVolume(AudioManager.STREAM_MUSIC);

        float volume = mgr.getStreamVolume(AudioManager.STREAM_MUSIC);

        //change volume to 0.0 -> 1.0
        float normalizedVolume = volume / MAX_STREAM_MUSIC_VOLUME;
        return normalizedVolume;

    }

    public float setVolume(float volume) {

        Log.d(TAG, "setVolume volume " + volume);

        if (activity == null) {
            return -1.0f;
        }

        if (volume > 1.0f || volume < 0.0f) {
            return -1.0f;
        }

        AudioManager mgr = (AudioManager) activity.getSystemService(Context.AUDIO_SERVICE);
        final float MAX_STREAM_MUSIC_VOLUME = mgr.getStreamMaxVolume(AudioManager.STREAM_MUSIC);
        mgr.setStreamVolume(AudioManager.STREAM_MUSIC, (int) (volume * MAX_STREAM_MUSIC_VOLUME), 0);
        return volume;
    }

}

```
## IOS
### 定义接口
```
@interface VolumeCtrl : NSObject

+ (VolumeCtrl*) getInstance;

- (int ) setRootController:(long)rootControllerAddress;
- (float) getVolume;
- (float) setVolume:(float)volume;
@end
```
### 具体实现
与Unity调用相关的接口通过extern "C"来定义，通过Unity传进来的rootController来呼出全局音量的组件
```
extern "C" {

    int _media_player_set_root_control(long rootControllerAddress) {
        return [[VolumeCtrl getInstance] setRootController:rootControllerAddress];
    }

    float _media_player_volume_get_volume() {
        return [[VolumeCtrl getInstance] getVolume];
    }

    float _media_player_volume_set_volume (float volume) {
        return [[VolumeCtrl getInstance] setVolume:volume];
    }
}

@interface VolumeCtrl ()
@property (nonatomic,strong)UISlider *slider;
@end

@implementation VolumeCtrl

static VolumeCtrl * instance;
+ (VolumeCtrl*) getInstance {
    @synchronized(self) {
        if(instance == nil) {
            instance = [[self alloc]init];
        }
    }
    return instance;
}
- (int ) setRootController:(long)rootControllerAddress
{

    UIViewController* rootViewController = (__bridge UIViewController *)((void*)rootControllerAddress);

    MPVolumeView *volumeView = [[MPVolumeView alloc] init];
    [rootViewController.view addSubview: volumeView];
    [volumeView sizeToFit];
    [rootViewController.view bringSubviewToFront: volumeView];

    _slider = [[UISlider alloc]init];
    _slider.backgroundColor = [UIColor blueColor];
    for (UIControl *view in volumeView.subviews) {
        if ([view.superclass isSubclassOfClass:[UISlider class]]) {
            self.slider = (UISlider *)view;
        }
    }
    _slider.autoresizesSubviews = NO;
    _slider.autoresizingMask = UIViewAutoresizingNone;
    volumeView.alpha = 0.001;

    return 0;
}

- (float) getVolume
{
    AVAudioSession *audioSession = [AVAudioSession sharedInstance];
    return audioSession.outputVolume;
}
- (float) setVolume:(float)volume
{
    assert(_slider != nil);
    _slider.value = volume;
    return _slider.value;
}

@end

```
<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>
