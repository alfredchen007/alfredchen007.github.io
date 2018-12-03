---
layout: post

title: "missing xcrun：mac下git工具修复"

date: 2018-12-3 10:50:20 +0300

description:  

cover: 'https://ws2.sinaimg.cn/large/006tNbRwly1fxtdo8ba4rj31400p07wh.jpg'

color: rgb(225,215,0)

tags: [git]
---

![](https://ws2.sinaimg.cn/large/006tNbRwly1fxtdo8ba4rj31400p07wh.jpg)

### git clone 出现的问题

可能是由于最近升级到了Mojave(10.14.1)的原因，mac下运行git命令时出现报错提示

```shell
AlfreddeMacBook-Pro:uploadFile alfred$ git clone git@github.com:*******
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
```

怀疑git包被破坏了，使用brew尝试安装

```shell
AlfreddeMacBook-Pro:uploadFile alfred$ brew install git
==> Downloading https://homebrew.bintray.com/bottles/git-2.18.0.mojave.bottle.tar.gz
Updating Homebrew...
######################################################################## 100.0%
==> Pouring git-2.18.0.mojave.bottle.tar.gz
==> Caveats
Bash completion has been installed to:
  /usr/local/etc/bash_completion.d

zsh completions and functions have been installed to:
  /usr/local/share/zsh/site-functions

Emacs Lisp files have been installed to:
  /usr/local/share/emacs/site-lisp/git
==> Summary
🍺  /usr/local/Cellar/git/2.18.0: 1,488 files, 296.7MB
Error: Git must be installed and in your PATH!
Warning: git 2.18.0 is already installed and up-to-date
To reinstall 2.18.0, run `brew reinstall git`
```

结果提示Error，接着尝试重装

```shell
AlfreddeMacBook-Pro:uploadFile alfred$ brew reinstall git
==> Reinstalling git 
==> Downloading https://homebrew.bintray.com/bottles/git-2.18.0.mojave.bottle.tar.gz
Already downloaded: /Users/alfred/Library/Caches/Homebrew/downloads/569b15b4cd875d89913341d8a9a8dd1d07159445dc1734e27539dbb40d121e9e--git-2.18.0.mojave.bottle.tar.gz
==> Pouring git-2.18.0.mojave.bottle.tar.gz
==> Caveats
Bash completion has been installed to:
  /usr/local/etc/bash_completion.d

zsh completions and functions have been installed to:
  /usr/local/share/zsh/site-functions

Emacs Lisp files have been installed to:
  /usr/local/share/emacs/site-lisp/git
==> Summary
🍺  /usr/local/Cellar/git/2.18.0: 1,488 files, 296.7MB
```

重装似乎没什么问题，但是使用git命令时依然出现`xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun`的错误信息

尝试`xcode-select`重装

```
AlfreddeMacBook-Pro:uploadFile alfred$ xcode-select --install
xcode-select: note: install requested for command line developer tools
AlfreddeMacBook-Pro:uploadFile alfred$ 
```

在弹出的gui界面中选择安装，等待安装完成

![](https://ws2.sinaimg.cn/large/006tNbRwly1fxtdi1tdf6j30ry078abi.jpg)

再次使用git命令，正常。