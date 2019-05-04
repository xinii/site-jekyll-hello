---
layout: post
title: "The importance of gnutls for emacs"
subtitle:  "在使用 emacs 时 gnutls 的重要性"
date:   2019-05-04 07:10:00 +0900
background: '/img/posts/fish.png'
---

若当前环境没有 gnutls，
那么在使用 emacs 源代码进行编译安装前的配置
（`./configure`）时可以临时使用 `--with-gnutls=no` 选项，
以使编译安装能够顺利进行。
例如下面的配置：

```
bash configure --prefix=$HOME/.local --with-gif=no --with-gnutls=no
```

上面的例子虽然可以正常安装 emacs 到 `$HOME/.local` 目录，
但缺少了 gnutls 的支持，
emacs 内就只能使用 http 协议而并不能使用 https 协议，
虽然在 emacs 初始化时可以将 melpa 的链接都改为 http 协议解决问题，
但如果后期使用 google-translate 之类的包时还是会遇到问题。


例如即使在 elpa 下将 google-translate 包内的所有 .el 文件中出现的 https 改为 http，
中途还是会出现自动使用某些 https 服务的情况，从而导致翻译异常。
（这里需要注意，修改后需删除所有 .elc 文件以强制重新编译使修改生效）

上面问题的解决办法很简单，只需在当前环境安装 gnutls 即可。

以 OpenSUSE 为例，在终端下运行下面的语句即可完成安装。

```
sudo zypper install gnutls
```

另外，在安装 emacs 时，如果 make 报了以下错误：

```
checking for library containing tputs... no
configure: error: The required function 'tputs' was not found in any library.
The following libraries were tried (in order):
  libtinfo, libncurses, libterminfo, libcurses, libtermcap
Please try installing whichever of these libraries is most appropriate
for your system, together with its header files.
For example, a libncurses-dev(el) or similar package.
make: *** No targets specified and no makefile found.  Stop.
make: *** No rule to make target 'install'.  Stop.
```

那么对于 OpenSUSE 的环境可能需要安装一个 ncurses-devel 的包，
执行下面的语句即可。
```
sudo zypper install ncurses-devel
```
