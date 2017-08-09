## 通配符  
make支持三个通配符`*`,`?`,`~`.  
波浪号`~`表示$HOME目录。  
例如
“~/test”，这就表示当前用户的$HOME目录下的test目录。  
“~hchen/test”则表示用户hchen的宿主目录下的test目录。  
 
星号`*`代替了你一系列的文件。  
例如：
“*.c”表示所有后缀为c的文件。  

## 文件搜索  
在一些大的工程中，有大量的源文件，我们通常的做法是把这许多的源文件分类，并存放在不同的目录中。所以，当make需要去找寻文件的依赖关系时，你可以在文件前加上路径，但最好的方法是把一个路径告诉make，让make自动去找。**目录永远是最优先搜索的地方**  
```ruby
VPATH = src1:src2
```
VPATH指明两个目录"src1"和"src2"，目录之间用冒号`“”`分隔。  
相比VPATH而言，vpath搜索目录更加灵活，可以搜索不同目录下的不同类型的文件。使用方法如下：  
	vpath <pattern> <directories>
    
为符合模式<pattern>的文件指定搜索目录<directories>。  
例如：搜索上级目录headers文件下的头文件。  
```ruby
vpath %.h ../headers
```

## 伪目标  
伪目标一般没有依赖的文件。但是，我们也可以为伪目标指定所依赖的文件。伪目标同样可以作为“默认目标”，只要将其放在第一个。一个示例就是，如果你的Makefile需要一口气生成若干个可执行文件，但你只想简单地敲一个make完事，并且，所有的目标文件都写在一个Makefile中，那么你可以使用“伪目标”这个特性：  
```ruby
all : prog1 prog2 prog3
.PHONY : all

prog1 : prog1.o utils.o
	cc -o prog1 prog1.o utils.o

prog2 : prog2.o
	cc -o prog2 prog2.o

prog3 : prog3.o sort.o utils.o
	cc -o prog3 prog3.o sort.o utils.o
```
> * makefile中的第一个目标会被作为默认目标，默认目标总是被执行。
> * 伪目标是标签，会执行，但不会生成all目标文件。
> * 伪目标的依赖会被复议执行，产生三个目标文件。（目标也会成为依赖）

## 静态模式

目的：更简明地定义多目标的规则。  
模式：  
```ruby
<targets ...>: <target-pattern>: <prereq-patterns ...>
	<commands>
	...
```

> * targets定义了一系列的目标文件，是一个目标集合。
> * target-pattern是指明了targets的模式，也就是的目标集模式。
> * prereq-patterns指明依赖的模式。

```ruby
objects = foo.o bar.o

all: $(objects)

$(objects): %.o: %.c
	$(CC) -c $(CFLAGS) $< -o $@
```

`$(objects)`表示目标集合（foo.o bar.o)  
`%`表示去目标集合模式（foo bar)  
`$<`表示依赖集（foo.c bar.c)  
`$@`表示目标集（foo.o bar.o)  








