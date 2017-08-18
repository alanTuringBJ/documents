## 在tina中添加软件包

在学习过程中，发现openwrt是buildboot的一种进阶形式，而我们公司的tina是buildboot的高阶形式，打个比方说，buildboot是iphone 3，openwrt是iphone6，而Tina就是iphone 7了，这样的比喻好像不恰当。网上对于Tina的介绍和资料较少，所以学习的时候按照openwrt来学习，毕竟两者的差别不大。

在Tina中添加软件包，通俗而言是给系统添加应用。这些应用可以是用户程序，也可以是内核模块程序，可以是网上下载的开源软件，也可以是自己开发的软件。

加入软件包需要在package目录下创建一个目录，目录下包括src目录和Makefile，还有Patch目录。src目录包含软件源代码和用以编译的Makefile；Makefile包含软件包的各种信息和与OpenWrt建立联系的文件，它的写作方式与普通的Makefile存在差异。另外可以创建一个patchs目录保存patch文件，对下载的源代码进行适当修改。

下面介绍这个与众不同的Makefile，以下面这个Makefile模板为例。
```ruby
##############################################
# OpenWrt Makefile for helloworld program
#
#
# Most of the variables used here are defined in
# the include directives below. We just need to
# specify a basic description of the package,
# where to build our program, where to find
# the source files, and where to install the
# compiled program on the router.
#
# Be very careful of spacing in this file.
# Indents should be tabs, not spaces, and
# there should be no trailing whitespace in
# lines that are not commented.
#
##############################################
include $(TOPDIR)/rules.mk
# Name and release number of this package
PKG_NAME:=helloworld
PKG_RELEASE:=1

# This specifies the directory where we're going to build the program.
# The root build directory, $(BUILD_DIR), is by default the build_mipsel
# directory in your OpenWrt SDK directory
PKG_BUILD_DIR := $(BUILD_DIR)/$(PKG_NAME)

include $(INCLUDE_DIR)/package.mk

# Specify package information for this program.
# The variables defined here should be self explanatory.
# If you are running Kamikaze, delete the DESCRIPTION
# variable below and uncomment the Kamikaze define
# directive for the description below
define Package/helloworld
    SECTION:=utils
    CATEGORY:=Utilities
    TITLE:=Helloworld -- prints a snarky message
endef

# Uncomment portion below for Kamikaze and delete DESCRIPTION variable above
define Package/helloworld/description
If you can't figure out what this program does, you're probably
brain-dead and need immediate medical attention.
endef

# Specify what needs to be done to prepare for building the package.
# In our case, we need to copy the source files to the build directory.
# This is NOT the default. The default uses the PKG_SOURCE_URL and the
# PKG_SOURCE which is not defined here to download the source from the web.
# In order to just build a simple program that we have just written, it is
# much easier to do it this way.
define Build/Prepare
    mkdir -p $(PKG_BUILD_DIR)
    $(CP) ./src/* $(PKG_BUILD_DIR)/
endef

# We do not need to define Build/Configure or Build/Compile directives
# The defaults are appropriate for compiling a simple program such as this one

# Specify where and how to install the program. Since we only have one file,
# the helloworld executable, install it by copying it to the /bin directory on
# the router. The $(1) variable represents the root directory on the router running
# OpenWrt. The $(INSTALL_DIR) variable contains a command to prepare the install
# directory if it does not already exist. Likewise $(INSTALL_BIN) contains the
# command to copy the binary file from its current location (in our case the build
# directory) to the install directory.
define Package/helloworld/install
$(INSTALL_DIR) $(1)/bin
$(INSTALL_BIN) $(PKG_BUILD_DIR)/helloworld $(1)/bin/
endef

# This line executes the necessary commands to compile our program.
# The above define directives specify all the information needed, but this
# line calls BuildPackage which in turn actually uses this information to
# build a package.
$(eval $(call BuildPackage,helloworld))
```
### 引入文件

OpenWrt使用三个makefile的子文件,包括rules.mk,kernel.mk和packafe.mk。这些makefile子文件确立软件包加入OpenWrt的方式和方法。

```ruby
# 不可缺少
include $(TOPDIR)/rules.mk			
# 对于软件包为内核模块时不可缺少
include $(INCLUDE_DIR)/kernel.mk
# 一般在软件包的基本信息完成后再引入
include $(INCLUDE_DIR)/package.mk
```

### 编写软件包基本信息

