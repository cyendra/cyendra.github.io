---
layout: post
title: Linux 下工作环境配置
categories: environment
description: Linux 下，个人常用环境配置
keywords: linux
---

# zsh 配置

## 安装 zsh
官网：[zsh](https://github.com/robbyrussell/oh-my-zsh)
```
sudo yum install zsh
或
sudo apt-get install zsh
```
设为默认 shell
```
chsh -s `which zsh`
```

## 安装 oh-my-zsh
直接从 git 上下载安装
```
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
cd oh-my-zsh/tools
./install.sh
```
或使用命令安装
```
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

## zsh 主题设置
在 ~/.zshrc 下设置主题，默认就好
```
ZSH_THEME="robbyrussell"
```
安装一些其他插件，插件介绍：[插件](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins)
```
目前在用的插件：
plugins=(git autojump extract d osx z)
```

# vim with lua 安装

## 前置安装

### 直接 yum 安装 
```readline 和 ncurses
yum -y install readline-devel ncurses-devel
```

### 手动安装

#### 安装 ncurses
官网：[ncurses](http://ftp.gnu.org/pub/gnu/ncurses/)
```
wget http://ftp.gnu.org/pub/gnu/ncurses/ncurses-6.0.tar.gz
tar -xzvf ncurses-6.0.tar.gz
cd ncurses-6.0
./configure
sudo make
sudo make install
```

#### 安装 readline
官网：[readline](http://cnswww.cns.cwru.edu/php/chet/readline/rltop.html)
```
wget ftp://ftp.cwru.edu/pub/bash/readline-7.0.tar.gz
cd readline-7.0
./configure
sudo make
sudo make install
```

## 安装 lua
官网：[lua](http://www.lua.org/download.html)
```
curl -R -O http://www.lua.org/ftp/lua-5.3.4.tar.gz
tar zxf lua-5.3.4.tar.gz
cd lua-5.3.4
make linux
sudo make install
```

## 安装 vim
官网：[vim](https://vim.sourceforge.io/download.php)
```
wget ftp://ftp.vim.org/pub/vim/unix/vim-7.4.tar.bz2
tar -xf  vim-7.4.tar.bz2
cd vim74
./configure --prefix=/usr --with-features=huge --enable-rubyinterp --enable-pythoninterp --enable-luainterp --with-lua-prefix=/usr/local
sudo make
sudo make install
```
查看 configure
```
at log | grep lua
checking --enable-luainterp argument... yes
checking --with-lua-prefix argument... /usr/local
checking --with-luajit... no
checking for lua... /usr/local/bin/lua
checking if lua.h can be found in /usr/local/include... yes
checking if link with -L/usr/local/lib -llua is sane... yes
```
问题
```
undefined reference to luaL_optlong
```
解决方法，修改vim74/src/if_lua.c
```
//long pos = luaL_optlong(L, 3, 0);
long pos = （long)luaL_optinteger(L, 3, 0);
```

## 验证是否成功
```
vim
:version
```
有 +lua 说明安装成功

# spf13-vim 安装

## 简介
官网：[spf13-vim](http://vim.spf13.com/)
github：[spf13-vim](http://github.com/spf13/spf13-vim)

## 前置
TagToggle 依赖 ctags，首先要安装 ctags
```
sudo yum install ctags
```

## 安装
安装官网说明进行安装
```
curl https://j.mp/spf13-vim3 -L > spf13-vim.sh && sh spf13-vim.sh
```

# 安装 blade
## 简介
官网：[blade](https://github.com/chen3feng/typhoon-blade)

## 前置
依赖 scons，官网：[scons](https://sourceforge.net/projects/scons/files/scons)
```
python setup.py install
```

其他问题：  
/usr/bin/ld: cannot find -lstdc++
```
sudo yum install libstdc++-static
```

## 安装
```
git clone 
cd typhoon-blade
./install
```