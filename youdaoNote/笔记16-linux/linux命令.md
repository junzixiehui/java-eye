[TOC]


- 命令文档 http://linuxtools-rst.readthedocs.io/zh_CN/latest/base/01_use_man.html
- 参考命令 http://www.cnblogs.com/peida/archive/2012/12/05/2803591.html
- 命令 http://codingstandards.iteye.com/blog/786653
- find ./ -name "abc.txt" https://mp.weixin.qq.com/s/7fj6yNRdlQXQQrTq5OGWFA
- [qulibin@fortress /]$ scp /Users/qulibin/open-soft/sonarqube-6.7.4/extensions/plugins/sonar-findbugs-plugin-3.7.0.jar qulibin@XXXX.XXXX.cn:/tmp
- scp /tmp/sonar-findbugs-plugin-3.7.0.jar root@sonarqube:/data
- 定时任务crontab -l  | 
  crontab -e
  > 注意 sonarJava.sh里面的命令也要全局路径

```
02 03 * * * /bin/bash /data/java_project/sonarJava.sh
```
#### 权限命令

> 关于LINUX权限-bash: ./startup.sh: Permission denied
在执行./startup.sh,或者./shutdown.sh的时候，爆出了Permission denied，
其实很简单，就是今天在执行tomcat的时候，用户没有权限，而导致无法执行，
用命令chmod 修改一下bin目录下的.sh权限就可以了
chmod u+x *.sh 

#### 端口命令
- 根绝关键字查询占用情况: 
```
netstat -an|grep 8080
```
- 查看端口占用情况: 
```
lsof -i:5001
```
- 查看某个端口的连接情况
```
netstat -lap | fgrep port
```

#### 连接数

- 了解机器连接数情况
问题：1.2.3.4的sshd的监听端口是22，如何统计1.2.3.4的sshd服务各种连接状态(TIME_WAIT/ CLOSE_WAIT/ ESTABLISHED)的连接数。
 
> 参考答案：

```
netstat -n | grep 1.2.3.4:22 | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}’

netstat -lnpta | grep ssh | egrep “TIME_WAIT | CLOSE_WAIT | ESTABLISHED”

n [仅限于阿里云]
```

#### 文件

> 磁盘报警，清空最大文件
问题：找出服务器上，某个正在运行的tomcat产生的大量异常日志，找出该文件，并释放空间。不妨设该文件包含log关键字，并且大于1G。
 
参考答案：
第一步，找到该文件

```
find / -type f -name "*log*" | xargs ls -lSh | more 
du -a / | sort -rn | grep log | more
find / -name '*log*' -size +1000M -exec du -h {} \;
```

第二步，将文件清空
假设找到的文件为a.log

```
正确的情况方式应该为：echo "">a.log，文件空间会立刻释放。
很多同学：rm -rf a.log，这样文件虽然删除，但是因tomcat服务仍在运行，空间不会立刻释放，需要重启tomcat才能将空间释放。
```


#### 磁盘

> 磁盘IO异常排查
问题：磁盘IO异常如何排查，类似写入慢或当前使用率较高，请查出导致磁盘IO异常高的进程ID。
 
参考答案：

```
第一步：iotop -o 查看当前正在写磁盘操作的所有进程ID信息。

第二步：如果此时各项写入指标都很低，基本没有大的写入操作，则需要排查磁盘自身。可以查看系统dmesg或cat /var/log/message 看看是否有相关的磁盘异常报错，同时可以在写入慢的磁盘上touch 一个空文件看看，是否磁盘故障导致无法写入。
```

#### awk

https://mp.weixin.qq.com/s/ofvB31w5rRywIdUUtOjskw

#### sed

https://mp.weixin.qq.com/s/29iMrw1CFpmLiW36p40szw