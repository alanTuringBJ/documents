## 创建软件包

这里，以helloworld为例在系统中创建helloworld程序.

步骤：

* 创建helloworld项目，包括源程序helloworld.c和相应的Makefile;
* 创建上层Makefile，描述helloworld信息；
* 修改编译过程中出现的错误；
* helloworld在开发板运行。

## 创建helloworld项目

在Package目录下创建src目录，src目录下创建源程序helloworld.c和相应的Makefile。

源程序helloworld.c
```ruby
#include <stdio.h>
int main()
{
    printf("This is my hello word!\n");

    return 0;
}
```
Makefile
```ruby
helloworld : helloworld.o
    $(CC) $(LDFLAGS) helloworld.o -o helloworld

helloworld.o : helloworld.c
    $(CC) $(CFLAGS) -c helloworld.c

clean :
    rm *.o helloworld
```

## 创建上层Makefile
上层Makefile，描述helloworld信息，信息包括：软件包配置、编译规则和打包方式。

这里是我照框架修改的Makefile,力求用最简单的方式来写，缺少哪里再补充，这样能更加了解这个Makefile。

```ruby
include $(TOPDIR)/rules.mk

PKG_NAME:=helloworld

PKG_BUILD_DIR:=$(COMPILE_DIR)/$(PKG_NAME)

include $(BUILD_DIR)/package.mk

define Package/helloworld
	CATEGORY:=Allwinner
	TITLE:=Helloworld
endef

define Package/helloworld/install
	$(INSTALL_DIR) $(1)/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/$(PKG_NAME) $(1)/bin/
endef

$(eval $(call BuildPackage,helloworld))
```
## 修改编译过程中出现的问题

* 缺少版本号（VERSION）
`PKG_VERSION`描述软件包的版本，最终生成的包为`$(PKG_NAME)$(PKG_VERSION).ipk`文件。
例如helloworld0.0.1.ipk

* No targets specified and no makefile found.

```ruby
make[4]: Entering directory `/home/alanzhou/workspace/tina/out/azalea-m2ultra/compile_dir/target/helloworld-0.0.1'
make[4]: *** No targets specified and no makefile found.  Stop.

```

在编译过程中产生的target/helloworld-0.0.1目录下没有源文件和Makefile,联想到package文件夹下的helloworld.c和相应的Makefile，以及Build/Prepare所做的工作就是创建tina/out/azalea-m2ultra/compile_dir/target/helloworld-0.0.1这个文件夹，还有将src下的文件拷贝过去。

所以，这里加入
```ruby
define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef
```
* Package helloworld is missing dependencies 
```ruby
Package helloworld is missing dependencies for the following libraries:
libc.so.6
make[3]: *** [/home/alanzhou/workspace/tina/out/sitar-perf1/packages/base/helloworld_0.0.1_sunxi.ipk] Error 
```
最初的想法是去找相关的依赖库，然后
```ruby
find -name libc.so.6
```
发现不同编译器的很多libc.so.6库文件

于是找其他答案，发现[OpenWRT - package missing dependencies when recompiling](https://stackoverflow.com/questions/20190030/openwrt-package-missing-dependencies-when-recompiling)
提到
```ruby
Is your src/ directory actually clean? I suspect it contains an "amldmonitor" executable already which was built for your host. Make sure your src/ directory does not contain any junk like *.o files or final executables.
```

在写底层源文件和Makefile时编译源文件生成可执行文件helloworld，进入src文件夹make clean后编译通过。

## helloworld在开发板运行

过程如下：

* 通过minicom与开发板链接----------------->minicom设置
* 传输helloworld0.0.1.ipk软件包到开发板--->adb设置
* 安装helloworld0.0.1.ipk软件包---------->opkg配置安装
* 运行helloworld程序

### minicom设置

* `lsusb`查找使用的USB的ID
```ruby
zhouyj@zhouyj-ubuntu:~$ lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 004: ID 09da:c10a A4 Tech Co., Ltd 
Bus 001 Device 003: ID 1c4f:0026 SiGma Micro Keyboard
Bus 001 Device 005: ID 067b:2303 Prolific Technology, Inc. PL2303 Serial Port
Bus 001 Device 002: ID 1a40:0201 Terminus Technology Inc. FE 2.1 7-port Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

* 安装minicom
```ruby
sudo apt-get install minicom
```

* 启动minicom
```ruby
sudo minicom
```
<img src="https://github.com/ergasterzhou/img/blob/master/001.png" alt="001" title="001" width="500" height="100" />

* `[Ctrl]A Z`进入Minicom Command Summary
<img src="https://github.com/ergasterzhou/img/blob/master/002.png" alt="002" title="002" width="500" height="350" />

* `O`进入串口设置
<img src="https://github.com/ergasterzhou/img/blob/master/003.png" alt="003" title="003" width="500" height="300" />

* 设置串口
<img src="https://github.com/ergasterzhou/img/blob/master/004.png" alt="004" title="004" width="500" height="300" />

* 保存设置
<img src="https://github.com/ergasterzhou/img/blob/master/005.png" alt="005" title="005" width="500" height="300" />

* `[Ctrl]A X`退出minicom

参考链接：[minicom的配置](http://blog.csdn.net/huang_jinjin/article/details/7475188)

### adb的设置
当遇到需要安装新的文件到开发板系统的时候，将重新编译的系统文件烧写到开发板，这种做法时间长，操作复杂而且容易出错。
那么有没有简单方式安装新的文件到开发板上呢？通过adb（Android Debug Bridge）传输命令行工具。
```ruby
adb push <local> <remote>
```
例如：
```ruby
adb push /home/zhouyj/compile/tina/out/sitar-perf1/packages/base/helloworld_0.0.1_sunxi.ipk /ipk

```
在安装完adb后连接设备，提示权限不正确，出现错误信息：
```ruby
shily@hh-desktop:~$adb shell
error: insufficient permissions for device
shily@hh-desktop:~$ adb devices
List of devices attached 
????????????    no permissions
```
解决方式：

* 用`lsusb`在终端查看USB的ID号;
* 进入`/etc/udev/rules.d/`目录，修改`70-Andriod.rules`文件，如果没有就创建，在在这个文件中写上：
```ruby
SUBSYSTEM=="usb", ATTRS{idVendor}=="youridVendor", ATTRS{idProduct}=="youridProduct",MODE="0666"
```
* 修改`70-Andriod.rules`文件权限
```ruby
sudo chmod a+x 70-android.rules
```

* 采用root权限启动adb server
```ruby
sudo adb kill-server ; adb start-server
```

















