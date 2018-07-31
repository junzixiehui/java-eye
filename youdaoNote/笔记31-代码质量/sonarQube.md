

[3.17.2 Jenkins集成Sonar
](https://www.kancloud.cn/devops-centos/centos-linux-devops/392411)


[单元测试覆盖率](https://ningyu1.github.io/site/post/77-jenkins-sonarqube-jacoco-junit/)


- [SonarQube 之 gitlab-plugin 配合 gitlab-ci 完成每次 commit 代码检测](https://blog.csdn.net/aixiaoyang168/article/details/78115646)




- [sonar-gerrit plugin配置](http://www.cnblogs.com/spillage/p/6181934.html)
- 
https://www.jianshu.com/p/f11fbd1c316e


[高版本sonar安裝遇到的坑-sonar 6.6
](https://hk.saowen.com/a/2fe9ee18baecb2a4374681ecf6505bc59ed46ba57c21cdaaea09eca22b79431f)

创建规则 https://blog.csdn.net/cspecialy/article/details/80087375

> 错误原因：因为安全问题elasticsearch 不让用root用户直接运行

 - 解决方法：liunx创建新用户sonarUser,使用该用户（sonarUser）运行sonar即可。

> 步骤如下：

- 创建用户
$ adduser sonaruser
- 为用户创建密码
$ passwd sonaruser
- 输入两次密码，
- 修改sonar的目录和用户组为sonaruser
$ chown -R sonaruser:sonaruser sonarqube-6.7.2
- 重新启动sonar
$ ./sonar.sh start



> 
原因：当前用户没有执行权限 
解决方法： chown linux用户名 elasticsearch安装目录 -R 
例如：chown ealsticsearch /data/wwwroot/elasticsearch-6.2.4 -R




### linux安装部署

#### 环境

- jdk1.8
- mysql5.6及以上
- sonarqube 6.7.4


#### 安装

- 下载 SonarQube-ç.zip
- 解压 
```
cd /usr/local/src

wget https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-6.7.4.zip

unzip SonarQube-6.7.4.zip

mv sonarqube-6.7.4 /usr/local/

ln -s /usr/local/sonarqube-6.7.4/ /usr/local/sonarqube

```

- 配置数据库

```
文件 /conf/sonar.properties

sonar.jdbc.url=jdbc:mysql://xxx.xxx.xxx.xxx:3306/xxxx?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance 
sonar.jdbc.username=xxxx 
sonar.jdbc.password=xxxx

```

- 启动

```
#启动
/usr/local/sonarqube/bin/linux-x86-64/sonar.sh start
#重启
/usr/local/sonarqube/bin/linux-x86-64/sonar.sh restart
#停止
/usr/local/sonarqube/bin/linux-x86-64/sonar.sh stop

```

- 访问登录

```
http://xxxx:9000
登陆：用户名：admin 密码：admin 
```

- 安装插件
```
汉化插件
进入配置--应用市场--搜索Chinese Pack安装简体中文汉化包，安装完成重启

```