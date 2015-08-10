title: Git 个人配置
tags: [Git]
date: 2015-08-10 14:18:28
description: Git中比较方便的优化配置
---

# 快捷键配置
## Git自带alias配置
```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch
```
## 自定义alias配置
```bash
alias gitcommit="git commit -m"
alias gitlog="git log"
alias gitshow="git show"#查看修改内容
alias gitcat="git cat-file"
alias gitloggraph="git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset'"#以图形方式显示提交日志
alias gitinfo="git remote show origin"#查看仓库信息
alias gitshortlog="git shortlog --since=1.day.ago"#查看最近一天的提交日志
alias gitweekshortlog="git shortlog --since=1.week.ago --author='pengjun' | grep -v Merge | uniq"#查看最近一周pengjun的提交日志
alias gitfilelog="git log --follow -p"#查看文件的修改日志
```

# 在命令行提示符后面显示git分支
## Unix
```bash
parse_git_branch() {
    git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}

export PS1="\u@\h \w\[\033[32m\]\$(parse_git_branch)\[\033[00m\] $ "
```
## OSX
```bash
parse_git_branch() {
    git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}
RED="\[\033[0;31m\]"
YELLOW="\[\033[0;33m\]"
GREEN="\[\033[0;32m\]"
NO_COLOR="\[\033[0m\]"

export PS1="\w$GREEN\$(parse_git_branch)$NO_COLOR\$ "
```

# meld配置
meld是Git本地合并冲突时候的UI界面，在~/.gitconfig里面进行如下配置
```bash
[difftool "meld"]
path = /usr/local/bin/meld
trustExitCode = false
[difftool]
prompt = false
[diff]
tool = meld
[mergetool "meld"]
path = /usr/local/bin/meld
trustExitCode = false
[mergetool]
keepBackup = false
[merge]
tool = meld
```
git merge操作后执行git mergetools，如果本地有冲突，就会可以通过meld界面来进行冲突合并

<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>