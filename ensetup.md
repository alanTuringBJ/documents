## 环境变量启动脚本ensetup.sh脚本分析

环境变量启动脚本`build\ensetup.sh`是lunch\make\pack的入口，提供一系列命令和全局变量辅助自定义产品加载（lunch）、编译和打包的过程。

### lunch()

作用：提供给用户需要生成的产品选项，根据选项确定变量，并将变量设为全局变量，供给其他程序使用。

主要函数：
* `print_lunch_menu()` 打印LUNCH列表供用户选择；
* `read answer` 等待用户输入，将输入传送给`answer`;
* 根据`answer`的值，确定版本选择`selection`;
* 分离`selection`选项，得到变量`pltform\product\variant`;
* 通过`export`命令将变量设置为全局变量并传递；
* 执行`set_stuff_for_environment`命令设置tina顶层路径；
* 执行`make --no-print-directory -f build/config.mk dumpvar-report_config`

lunch执行过程如下图所示:

<img src="https://github.com/ergasterzhou/img/blob/master/lunch.png" alt="lunch" title="001" width="500" height="300" />

### make()

make()过程相对简单，主要是执行$(get_make_command)命令，即make V=s，然后即可执行顶层目录下的Makefile；

其次，针对时间进行处理，计算并打印编译开始时间和编译结束时间。

### pack()

划分成3个部分：

* 设置一系列打包过程中需要的变量；
* 根据打包参数设置变量；
* 交由$T/scripts/pack_img.sh脚本执行

以选项6.azalea_m2ultra-tina为例，pack中传递的参数包括：
* chip : sun8iw11p1
* platform : tina
* board : azalea-m2ultra
* debug : uart0(默认) card0(-d 选项)
* sigmod : none(默认) secure(-s 选项)
* securemode : none(默认) secure(-v 选项)
* $T : 顶层路径
