## 概述  
**makefile关系到了整个工程的编译规则。**  
make命令执行时，需要一个 makefile 文件，以告诉make命令如何去编译和链接程序。 
**自动化编译，提高软件开发的效率。**  

## 编译过程  
**编译（complile）:源文件生成中间代码文件（.o）**  
编译时，编译器需要的是语法的正确，函数与变量的声明的正确，只要所有的语法正确，编译器就可以编译出中间目标文件。  
**链接（link）：把中间代码文件合成可执行文件**  
链接程序时，链接器会在所有的Object File中找寻函数的实现，如果找不到，那到就会报链接错误码（Linker Error）。  

## 规则  
```ruby
target ... : prerequisites ...
	command
	...
	...
```  

target可以是一个object file(目标文件)，也可以是一个执行文件，还可以是一个标签（label）。  
prerequisites就是，要生成那个target所需要的文件或是目标。  
command也就是make需要执行的命令（**任意**的shell命令）。  

**makefile包括：显式规则、隐式规则、变量定义、文件指示和注释。**

1. 显式规则。显式规则说明了，如何生成一个或多个目标文件。这是由Makefile的书写者明显指出，要生成的文件，文件的依赖文件，生成的命令，**在Makefile中的命令，必须要以[Tab]键开始**。

2. 隐晦规则。由于我们的make有自动推导的功能，所以隐晦的规则可以让我们比较简略地书写Makefile，这是由make所支持的。  
3. 变量的定义。在Makefile中我们要定义一系列的变量，变量一般都是字符串，这个有点像你C语言中的宏，当Makefile被执行时，其中的变量都会被扩展到相应的引用位置上。  
4. 文件指示。其包括了三个部分
* 一个Makefile中引用另一个Makefile，就像C语言中的include一样；
* 另一个是指根据某些情况指定Makefile中的有效部分，就像C语言中的预编译#if一样；
* 还有就是定义一个多行的命令。
5. 注释。Makefile中只有行注释，和UNIX的Shell脚本一样，其注释是用“#”字符。  

## 实例  
```ruby
edit : main.o kbd.o command.o display.o \
		insert.o search.o files.o utils.o      
        /*注释:如果后面这些.o文件比edit可执行文件新,那么才会去执行下面这句命令*/
	cc -o edit main.o kbd.o command.o display.o \
		insert.o search.o files.o utils.o

main.o : main.c defs.h
	cc -c main.c
kbd.o : kbd.c defs.h command.h
	cc -c kbd.c
command.o : command.c defs.h command.h
	cc -c command.c
display.o : display.c defs.h buffer.h
	cc -c display.c
insert.o : insert.c defs.h buffer.h
	cc -c insert.c
search.o : search.c defs.h buffer.h
	cc -c search.c
files.o : files.c defs.h buffer.h command.h
	cc -c files.c
utils.o : utils.c defs.h
	cc -c utils.c
clean :
	rm edit main.o kbd.o command.o display.o \
		insert.o search.o files.o utils.o
```

在这个makefile中，
目标文件（target）包含：执行文件edit和中间目标文件（*.o），
依赖文件（prerequisites）就是冒号后面的 .c 文件和 .h文件。每一个 .o 文件都有一组依赖文件，而这些 .o 文件又是执行文件 edit 的依赖文件。  

clean不是一个文件，它只不过是一个动作名字，有点像c语言中的lable一样，其冒号后什么也没有，那么，make就不会自动去找它的依赖性，也就不会自动执行其后所定义的命令。要执行其后的命令（不仅用于clean，其他lable同样适用），就要在make命令后明显得指出这个lable的名字。这样的方法非常有用，我们可以在一个makefile中定义不用的编译或是和编译无关的命令，比如程序的打包，程序的备份，等等。  

## how to work
**makefile具有文件依赖性，会向下层级搜索依赖关系，直到生成目标文件或者出现错误。**  

## 使用变量  
**用变量的形式描述一个或一组文件或路径，以便使makefile降低错误发生和后期维护**  
*申明过程 :*  
```ruby
objects = main.o kbd.o command.o display.o \
		insert.o search.o files.o utils.o
```

*mafile中以`$(objects)`的方式使用变量 :*  
```ruby
objects = main.o kbd.o command.o display.o \
		insert.o search.o files.o utils.o

edit : $(objects)
	cc -o edit $(objects)
main.o : main.c defs.h
	cc -c main.c
kbd.o : kbd.c defs.h command.h
	cc -c kbd.c
command.o : command.c defs.h command.h
	cc -c command.c
display.o : display.c defs.h buffer.h
	cc -c display.c
insert.o : insert.c defs.h buffer.h
	cc -c insert.c
search.o : search.c defs.h buffer.h
	cc -c search.c
files.o : files.c defs.h buffer.h command.h
	cc -c files.c
utils.o : utils.c defs.h
	cc -c utils.c
clean :
	rm edit $(objects)
```

## make自动推导  
**GNU的make很强大，它可以自动推导文件以及文件依赖关系后面的命令**  
只要make看到一个[.o]文件，它就会自动的把[.c]文件加在依赖关系中，如果make找到一个whatever.o，那么 whatever.c，就会是whatever.o的依赖文件。并且 cc -c whatever.c 也会被推导出来，于是，我们可以进一步简化makefile。
```ruby
objects = main.o kbd.o command.o display.o \
		insert.o search.o files.o utils.o
 cc = gcc

edit : $(objects)
	cc -o edit $(objects)

main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h

.PHONY : clean
clean :
	rm edit $(objects)
```
**clean从来都是放在文件的最后。**  
`.PHONY`是显示指明clean是一个“伪目标”。而在rm命令前面加了一个小减号`-`的意思就是，也许某些文件出现问题，但不要管，继续做后面的事。  

[下一章：Makefile书写规则]()





