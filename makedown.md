##Let's begin
###语法
####标题
1.可以用对称的#包括文本
2.可以只在左侧使用#
3.左侧不能有任何空白
4.#的个数决定几级标题
5.个数越多，等级越低
6.超过6，等级一致

####引用
1.在引用内容前加入符号>,该段内容即为引用
>天才在于百分之一的灵感和百分之九十九的汗水
>							————忘记谁说的了
>							
>未来是你的
>————波波  

2.不支持句前空格，被我发现了，我想在这里用个表情
3.引用嵌套就不写了，我是不会嵌套引用的

####列表
#####无序列表
>* 可以使用 '*' 作为标记
>+ 可以使用 ‘+’ 作为标记
>- 或者 ‘-’

1.注意符号和文字之间有空格
2.有序列表就不说了，就是我现在写的，内啥内涵
3.来个嵌套玩玩

>1.我和尼玛同时掉水里你救谁？
>- 我
>- 我
>- 还是我    
>
>2.我爱你，但是____
>i 我没钱
>ii 我们不是同一个物种
>

####代码
#####代码块
	<html>
		<title>hello world!</title>
	</html>
有缩进就OK，不要被html影响
行内插入代码，使用``，(键盘Tab键上，数字1左侧)

另一种方法，` ``` `独占一行
```
code here
```
代码高亮
```js
zhouyongjie.owns.phone('balala',function()){
	console.log('window loader');
}
```

例如`print（“hello world！”）`

####超链接
格式为`[link text](URL 'title text')`,
例如：[爱情电影网](http://www.aqdyy.com/)
主要URL上的引号是英文，否则出错

####图像
格式为`<img src="URL",alt="",title="",width="宽"，height="高" />`
例如：
<img src="http://upload-images.jianshu.io/upload_images/3678149-649933f814ff231f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240",alt="english only" title="Bags" width="500" height="500" />

####强调
1.斜体，使用`* *`或`_ _`
*这是一个重点项目*
2.粗体，使用`** **``__ __`
__这是国际级比赛__

###扩展语法
删除线
`今天风和日丽，~~王丽~~万里无云。`

####表格
```
|   name    |  age  |
| --------  | ----- |
| allwinner |  10   |
|    MTK    |  30   |
```
|   name    |  age  |
| --------  | ----- |
| allwinner |  10   |
|    MTK    |  30   |

用冒号改善对齐方式
* `:---`    代表左对齐
* `:--：`   代表居中对齐
* `---：`   代表右对齐
```
|   name    |  age  | money |
| :------:  |:-----:| :---: |
| allwinner |  10   |   1   |
|    MTK    |  30   |   10  |
```
|   name    |  age  | money |
| :------:  |:-----:| :---: |
| allwinner |  10   |   1   |
|    MTK    |  30   |   10  |