* PKG_NAME表示软件包名称，将在menuconfig和ipkg可以看到。
* PKG_VERSION   表示软件版本号。
PKG_RELEASE表示Makefile的版本号
PKG_SOURCE表示源代码的文件名。
PKG_SOURCE_URL 表示源代码的下载网站位置。
PKG_MD5SUM表示源代码文件的效验码。用于核对软件包是否正确下载。
PKG_CAT表示源代码文件的解压方法。包括zcat, bzcat, unzip等。
* PKG_BUILD_DIR表示软件包编译目录。它的父目录为$(BUILD_DIR)。如果不指定，默认为$(BUILD_DIR)/$( PKG_NAME)$( PKG_VERSION)。

### 用户程序包定义

用户程序的编译包以Package/开头，然后接软件名

* Package/$(PKG_NAME)
```
包含以下选项（SECTION保留，DESCRIPTION弃用）：
CATEGORY表示分类，在menuconfig的菜单下将可以找到；
TITLE用于软件包的简短描述；
URL表示软件包的下载位置；
MAINTAINER表示维护者，选项；
DEPENDS表示与其他软件的依赖；
```

* Package/$(PKG_NAME)/description软件包的详细描述
* Build/Prepare编译准备方法，将源文件放在目标生成目录下
```ruby
define Build/Prepare  
        mkdir -p $(PKG_BUILD_DIR)  
        $(CP) ./src/* $(PKG_BUILD_DIR)/  
endef  
```
* Package/$(PKG_NAME)/install
软件包的安装方法，包括一系列拷贝编译好的文件到指定位置。$(1)表示嵌入式系统镜像文件系统目录，例如下面例子中的helloworld将安装在板子系统的/bin文件夹下
```ruby
define Package/$(PKG_NAME)/install  
        $(INSTALL_DIR) $(1)/usr/bin  
        $(INSTALL_BIN) $(PKG_BUILD_DIR)/ $(PKG_NAME) $(1)/usr/bin/  
endef
```
NSTALL_DIR、INSTALL_BIN在$(TOPDIR)/rules.mk文件定义，所以本Makefile必须引入$(TOPDIR)/rules.mk文件。

INSTALL_DIR :=install -d -m0755    

创建所属用户可读写即执行，其他用户可读且执行（不能写）的目录。

INSTALL_BIN:=install -m0755 

编译好的文件到镜像文件的文件目录。

* $(eval $(call BuildPackage,$(PKG_NAME))) 对于用户程序软件包
* $(eval $(call KernelPackage,$(PKG_NAME))) 对于内核模块软件包

特别注意：BuildPackage,$(PKG_NAME)之间只有逗号，不能有空格。




### 下面，以helloworld为例在系统中创建helloworld程序.

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
Is your src/ directory actually clean?
I suspect it contains an "amldmonitor" executable already which was built for your host.
Make sure your src/ directory does not contain any junk like *.o files or final executables.
```

在写底层源文件和Makefile时编译源文件生成可执行文件helloworld，进入src文件夹make clean后编译通过。

解决以上问题后，最终的Makefile，这个只针对helloworld，以后其他程序可在此基础上做出修改。
```ruby
include $(TOPDIR)/rules.mk

PKG_NAME:=helloworld
PKG_VERSION:=0.0.1
#PKG_RELEASE:=1.0

PKG_BUILD_DIR:=$(COMPILE_DIR)/$(PKG_NAME)

include $(BUILD_DIR)/package.mk

define Package/helloworld
	CATEGORY:=Allwinner
	TITLE:=Helloworld
endef

define Package/helloworld/description
	This is my helloworld:
	I look foward a happy beggin.
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef

define Package/helloworld/install
	$(INSTALL_DIR) $(1)/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/$(PKG_NAME) $(1)/bin/
endef

$(eval $(call BuildPackage,helloworld))
```

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
参考链接：
[解决adb shell问题：error: insufficient permissions for device](http://blog.csdn.net/xiaxiangnanxp1989/article/details/8605611)
[adb 权限问题 （insufficient permissions for device）](http://blog.csdn.net/hnmsky/article/details/7417151)

### 安装helloworld

opkg install helloworld0.0.1.ipk

知道opkg没有安装，然后被告知make menuconfig的时候编译进系统就好了，以后在某个程序没有时，先去menuconfig里查找，用`\`。

参考链接：
[Openwrt 学习记录：openWRT添加用户模块-helloword](http://blog.csdn.net/fenggui/article/details/46669983)















