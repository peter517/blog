title: 主流平台的反编译工具
tags: [decompile]
date: 2015-08-10 16:16:08
description: Unix、OSX、Windows平台中系统自带的反编译工具
---

反编译在工程中是绝对的利器，我们主要用它来做一下几件事情：
- 查找和修改二进制文件依赖的库路径
- 查看二进制文件符号表，检查是否包涵某个函数
- 查找二进制文件中的可打印字符串，可用来查看版本信息
- 将多个二进制文件合并成同一个二进制文件
- 将二进制文件不同系统架构版本合成同一个文件

# 二进制文件格式
## ELF
ELF是Unix体系中二进制文件格式

## PE

```bash
$ libtool -static -o $dst_lib_path $source_lib_path1 $source_lib_path2 ...
```

```bash
$ install_name_tool -change $file_path $file_depends_path_before $file_depends_path_after
```

```bash
$ otool -L $file_path
```

```bash
$ lipo -info $file_path
```
```bash
$ lipo -create $file_i386_path.a $file_arm_path.a -output $file_path
```


<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>