title: 基于dp的Android UI 布局适配
tags: [Android]
date: 2015-08-06 09:01:06
description: 用dp来做Android适配

---

# 分辨率布局的局限 
Android设备分辨率千奇百怪，基于pixcel的布局无法在不同的分辨率有一致的显示效果，如下图所示，要让一个button在320\*240和640\*480的两种分辨率下有一致的布局，设置相同的pixcel行不通。

![android_ui_adaptor_01](/img/android_ui_adaptor_01.png)

为减少开发人员的适配工作量，Android引入了dp的概念，在xml布局文件中可以直接设置单位为dp的值，要了解dp，先要了解下面几个概念。

# dip和desity
## dip
**dip**：每英寸像素数，`这个概念和Android设备分辨率一样，和连接Android设备的显示器尺寸是没有关系的（很多博客上面用手机的尺寸和分辨率来计算dip是不对的，本末倒置了），只表明android设备的显示输出能力`，如果一个Android设备分辨率为640\*480，每英寸像素数为320，那么连接这个android设备的显示器最佳尺寸为：
```
(640/320) * (480/320) ＝ 2英寸 * 1.5英寸
```
目前Android设备上dip的主要值是120 dpi、160 dpi、240 dpi、320 dpi 如果显示器尺寸大于或小于这个尺寸，显示器模块负责缩放。
## desity
**desity**：像素显示密度，基于dip的一个衍生概念，是dip归一化的值，计算方法为:
```
desity ＝ dip / 160
```
目前Android设备上desity主要值为0.75、1、1.5、2、3，Android中可以用代码来获取：
```
float density = getContext().getResources().getDisplayMetrics().density; 
```

# dp表示缓解适配压力

dp的计算就是基于分辨率和desity的，如果一个Android设备分辨率为640\*480，desity为1.5，那么屏幕的dp为:
```
(640/1.5) * (480/1.5) = 320dp * 240dp
```
代码中可以根据分辨率和desity来计算dp
屏幕可以用分辨率划分，引入dp的概念后，可以用dp来划分屏幕大小，如果屏幕大小为如果两个设备之间分辨率不一样，只要分辨率和desity的比例是一样的，就可以，比如之前的320\*240的手机设备，如果它的desity为1，那么它屏幕的dp为：
```
(320/1) * (240/1) = 320dp * 240dp
```
这样两个Android设备对屏幕都有同样的表达方式：320dp * 240dp，基于dp的布局就可以这样：
![android_ui_adaptor_02](/img/android_ui_adaptor_02.png)

# 没有完美解决方案
## dp也做不到的事情
不同设备之间只有用拥有统一的dp大小才能共享同一个布局方案，如果dp不一样的情况，那么就需要针对不同的dp进行处理了，android下面有一个values目录，里面通过dimens.xml来设置布局的dp
```
values-w540dp-hdpi
```
格式如上所示w540dp代表宽度为540dp,hdpi代表density为1，针对不同的dp，需要新建对应的目录，App会在运行时根据设备情况加载对应的目录里面的dp值

## 建议
- 如果可以不用数字来布局的尽量不要用，可以用百分比来代替：[开源库](https://github.com/JulienGenoud/android-percent-support-lib-sample)
- 不同的手机里面dp之间差距不大，布局之间的差异不明显，但是现在的智能电视和手机之间dp差距大，需要做好适配

# 参考资料
- http://developer.android.com/guide/practices/screens_support.html

<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>