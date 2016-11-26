title: Qt中Mac端应用开发
tags: [Qt Mac]
date: 2016-11-26 09:58:11
description: 利用qmake、macdeployqt、sparkle、codesign工具开发可发布的Mac客户端
---

# 工程建立

qt建立工程的三个关键文件：
- YourApp.pro：管理工程源文件、库依赖、基本工程配置
- YourApp.qrc：管理资源
- YourApp.ui：管理布局

## YourApp.pro

## YourApp.qrc

## YourApp.ui

```
qmake -spec macx-xcode YourApp.pro
```
# 应用打包

# macdeployqt
利用qmake生成的App目录结构很简单，只有Contents下面只有Info.plist、MacOS、PkgInfo。
执行macdeployqt命令后会增加Frameworks、PlugIns、Resources目录，会把对应的库拷贝到Frameworks下面，依赖的Resources拷贝到Resources下，但是有些依赖库和资源需要自己拷贝

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

## 签名目录要求

# 应用dmg打包
应用dmg打包可以通过macdeployqt，也可以通过dropdmg工具

# 在线升级
在线升级目前开源的成熟跨平台框架是Sparkle，提供generate_keys和sign_update工具，通过xml文件信息更新来进行在线升级。

## sparkle流程
利用generate_keys生成dsa_pub.pem、dsa_priv.pem--->利用sign_update用dsa_priv.pem对dmg进行签名--->将签名信息和应用版本信息、下载链接更新到线上xml中--->旧的客户端通过检测到xml更新进行在线升级

## Sparkle如何保障应用签名
- 使用私钥dsa_priv.pem，对App进行计算，得出结果dsaSignature --- 算法1
- 将公钥dsa_pub.pem，放入app内，可以更加它对App进行计算，结果如果为dsaSignature，则验证通过 -- 算法2

算法1和算法2是公开的，有DSA、RSA、DES等等，Sparkle的签名原理是openssl命令实现的DSA算法

# Sparkle xml格式

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
			           sparkle:dsaSignature="MC0CFD+2Z3uk9GG0EoNhTUCQJ6U6WAnuAhUAqlbBoMIEhdcecPn9MWF1GxsilR4="
			           type="application/octet-stream"/>
		</item>
	</channel>
</rss>
```

> dsaSignature就是通过dsa_priv.pem + App生成的数字签名
> sparkle客户端通过version和shortVersionString来判断是否要升级

<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>
