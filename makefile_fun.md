## 使用函数

在Makefile中可以使用函数来处理变量，从而让我们的命令或是规则更为的灵活和具有智能。

函数调用
```ruby
$(<function> <arguement>,... ...,<arguement>)
```

`<function>`是函数名，`<arguments>`为函数的参数，函数名和参数之间以“空格”分隔，参数间以逗号`,`分隔，函数调用以`$`开头，以圆括号或花括号把函数名和参数括起。

## 字符串处理函数

|   函数名   |     函数内容      |格式| 
| :-----:   |    :-----:       | :---|
|   subst   |   字符串替换函数   | `$(subst <from>,<to>,<text>)` |
|  patsubst | 模式字符串替代函数  |`$(patsubst <pattern>,<replacement>,<text>) `
|  strip    | 去字符串首尾空格函数|`$(strip <string>)`|
|findstring|查找字符串函数|`$(findstring <find>,<in>)`
|filter|过滤函数|`$(filter <pattern...>,<text>)`
|filter-out|反过滤函数|`$(filter-out <pattern...>,<text>)`
|sort|排序函数（按单词首字母升序）|`$(sort <list>)`
|word|取单词函数|`$(word <n>,<text>)`
|word-list|取单词函数|`$(wordlist <ss>,<e>,<text>)  `
|words|单词个数统计函数|`$(words <text>)`
|firstword|单词首个函数|`$(firstword <text>)`

以上，是所有的字符串操作函数，如果搭配混合使用，可以完成比较复杂的功能。这里，举一个现实中应用的例子。我们知道，make使用“VPATH”变量来指定“依赖文件”的搜索路径。于是，我们可以利用这个搜索路径来指定编译器对头文件的搜索路径参数CFLAGS，如：

```ruby
override CFLAGS += $(patsubst %,-I%,$(subst :, ,$(VPATH)))
```
如果我们的`“$(VPATH)”`值是`“src:../headers”`，那么`“$(patsubst %,-I%,$(subst :, ,$(VPATH)))”`将返回`“-Isrc -I../headers”`，这正是cc或gcc搜索头文件路径的参数。

## 文件名操作函数
|   函数名   |     函数内容      |格式| 
| :-----:   |    :-----:       | :---|
|dir|取目录函数(`/`之前部分)|`$(dir <names...>) `
|notdir|取文件函数(`/`之后部分)|`$(notdir <names...>) `
|suffix|取后缀函数|`$(suffix <names...>) `
|basename|取前缀函数|`$(basename <names...>)`
|addsuffix|加后缀函数|`$(addsuffix <suffix>,<names...>) `
|addprefix|加前缀函数|`$(addprefix <prefix>,<names...>) `
|join|连接函数|`$(join <list1>,<list2>)`

## foreach函数

循环函数

语法：把参数<list>中的单词逐一取出放到参数所指定的变量中，然后再执行< text>所包含的表达式。每一次<text>会返回一个字符串，循环过程中，<text>的所返回的每个字符串会以空格分隔，最后当整个循环结束时，<text>所返回的每个字符串所组成的整个字符串（以空格分隔）将会是foreach函数的返回值。
```ruby
$(foreach n,<list>,<text>)
```
```ruby
names := a b c d

files := $(foreach n,$(names),$(n).o)
```
上面的例子中，`$(name)`中的单词会被挨个取出，并存到变量`“n”`中，`“$(n).o”`每次根据`“$(n)”`计算出一个值，这些值以空格分隔，最后作为foreach函数的返回，所以，$(files)的值是`“a.o b.o c.o d.o”`。

## call函数

参数传递函数

语法：当make执行这个函数时，<expression>参数中的变量，如$(1)，$(2)，$(3)等，会被参数< parm1>，<parm2>，<parm3>依次取代。而<expression>的返回值就是 call函数的返回值。例如：
```ruby
reverse =  $(1) $(2)

foo = $(call reverse,a,b)
```
那么，foo的值就是`“a b”`。当然，参数的次序是可以自定义的，不一定是顺序的。

## origin函数

告知变量从何而来

语法：
```ruby
$(origin <variable>;)
```
注意，`<variable>`是变量的名字，不应该是引用。

|   返回值   |     返回值含义      |
| :-----:   |    :-----:       |
| undefines | 未曾定义
| default   | 默认定义
|environment| 环境变量
|file|在Makefile中定义
|command line|这个变量被命令行定义
|override|变量被override指示符重新定义
|automatic|命令运行中的自动化变量

这些信息对于我们编写Makefile是非常有用的，例如，假设我们有一个Makefile其包了一个定义文件Make.def，在 Make.def中定义了一个变量“bletch”，而我们的环境中也有一个环境变量“bletch”，此时，我们想判断一下，如果变量来源于环境，那么我们就把之重定义了，如果来源于Make.def或是命令行等非环境的，那么我们就不重新定义它。于是，在我们的Makefile中，我们可以这样写：

```ruby
ifdef bletch

ifeq "$(origin bletch)" "environment"
 
bletch = barf, gag, etc.

endif

endif

```
用override是可以达到这样的效果，可是override过于粗暴，它同时会把从命令行定义的变量也覆盖了，而我们只想重新定义环境传来的，而不想重新定义命令行传来的。

## 控制make的函数
|   函数名   |     函数内容      |格式| 
| :-----:   |    :-----:       | :---|
| error |产生一个致命错误，警告<text>信息|`$(error <text ...>;)`
|warning|产生一个提示，输出警告信息，不会终止make|`$(warning <text ...>;)`

```ruby
ifdef ERROR_001

$(error error is $(ERROR_001))

endif
```


