## 变量

> * Makefile中定义的变量与C/C++中的宏一样，代表一个字符串，在make时在变量使用的地方展开。
> * 变量名区分大小写，推荐使用大小写搭配的方式命名，以避免与系统变量冲突。
> * 变量引用，$(VarName)

## 变量赋值

*不好的赋值*
```ruby
CFLAGS = $(include_dirs) -O
include_dirs = -Ifoo -Ibar
```
前面的变量使用后面的变量，这种递归定义，容易造成make陷入无限的变量展开过程，使得make运行十分缓慢。

为了避免上面的方法，采用“`:=`”操作符。**前面的变量不能使用后面的变量**

```ruby
y := $(x) bar
x := foo
```
y值为“bar”,x值为“foo”.

## 变量值替换

用途：把变量“var”中所有以“a”字串“结尾”的“a”替换成“b”字串。
格式：`$(var:a=b)`或`${var:a=b}`

```ruby
foo := a.o b.o c.o
bar := $(foo:.o=.c)
```
或者
```ruby
foo := a.o b.o c.o
bar := $(foo:%.o=%.c)
```

结果bar值为“a.c b.c c.c”

## 把变量的值当做变量

举个例子就知道了

```ruby
x = $(y)
y = z
z = Hello
a := $($(x))
```
```ruby
$(x) = $(y)
$($(x)) = $($(y)) = $(z) = hello
a = hello
```
## 环境变量

make运行时的系统环境变量可以在make开始运行时被载入到Makefile文件中，但是如果Makefile中已定义了这个变量，或是这个变量由make命令行带入，那么系统的环境变量的值将被覆盖。（如果make指定了“-e”参数，那么，系统环境变量将覆盖Makefile中定义的变量）

因此，如果我们在环境变量中设置了“CFLAGS”环境变量，那么我们就可以在所有的Makefile中使用这个变量了。这对于我们使用统一的编译参数有比较大的好处。如果Makefile中定义了CFLAGS，那么则会使用Makefile中的这个变量，如果没有定义则使用系统环境变量的值，一个共性和个性的统一，很像“全局变量”和“局部变量”的特性。

当make嵌套调用时，上层Makefile中定义的变量会以系统环境变量的方式传递到下层的Makefile 中。当然，默认情况下，只有通过命令行设置的变量会被传递。而定义在文件中的变量，如果要向下层Makefile传递，则需要使用export关键字来声明。

当然，我并不推荐把许多的变量都定义在系统环境中，这样，在我们执行不用的Makefile时，拥有的是同一套系统变量，这可能会带来更多的麻烦。

## 目标变量

变量划分为

> * 全局变量，在整个文件内都可以访问这些变量
> * 局部变量，作用范围只在这条规则以及连带规则之间，不会影响其规则链以外的全局变量。

局部变量的语法：
```ruby
<target ...> : <variable-assignment 赋值表达式>;
```

举例：

```ruby
prog : CFLAGS = -g
prog : prog.o foo.o bar.o
        $(CC) $(CFLAGS) prog.o foo.o bar.o

prog.o : prog.c
        $(CC) $(CFLAGS) prog.c

foo.o : foo.c
        $(CC) $(CFLAGS) foo.c

bar.o : bar.c
        $(CC) $(CFLAGS) bar.c
```
在这个示例中，不管全局的$(CFLAGS)的值是什么，在prog目标，以及其所引发的所有规则中（prog.o foo.o bar.o的规则），$(CFLAGS)的值都是“-g”。

## 模式变量

作用：可以将设定的模式定义在符合这种模式的所有目标上。

语法：
```ruby
%.x : <variable-assignment 赋值表达式>
```

举例：

语法：
```ruby
%.o : Whateve = -O

you : mam.o bab.o
	$(CC) $(Whateve) mam.o bab.o
    
mam.o : hermam.c herbab.c
	$(CC) $(Whateve) hermam.o herbab.o
    
bab.o : hismam.c hisbab.c
	$(CC) $(Whateve) hismam.o hisbab.o
```



