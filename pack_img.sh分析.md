## 打包脚本pack_img.sh分析

镜像文件经tina/scripts/pack_img.sh打包后生成用于下载到开发板的固件。

pack_img.sh的流程如下：

* do_prepare() 主要工作是将用于打包的工具、配置文件、root资源和文件拷贝到$(ROOT_DIR)/img目录下；
* do_init_to_dts() 通过lichee下的文件生成sunxi.dbt
* do_common()  分部分构建系统的各个部分（boot0/uboot）
* do_pack_tina()  建立文件软链接
* do_finish()  建立分区

文件结构如下图所示：

<img src="https://github.com/ergasterzhou/img/blob/master/pack_img.sh.png" alt="pack_img" title="001" width="1000" height="600" />

### awk -F : '${print $1}'

```ruby
for file in ${boot_resource_list[@]} ; do
                cp -rf ${PACK_TOPDIR}/target/allwinner/`echo $file | awk -F: '{print $1}'` \
                        ${ROOT_DIR}/`echo $file | awk -F: '{print $2}'` 2>/dev/null

```

```ruby
boot_resource_list=(
generic/boot-resource/boot-resource:image/
generic/boot-resource/boot-resource.ini:image/
${PACK_BOARD_PLATFORM}-common/boot-resource:image/
${PACK_BOARD_PLATFORM}-common/boot-resource.ini:image/
${PACK_BOARD}/configs/*.bmp:image/boot-resource/
)

```
awk -F中-F指定分隔符，脚本中分隔符为：

{print $1}输出第一个字段，{print $2}输出第二个字段

所以例句的意思是将${PACK_TOPDIR}/target/allwinner/下的
* generic/boot-resource/boot-resource
* generic/boot-resource/boot-resource.ini
* ${PACK_BOARD_PLATFORM}-common/boot-resource
* ${PACK_BOARD_PLATFORM}-common/boot-resource.ini

拷贝到$(ROOT_DIR)/img目录下

### unix2dos
```ruby
busybox unix2dos sys_config.fex
busybox unix2dos board_config.fex
script  sys_config.fex > /dev/null
cp -f   sys_config.bin config.fex
script  board_config.fex > /dev/null
cp -f board_config.bin board.fex

```
fex文件解释说明SoC工作的不同方面，包括GPIO设置、DRAM启动和显示等相关参数。具体解释可参考：[Fix Guide](http://linux-sunxi.org/Fex_Guide)

fex文件编译器需要fex文件转换成dos格式，所以在scripts编译前，需要通过uinx2dos进行转换。

script命令应该是公司设计命令，没有找到相关介绍和源码，所以只能猜想其左右是将fex文件编译生成bin文件。

### >> 与 >的区别
L418：
```ruby
 echo "imagename = $IMG_NAME" >> ${ROOT_DIR}/image/image.cfg

```
```ruby
if [ ! -f board_config.fex ]; then
   echo "[empty]" > board_config.fex
fi

```

>> : 如果文件不存在，将创建新的文件
     如果文件存在，将数据添加到文件后面
>  : 如果文件不存在，将创建新的文件
     如果文件存在，将文件清空，然后填入数据
     

### update_mbr命令

```ruby
update_mbr sys_partition_nor.bin 1 sunxi_mbr_nor.fex
```
update_mbr是公司定义命令，可以参考[update_mbr.c](https://github.com/friendlyarm/a64_lichee/blob/master/brandy/pack_tools/create_mbr/update_mbr.c)

语法：

update_mbr source mbr_count

source:源文件

mbr_count:系统分区数

执行：
* 打印分区文件路径、mbr_name文件路径以及下载路径
* 调用script_parser_init初始化脚本
* 调用script_parser_fetch函数取得任意数值项，包括了
```ruby
用户类型、读写属性、写保护属性、签名是否需要校验等等
```
* 调用update_for_part_info更新CRC校验码

以上是针对pack_img.sh脚本的分析，通过学习，对脚本的常用命令、复杂操作都有了进一步了解，同时也存在很多问题，比如对boot的不了解，影响对do_finish的判断，其次公司内部命名有好几个，例如：script unic2dos
updare_mbr和drangen 除update_mbr找到源码，对照观看外，网络上缺少对这些命令的介绍。





     
