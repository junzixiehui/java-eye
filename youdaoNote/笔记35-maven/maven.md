

- 越不起眼越值得学习
- setting.xml
- pom.xml
- 

#### 简介

- 坐标：世界上任何一个构件都可以使用maven坐标唯一标识。
 
- groupId：定义当前Maven项目隶属的实际项目。由于Maven项目和实际项目未必是一对一的关系，比如SpringFramework这个实际项目可能对应的Maven项目有很多，像core、context、expression等等，因此groupId不应该对应项目隶属的公司或组织，否则artifact将很难定义
- artifactId：定义实际项目中的一个Maven模块，推荐的做法是使用实际项目名称作为artifactId的前缀，这样会很方便去寻找实际构件
version：定义Maven项目当前所处的版本，如上面的spring-context就是4.2.6的，RELEASE表示正式发行版本
- packing：定义Maven项目的打包方式，这项不是必须的，没列出来，不定义默认就是jar的打包方式
- classifier：帮助定义构件输出的一些附属构件，比如xxx-javadoc.jar、xxx-sources.jar，附属构件与主构件对应，这项是不能直接定义的
