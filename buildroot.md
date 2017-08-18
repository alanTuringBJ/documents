## Buildroot

Buildroot是通过交叉编译器简化、自动化嵌入式设备搭建完整Linux系统过程的工具。它能为你的目标设备生成交叉编译工具链、根文件系统、内核镜像以及bootloader.

openwrite Buildroot 是在Buildroot的基础上做出大量修改，它包含一系列的Makefiles和patches，方便在嵌入式系统上生成交叉编译工具链和root文件系统。交叉编译工具链使用一个轻巧的C标准库uClibc。

* 非常方便地移植软件；
* 使用kconfig（linux内核menuconfig）配置各种特性；
* 提供继承的交叉编译工具链（gcc、ld等）；
* 提供抽象的自动化工具（automake、autoconf、Cmake、scons等）；
* 标准化下载、打补丁、配置、编译和打包的工作流程；
* 提供大量常用的包修复补丁；

* Openwrt Buildroot包含的工具链是交叉编译工具链，运行在host机器上,但生成的代码是给target机器上运行。
* OpenWrt Buildroot 通过 Makefiles 把相关流程都自动化了, 并且通过一系列的 patch, 使得每个版本的gcc and binutils 都能正常运行在各种指令集架构的嵌入式系统之上。

OpenWrt的makefile定义了软件包的一些元信息，在哪里下载、如何编译、编译后二进制文件安装到哪里等等。

