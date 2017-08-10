## make运行

## 指定目标

一般来说，make的最终目标是makefile中的第一个目标，而其它目标一般是由这个目标连带出来的。这是make的默认行为。当然，一般来说，你的makefile中的第一个目标是由许多个目标组成，你可以指示make，让其完成你所指定的目标。要达到这一目的很简单，需在make命令后直接跟目标的名字就可以完成（如前面提到的“make clean”形式）。

```ruby
.PHONY: all
all: prog1 prog2 prog3 prog4
```
从这个例子中，我们可以看到，这个makefile中有四个需要编译的程序——“prog1”， “prog2”， “prog3”和 “prog4”，我们可以使用“make all”命令来编译所有的目标（如果把all置成第一个目标，那么只需执行“make”），我们也可以使用 “make prog2”来单独编译目标“prog2”。

即然make可以指定所有makefile中的目标，那么也包括“伪目标”，于是我们可以根据这种性质来让我们的makefile根据指定的不同的目标来完成不同的事。在Unix世界中，软件发布时，特别是GNU这种开源软件的发布时，其makefile都包含了编译、安装、打包等功能。我们可以参照这种规则来书写我们的makefile中的目标。

makefile中目标规则

|  目标  | 功能 |
| :------  | :------ |
| all | 所有目标的目标，作用是编译所有目标|
|clean|删除所有被make创建的文件
|install|安装已经编译好的程序，把目标执行文件拷贝到指定的目标
|print|列出改变过的文件
|tar|把源程序打包备份,tar文件
|dlst|创建压缩文件，一般把tar文件压成Z文件，或者gz文件
|TAGS|更新所有目标，以便完整地重新编译

## make的参数

参考
[make运行](http://wiki.ubuntu.org.cn/%E8%B7%9F%E6%88%91%E4%B8%80%E8%B5%B7%E5%86%99Makefile:make%E8%BF%90%E8%A1%8C)
