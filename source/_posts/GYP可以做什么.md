title: GYP可以做什么？
tags: [GYP]
date: 2015-10-21 10:17:37
description: 介绍GYP的基本功能以及和CMake的比较
---

写的一手好代码固然难得，代码写好后可以方便的在各个平台完成编译部署，也需要找到好的工具，GYP就是其中一个，类似于CMake，提供了一套跨平台生成编译脚本或IDE工程的方案
如下图所示，通过编写GYP脚本，就可以在不同平台下面生成对于的编译脚本或着IDE工程，比如Unix平台下面的Makefile或Ninja，Windows平台下面的VS工程，IOS平台下面的Xcode工程，从而达到一键构建的目的

![cross-platform-script-framework](/img/cross-platform-script-framework.png)

# 简单实例
下面是一个简单的GYP脚本，通过它可以在不同的平台下面生成可my_target的可执行文件
```
 {
    'targets': [
      {
        'target_name': 'my_target',
        'type': 'executable',
        'sources': [
          'main.cc',
        ],
      },
    ],
  },
```
# 基本功能
跟CMake这种跨平台脚本生成工具一样，GYP提供一个编译脚本所需要的基本功能
## 定义宏
定义C中的宏，如下所示：
```
'defines': [
            'DEFINE_FOO',
        ],
```
## default变量
通过OS变量判断当前操作系统平台，如下所示：
```
'conditions': [
          ['OS=="linux"', {
            ...
          ],
          ['OS=="win"', {
            ...
          }, { # OS != "win",
            ...
          }]
        ],
```
除了OS变量，PRODUCT_DIR变量表示编译生成文件路径，DEPTH表示文件执行路径
## 模块依赖
依赖其他GYP里面的target，如下所示，bar为bar.gyp里面的一个target\_name：
```
'dependencies': [
          '../bar/bar.gyp:bar',
        ],
```
## 变量定义
定义变量，用来进行项目配置，如下所示：
```
'variables': {
      'is_enable_video': 'true'
    },
```
## include其他gyp文件
include其他gypi文件，如下所示：
```
 'includes': [
      '../build/common.gypi',
    ],
```
## 包含头文件搜索路径
如下所示：
```
'include_dirs': [
          'include',
        ],
```
## cflag设置
如下所示：
```
'cflags': [
            '-Werror',
            '-Wall',
         ],
```
## shell脚本执行
在脚本里面执行shell命令，如下所示：
```
"defines": [
              'BUILD_DATE=<！(echo `date +%Y%m%d`)'
            ],
```
# 特有功能
## action
GYP出了支持生成编译C的脚本外，还支持第三方模块执行的接口action，这样就可以考虑到利用GYP编译其他语言，比如用javac编译Java、用ant来Android App、用yasm来编译汇编语言等。利用这个特性，还可以用来执行单元测试，下面就是在每次编译的时候，**根据“inputs”文件（func.cc、func.h、func_test_main.cc）是否有改变**，来执行单元测试out/Release/fun_test
```
'targets': [
    {
        'target_name': 'fun_test',
        'type': 'executable',
        'sources': [
           'test/func_test_main.cc',
        ],
        'actions': [
         {
            'action_name': 'run_fun_test',
                'inputs': [
                    'func.h',
                    'func.cc',
                    'test/func_test_main.cc',
                ],
                'outputs': [ 'fun_test.txt' ],
                'action': [
                    'out/Release/fun_test',
                    ],
                },
         ],
},]
```
## direct_dependent_settings
direct_dependent_settings表示依赖这个模块的模块也自动包含这个属性，比如入unit\_test这个模块依赖gtest这个模块，那么unit\_test会被设置“include_dirs”这个属性，把gtest这个模块中include的绝对路径添加为头文件搜索路径
```
'targets': [
    {
        'target_name': 'gtest',
        'type': 'static_library',
        'include_dirs': [
            'include',
            '.',
        ],
        'sources': [
            'include/gtest.h',
            'source/gtest.cpp',
        ],
      'direct_dependent_settings': {
             'include_dirs': [
              'include',
           ]
        },
    },
]
```
# GYP Vs CMake
下图为GYP和CMake五个特性之间的比较：
![GYP_Vs_CMake](/img/GYP_Vs_CMake.png)
- 可读性：相比较于CMake的脚本瀑布式书写方式，GYP的书写方式类似于Json，可读性比较强
- 扩展性：action特性可以让GYP可以整合其他编译器，让C语言和其他语言一起一键编译
- 模块化：GYP以targets为书写单元，模块性很好，CMake通过一行语句添加依赖的模块目录，GYP通过dependencies属性来设置依赖的targets
- 源文件批量添加：GYP对批量添加源文件支持不好，CMake通过aux\_source\_directory命令轻松添加
- 流行度：CMake在很多成熟的项目中使用，也有很多配套的工具，GYP起步比较晚，这块还需努力


<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>