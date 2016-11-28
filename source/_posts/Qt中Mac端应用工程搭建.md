title: Qt中Mac端应用工程搭建
tags: [Qt,Mac]
date: 2016-11-26 09:58:11
description: 利用qmake、macdeployqt、sparkle、codesign工具开发可发布的Mac客户端
---

# 工程建立

[Qt中PC端应用工程搭建](http://peter517.github.io/2016/11/28/Qt%E4%B8%ADPC%E7%AB%AF%E5%BA%94%E7%94%A8%E5%B7%A5%E7%A8%8B%E6%90%AD%E5%BB%BA/)

# 应用打包

## macdeployqt
利用qmake生成的App目录结构很简单，Contents下面只有Info.plist、MacOS、PkgInfo。
执行macdeployqt命令后会增加Frameworks、PlugIns、Resources目录，会把对应的库拷贝到Frameworks下面，依赖的资源拷贝到Resources下，但是有些依赖库和资源需要自己拷贝，比如opencv和sparkle

## install_name_tool
应用二进制依赖的库都是本地路径，这样发布的时候会找不到库，需要利用install_name_tool来进行依赖库的重连接，把之前依赖的系统路径改为应用当前路径，利用otool -L 可以查看一个二进制的依赖项跟对应的路径

# 应用签名

## 签名流程
为什么说是“可发布的”，因为Mac有一个签名机制，需要开发者用一个可信任的证书对App中**App二进制、framework、dylib**进行签名，这样才能让普通用户可以打开网页下载的App。Mac提供codesign这个命令
签名
```
codesign --force --sign “your_licence”  need_to_be_sign_part --deep
```
验证签名是否正确
```
spctl -a -v need_to_be_sign_part

```
所要做的事情很简单，用一个python脚本遍历所有framework、dylib并签名，最后对App二进制签名

## Framework签名目录要求
Framework的目录结构必须要符合Mac的规范才能正确签名，执行完macqtdeploy后Qt中的Framework目录只有Resources和Version，比如QtCore，目录结构如下
```
QtCore.framework
  ---Versions
    ---5
      ---QtCore
  ---Resources
```
需要调整为
```
QtCore.framework
  ---Versions
    ---5
      ---QtCore
      ---Resources
        ---Info.plist        
    ---Current -> 5
  ---Resources -> Versions/Current/Resources
  ---QtCore -> Versions/Current/QtCore
```
> Info.plist 是从QT的SDK中拿到

# 应用dmg打包
应用dmg打包可以通过macdeployqt，也可以通过dropdmg工具

# 在线升级

[Qt中PC端应用工程搭建](http://peter517.github.io/2016/11/28/Qt%E4%B8%ADPC%E7%AB%AF%E5%BA%94%E7%94%A8%E5%B7%A5%E7%A8%8B%E6%90%AD%E5%BB%BA/)


<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>
