title: Unity常用第三方库
tags: [Unity]
date: 2016-09-18 19:27:48
description: Unity开发中常用的第三方库

---

相对于Android、IOS，Unity第三方库都是比较少的，而且成熟度不够，下面列举几个项目中经常会用到的第三方库

# Best Http
Unity对网络的支持主要是WWW类实现的，不好用，下面通过HttpClientProxy对Best Http进行了简单的封装，过程是异步的，用到了HTTPRequest、HTTPMethods、HTTPResponse类
```
public class HttpClientProxy {

    public delegate void OnRequestFinishedDelegate(HTTPResponse response);

    public event OnRequestFinishedDelegate OnRequestFinishedEvent;

    public void sendPostJson(string url, string json) {
        HTTPRequest request = new HTTPRequest(new Uri(url), HTTPMethods.Post, OnRequestFinished);
        request.AddHeader("Content-Type", "application/json;charset=utf-8");
        request.AddHeader("Accept", "application/json;charset=utf-8");
        request.RawData = System.Text.Encoding.UTF8.GetBytes(json);
        request.Send();
    }

    public void sendGet(string url) {
        HTTPRequest request = new HTTPRequest(new Uri(url), HTTPMethods.Get, OnRequestFinished);
        request.Send();
    }

    void OnRequestFinished(HTTPRequest request, HTTPResponse response) {
        if (OnRequestFinishedEvent != null) {
            OnRequestFinishedEvent(response);
        }
    }

}
```
注册了OnRequestFinishedEvent的类可以通过HTTPResponse拿到对应的信息
```
void OnGetMessageResponse(HTTPResponse response)
{
    Debug.Log("OnGetMessageResponse");
    if (response.StatusCode != 200)
    {
        Debug.Log("OnGetMessageResponse Failed" + response.StatusCode);
        return;
    }

    GetMessageResponse result = JsonUtility.FromJson<GetMessageResponse>(response.DataAsText);
}
```
# EasyMovieTexture
Unity设计目的是游戏开发，所以对视频播放支持不好，EasyMovieTexture提供移动端和PC端的SDK，接口比较简单，直接一个url就可以，或者一个放在StreamingAssets里面的视频文件名。
移动端都是调用原生的播放器，PC端用FFmpeg实现，还是Beta版本，移动端相对稳定一些。
# DOTween
DOTween用来实现3D的一些动画效果，比如一个物体从一个坐标移到另一个坐标，自己利用物理引擎实现比较麻烦。
DOTween的实现方式比较特别，利用C#的特性为现有类增加方法，比如下面这个函数，表示移动的距离，第一个参数表示为Transform类增加方法
```
public static Tweener DOMove (this Transform target, Vector3 endValue, float duration, bool snapping = false);
```
可以通过下面的调用来实现物体向下移动一个单位的动画
```
transform.DOMove(transform.position + Vector3.down, 2f);
```
# NGUI
NGUI是第三方的一套UI系统，包括事件处理，由于其他插件可能是基于原生的UGUI的实现的，所以决定NGUI使用还是要比较谨慎一些
# CurvedUI
CurvedUI用来实现菜单面板的曲面效果，VR界面设计经常会是曲面风格
# LitJson
LitJson是C#的Json实现，完成类和Json之间的映射，Best Http里面一般会包含这个库，但一般通过JsonUtility.ToJson（Object to Json String）和JsonUtility.FromJson<> （Json String to Object）是UnityEngine自带的方法就可以实现基本功能


<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>
