## 使用条件判断

使用条件判断，可以让make根据运行时的不同情况选择不同的执行分支。条件表达式可以是比较变量的值，或是比较变量和常量的值。

```ruby
libs_for_gcc = -lgnu
normal_libs =

foo: $(objects)
ifeq ($(CC),gcc)
        $(CC) -o foo $(objects) $(libs_for_gcc)
else
        $(CC) -o foo $(objects) $(normal_libs)
endif
```
在上面示例的这个规则中，目标`“foo”`可以根据变量`“$(CC)”`值来选取不同的函数库来编译程序。

## 语法

条件表达式的语法为：
```ruby
<conditional-directive>
<text-if-true>
endif
```
或者
```ruby
<conditional-directive>
<text-if-true>
else
<text-if-false>
endif
```

