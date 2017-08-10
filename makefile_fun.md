## 使用函数

在Makefile中可以使用函数来处理变量，从而让我们的命令或是规则更为的灵活和具有智能。

函数调用
```ruby
$(<function> <arguement>,... ...,<arguement>)
```

<function>就是函数名，<arguments>为函数的参数，函数名和参数之间以“空格”分隔，参数间以逗号“,”分隔，函数调用以“$”开头，以圆括号或花括号把函数名和参数括起。

## 字符串处理函数

字符串处理函数包括：
> * subst			字符串替换函数subst
> * patsubst		模式字符串替换函数patsu
> * strip			去空格函数
> * findstring		查找字符串函数
> * filter			过滤函数
> * filter-out		反过滤函数

|  函数名   |     函数内容      |
| ：----：  |    ：-----：     |
|  subst   |   字符串替换函数   |

