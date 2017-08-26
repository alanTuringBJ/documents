## JDK

以下是Ubuntu 14.04安装JDK1.8.0_25与配置环境变量过程笔记。

1. 官网下载JDK，下载相应的压缩包`jdk-xxx-linux-x64.tar.gz`
[JDK官网下载地址](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

2. 在`/usr/local`新建文件夹java
```ruby
sudo mkdir java
```

3. 压缩包拷贝到`/usr/local/java`目录下
```ruby
cp jdk-xxx-linux-x64.tar.gz /usr/local/java
```
4. 解压缩
```ruby
sudo tar xvf jdk-xxx-linux-x64.tar.gz 
```
## 设置jdk环境变量
这里采用全局变量的方式，设置用户共同的环境变量

```ruby
sudo getdit ~/.bashrc
```
末尾添加

```ruby
export JAVA_HOME=/usr/local/java/jdk1.8.0_25  
export JRE_HOME=${JAVA_HOME}/jre  
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
export PATH=${JAVA_HOME}/bin:$PATH
```
### 重要一步
```ruby
source ~/.bashrc
```

最后检验是否安装成功

```ruby
java -version
```