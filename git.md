## Attentions  
---
1. 在上传本地文件到github时，使用命令`git push -u origin master`上传的是位于master区的文件，所以在上传的本地文件需要`add` `commit`后才能上传到远程库github中。
```javascript
zhouyj@zhouyj-ubuntu:~/docs$ git push -u origin master
Counting objects: 4, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 1.74 KiB | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To git@github.com:ergasterzhou/documents.git
   e7c6c5a..7ce2ab6  master -> master
分支 master 设置为跟踪来自 origin 的远程分支 master。
```

## Problems  
---
#### fatal: Could not read from remote repository.  
```javascript
zhouyj@zhouyj-ubuntu:~/docs$ git remote add origin git@github.com:ergasterzhou/documents.git
zhouyj@zhouyj-ubuntu:~/docs$ git push -u origin master
Warning: Permanently added the RSA host key for IP address '192.30.253.112' to the list of known hosts.
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

解决思路:
1. `you have the correct access rights`  
**正确的进入权限，应当检查SSH服务器的公钥与github账户设置中的公钥是否匹配。**  
2. `the repository exists`  
远程仓库是否存在，此处一般不会出现问题。  

解决方法：  
**检查本地SSH服务器是否存在，且有没有设置密钥。**  
```ruby
zhouyj@zhouyj-ubuntu:~/githome$ cd ~/.ssh
zhouyj@zhouyj-ubuntu:~/.ssh$ ls
config  id_rsa  id_rsa.pub  known_hosts
```
id_rsa代表ssh本地私钥，id_rsa.pub代表公钥，公钥要与github账户设置中的公钥是否匹配。使用`ssh-add -l`查询本地SSH公钥
```ruby
zhouyj@zhouyj-ubuntu:~/.ssh$ ssh-add -l
2048 01:c9:4e:68:8b:c9:51:93:8c:f8:08:5d:4a:9c:1f:14 zhouyj@zhouyj-ubuntu (RSA)

```

发现该公钥应该是Ubuntu与公司服务器之间的公钥，没有与github的公钥  
所以要生成并添加github的公钥
`ssh-keygen -t rsa -b 4096 -C "你的github邮箱"`生成公钥
```javascript
zhouyj@zhouyj-ubuntu:~/githome$ ssh-keygen -t rsa -b 4096 -C "alanzhou@allwinnertech.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/zhouyj/.ssh/id_rsa): /home/zhouyj/.ssh/id_github
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/zhouyj/.ssh/id_github.
Your public key has been saved in /home/zhouyj/.ssh/id_github.pub.
The key fingerprint is:
05:ff:da:65:85:93:64:b8:ca:c2:09:32:3c:ac:c9:17 alanzhou@allwinnertech.com
The key's randomart image is:
+--[ RSA 4096]----+
|        .    .o  |
|         o  .o o |
|   o      o  .+ .|
|    E .  . ..  o |
| . o = oSo .. o  |
|  + .   + oo o   |
|   .     .. .    |
|                 |
|                 |
+-----------------+

```
这里我新建了一个文件id_github存放公钥，否则会覆盖掉本机与服务器之间的公钥，而且相互覆盖，下次又得改一遍。

接下来在github上修改SSH公钥，[Adding a new SSH key to your GitHub account](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)  

最终结果
```ruby
zhouyj@zhouyj-ubuntu:~/docs$ git push -u origin master
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 3.41 KiB | 0 bytes/s, done.
Total 5 (delta 0), reused 0 (delta 0)
To git@github.com:ergasterzhou/documents.git
 * [new branch]      master -> master
分支 master 设置为跟踪来自 origin 的远程分支 master。
```



