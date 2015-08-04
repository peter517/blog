title: 实用shell命令
date: 2015-08-04 09:31:19
tags: Shell
description: 平时用的比较多的shell命令

---

# 查看占用CPU最大的10个进程

```
$ ps au | awk 'NR>1 {print}'| sort -nrk +3 | awk '{print $3"\t"$11}' | head -10；
```
>其中sort中的k参数表示按照第几列来排序，有个t参数，表示按照\t分割内容，形成列

# 在某个时间点运行命令
```
$ echo "ls -l" | at midnight
```

# 显示某个目录下面最大的10个文件或文件夹
```
$ du -sh * | sort -nr | head -10
```
>其中s表示统计文件或文件夹容量总和，如果命令为du -ah *，表示统计当前目录文件和所有子目录的文件信息；与之相对的df命令用来统计硬盘容量信息

# 查看指定网络端口服务是否开启
```
$ netstat -natp | grep servicename
```
>其中n代表数字化，a代表所有，t代表tcp，p代表程序名,另一个查看网络服务的命令是lsof，列出系统已打开的文件，lsof -i | grep servicename，其中i表示查看所有网络服务打开的连接

# awk主要对文件内容进行统计操作
```
$ sed 's/localhost/127.0.0.1/g' mysql_virtual_*.cf 
```
>sed与vim中命令行操作类似，主要用来对文件进行处理（p选项+正则也可以完成强大的文本内容搜索）,如可以进行文本替换,%s/localhost/127.0.0.1/g,而在vim命令行中可以直接：%代表对文本中每一行都进行处理

# 输出最常用的10个命令
```
$ history|awk '{print $2}'|awk 'BEGIN {FS="|"} {print $1}'|sort|uniq -c|sort -rn|head -10
```
>其中history历史输入命令，uniq命令会从1列（元素）变成2列（个数，元素）

# 对指定文件夹中符合条件的文件进行操作
```
$ ls image/* | awk 'NR < 10 {system("cp "$1" uploadImage")}
```
>复制10个文件

```
$ find image/ -name "1*" -exec cp {} uploadImage \
```
>对以“1”开头的文件进行复制

```
$ ls image/* | head -10 | xargs -i cp {} uploadImage
```
>其中-i选项告诉 xargs 用每项的名称替换 {}

# 不登陆远程机器而进行远程操作
```
$ ssh user@server bash < /path/to/local/script.sh
```
>在远程机器上执行本地脚本

```
$ ssh user@host cat /path/to/remotefile | diff /path/to/localfile -
```
>比较本地和远程机器上面的两个文件

# 查看机器信息
```
$ dmidecode | grep "Product Name"
```
>查看机器型号

```
$ getconf LONG_BIT
```
>查看机器版本号（64位的系统中int类型还是4字节的，但是long已变成了8字节）

```
$ cat /proc/cpuinfo
```
>查看cpu信息，其中grep "core id"表示查看cpu内核id， grep "physical id"表示查看物理cpu的id，grep “processor”表示查看逻辑处理器。多核技术和多线程技术和区别为：前者表示有多少能力，后者表示使用该能力的能力）

```
$ cat /proc/meminfo
```
>查看内存信息

# 以列为单位抽取文件内容
```
$ cat /etc/passwd | cut -f1,3 -d":"
```
>利用cut命令可以对文件按照字节、字段、字符进行抽取，表示获取第一列和第三列，sed命令中1,3表示1至3列，而在cut中需要1-3来表示。awk命令也可以完成相同的内容，更复杂、更灵活； 

# 文件内容的查找与文件名的查找
```
$ grep -o str filename | wc -l
```
>文件名查找可以用经典的find命令，文件内容的查找则用grep。统计文件中出现某个字符串的个数，-o表示显示匹配的单词，而不是单词所在的行数：grep str filename

```
$ grep -l str filenames
```
>统计出现某个字符串的文件名

```
$ grep -i -R --include="*.cpp" 'example' 
```
>在当前目录及其子目录所有的.cpp文件中查找字符串"example", 不区分大小写