title: Qt中Windows端应用工程搭建
tags: [Qt,Windows]
date: 2016-11-28 17:28:09
description: 利用qmake、nsis、sparkle、工具开发可发布的Windows客户端
---

# 工程建立

[Qt中PC端应用工程搭建](http://peter517.github.io/2016/11/28/Qt%E4%B8%ADPC%E7%AB%AF%E5%BA%94%E7%94%A8%E5%B7%A5%E7%A8%8B%E6%90%AD%E5%BB%BA/#工程建立)

# 链接
## Debug or Release
Window App在编译Debug or Release模式时会找对应的Qt包，不然QtCored.dll是Debug需要的，QtCore.dll是Release需要的，在应用打包环境需要注意

## 其他SDK集成
- OpenCV编译出来的版本一般有两个维度，除了常见的CPU位数，还有VS版本，开发App和打包阶段需要把App的VS环境和CPU位数考虑进去
- 当集成许多SDK时，保证每个SDK都统一成Debug版本或者Release版本

# NSIS应用打包

NSIS是Windows上面的开源打包工具，用类似汇编和Shell混合的语言来构建一个安装程序，支持c、c++语言实现的插件，功能很全。

## 架构
NSIS以每个页面为单元构建一个安装包，开发者在对应页面中完成相应代码，NSIS用;代表注释

### Page和UninstPage
```
Page custom WelcomePage
Page custom InstallPage
Page custom FinishPage

UninstPage custom un.WelecomPage
UninstPage custom un.UnInstallPage
UninstPage custom un.FinishPage

Function WelcomePage
    ;TODO
End
```

### 生命周期
.onInit --> .onGUIInit --> Pages -->  un.onInit --> un.onGUIInit  --> un.Pages
```
!define MUI_CUSTOMFUNCTION_GUIINIT onGUIInit
!define MUI_CUSTOMFUNCTION_UNGUIINIT un.onGUIInit

Function .onInit
End

Function .onGUIInit
End

Function un.onInit
End

Function un.onGUIInit
End
```

### 翻页
```
Function Next_Page
  IntCmp $R9 0 0 Move Move
    StrCmp $R9 "X" 0 Move
      StrCpy $R9 "120"
  Move:
  SendMessage $HWNDPARENT "0x408" "$R9" ""
FunctionEnd
```
> 官方例子

## Shell风格
NSIS中只有两个基本概念，变量和函数，和Shell很类似，变量从来源上分为系统变量、UI变量，函数分为系统函数和自定义函数

### 基本信息配置
```
Name "YourAppName"
OutFile "Your-Setup.exe"
InstallDir "YourInstallDir"
XPStyle on
RequestExecutionLevel admin ;要求系统权限执行
```
> NSIS框架会读取这些信息来配置安装包

### 系统函数
```
CreateDirectory $YourDir; 创建目录
Exec YourApp.exe ; 运行文件
ExecShell "open" Your_URL ; 打开网页
CreateShortCut $YourShortCut; 创建快捷方式
SendMessage $Pb_Uninstall ${PBM_SETPOS} 80 0 ;更新进度条
DetailPrint "WindowsVersion:"; 打印日志
```
> $Pb_Uninstall是组件变量，${PBM_SETPOS} 是系统变量，与UI组件交互一般用SendMessage方法

### 自定义函数
```
Function Your_Method
    ;TODO
FunctionEnd

Function SomPage_Method
    Call Your_Method
FunctionEnd
```

## 汇编风格
通过Pop和Push命令和系统、第三方插件方法进行数据交互，很汇编的写法，常用Pop方法获取系统调用结果。
### 判断互斥
```
System::Call 'kernel32::CreateMutex(i 0, i 0, t "${INSTALL_NAME}") ?e'
	Pop $R0
	StrCmp $R0 0 +3
    MessageBox MB_OK "Already Running!"
    Abort
```
> Pop命令获取System::Call的结果，存放到$R0里面

### 创建一个组件及其注册事件
```
  ${NSD_CreateButton} 454 13 14 14 ""
	Pop $Bt_CloseDialog
	GetFunctionAddress $3 OnClick_CloseDialog ; 获取自定义OnClick_CloseDialog方法的地址
	SkinBtn::onClick $Bt_CloseDialog $3 ;把OnClick_CloseDialog设置为SkinBtn::onClick的响应事件
```
> ${NSD_CreateButton}里面存放一个创建UI的方法

### 组件联动
NSIS中很多组件的效果需要开发人员完成，例如下面的Checkbox效果，通过Checkbox来控制某个按钮是否可以点击，
```
  ${IF} $IS_AgreeLicence == 1
		IntOp $IS_AgreeLicence $IS_AgreeLicence - 1
		EnableWindow $Bt_Install 0 ;disable Bt_Install 组件
		StrCpy $1 $Cb_AgreeLicence
		Call SkinBtn_UnChecked ; 调用自定义方法并传入Checkbox组件
	${ELSE}
		IntOp $IS_AgreeLicence $IS_AgreeLicence + 1
		EnableWindow $Bt_Install 1
		StrCpy $1 $Cb_AgreeLicence ; enable Bt_Install 组件
		Call SkinBtn_Checked ; 调用自定义方法并传入Checkbox组件
	${EndIf}
```

### 隐藏组件
```
GetDlgItem $0 $HWNDPARENT 1034
ShowWindow $0 ${SW_HIDE}
```
> $HWNDPARENT和${SW_HIDE}是系统变量

### Kill进程
```
	StrCpy $0 "YourAppName"
	KillProc::KillProcesses
```
> 把AppName放入$0中，KillProc::KillProcesses从$0里取参数

# 在线升级

[Qt中PC端应用工程搭建](http://peter517.github.io/2016/11/28/Qt%E4%B8%ADPC%E7%AB%AF%E5%BA%94%E7%94%A8%E5%B7%A5%E7%A8%8B%E6%90%AD%E5%BB%BA/#在线升级)


<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>
