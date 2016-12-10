title: Qt中PC端应用工程搭建
tags: [Qt,Mac,Windows]
date: 2016-11-28 17:13:14
description: Qt中PC端工程搭建、在线升级功能
---

Qt作为跨平台UI开发框架，用c++实现，API也是类C的，很适合搭建一些集成Native SDK的演示环境，如果只是一些UI的原型展示，用Electron或者nwjs更快

# 工程建立

qt建立工程的三个关键文件：
- YourApp.pro：管理工程源文件、库依赖、基本工程配置
- YourApp.qrc：管理资源
- YourApp.ui：管理布局

## YourApp.pro
在Mac上面生成xcode工程很简单，下面一行命令就行
```
qmake -spec macx-xcode YourApp.pro
```
pro文件跟[GYP](http://peter517.github.io/2015/10/21/GYP%E5%8F%AF%E4%BB%A5%E5%81%9A%E4%BB%80%E4%B9%88/)、CMake是一类工具，最大区别是内置QT的工程配置，重要的几个配置项：

### 基本信息
```
TARGET = YourAppName #App名字
TEMPLATE = app #App类型
QT  += core gui widgets #依赖的Qt库
ICON = YourIcon.icns #Icon路径
CONFIG += YOUR_BUILD_MODE #编译模式
```
### UI
```
FORMS += YourLayout.ui #布局文件
```

### c++配置

GCC的写法
```
mac{
  QMAKE_MAC_SDK = macosx10.11
  QMAKE_MACOSX_DEPLOYMENT_TARGET = 10.11
  QMAKE_CXXFLAGS += -stdlib=libstdc++
  QMAKE_LFLAGS += /usr/lib/libstdc++.6.dylib
  QMAKE_INFO_PLIST = YourInfo.plist
}
```
仿[GYP](http://peter517.github.io/2015/10/21/GYP%E5%8F%AF%E4%BB%A5%E5%81%9A%E4%BB%80%E4%B9%88/)，CMake的写法
```
HEADERS += your_header.h
SOURCES += your_c_source.cpp
OBJECTIVE_SOURCES += your_object_c_source.cpp
INCLUDEPATH += YOUR_INCLUDE_PATH
LIBS += YOUR_DEPEND_LIBS
```

### Mac Resources
```
RESOURCES += YourApp.qrc #资源文件
RESFILES.files = files_add_to_resource_dir #需要加入资源目录的问题件
RESFILES.path = Contents/Resources
QMAKE_BUNDLE_DATA += RESFILES
```

### Qt生成文件配置
qt在xcode运行时会根据layout文件生成对应的代码

```
QT_RUN_GENERATE_FILE_DIR = ./qt-run-generate-files  #指定生成代码目录
UI_DIR +=  $$QT_RUN_GENERATE_FILE_DIR/ui #UI代码
RCC_DIR += $$QT_RUN_GENERATE_FILE_DIR/rc #资源代码
OBJECTS_DIR += YOUR_BUILD_MODE
MOC_DIR += $$QT_RUN_GENERATE_FILE_DIR/YOUR_BUILD_MODE
```

## YourApp.qrc
App中用到的资源文件需要在这里面定义，用来UI布局，如下例：
```
<RCC>
    <qresource prefix="/">
        <file>app_images/add.png</file>
    </qresource>
    <qresource prefix="/value">
        <file>en_US.qm</file>
        <file>zh_CN.qm</file>
    </qresource>
</RCC>
```
## YourApp.ui
这个文件是xml格式，相当于android的layout

# 在线升级
在线升级目前开源的成熟跨平台框架是Sparkle，提供generate_keys和sign_update工具，通过xml文件信息更新来进行在线升级。

## Sparkle在线升级流程
利用generate_keys生成dsa_pub.pem、dsa_priv.pem--->利用sign_update用dsa_priv.pem对dmg进行签名--->将签名信息、应用版本信息、下载链接更新到线上xml中--->旧客户端通过检测到xml更新进行在线升级

## Sparkle如何保障应用签名
- 使用私钥dsa_priv.pem，对App进行计算，得出结果dsaSignature --- 算法1
- 将公钥dsa_pub.pem，放入app内，可以更加它对App进行计算，结果如果为dsaSignature，则验证通过 -- 算法2

算法1和算法2是公开的，有DSA、RSA、DES等等，Sparkle的签名原理是openssl命令实现的DSA算法

## Sparkle xml格式

```
<?xml version="1.0" encoding="utf-8"?>
<rss version="2.0" xmlns:sparkle="http://www.andymatuschak.org/xml-namespaces/sparkle"
     xmlns:dc="http://purl.org/dc/elements/1.1/">
	<channel>
		<item>
			<description>
				<![CDATA[
            <h3></h3> <pre> your_update_log </pre>
            ]]>
			</description>
			<enclosure url="your_app_download_url"
			           sparkle:version="20161123"
			           sparkle:shortVersionString="1.1.0"
			           sparkle:dsaSignature="dsaSignature"
			           type="application/octet-stream"/>
		</item>
	</channel>
</rss>
```

> dsaSignature就是通过dsa_priv.pem + App生成的数字签名
> sparkle客户端通过version和shortVersionString来判断是否要升级


<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>
