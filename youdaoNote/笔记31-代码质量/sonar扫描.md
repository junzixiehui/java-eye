
参考 https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Maven
#### 准备
 - maven 3.x
 - sonarqube 5.6+
 - java8
 

#### maven settings


```
<settings>
    <pluginGroups>
        <pluginGroup>org.sonarsource.scanner.maven</pluginGroup>
    </pluginGroups>
    <profiles>
        <profile>
            <id>sonar</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <!-- Optional URL to server. Default value is http://localhost:9000 -->
                <sonar.host.url>
                  http://myserver:9000
                </sonar.host.url>
            </properties>
        </profile>
     </profiles>
</settings>
```
#### 执行


- 在pom.xml文件位置 执行


```
mvn clean verify sonar:sonar
  
# In some situation you may want to run sonar:sonar goal as a dedicated step. Be sure to use install as first step for multi-module projects
mvn clean install
mvn sonar:sonar
 
# Specify the version of sonar-maven-plugin instead of using the latest. See also 'How to Fix Version of Maven Plugin' below.
mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.4.1.1170:sonar
```
#### 覆盖率

- 为了生成覆盖报告；读 https://docs.sonarqube.org/display/PLUG/Code+Coverage+by+Unit+Tests+for+Java+Project
- Java插件将重用报表而不生成报表，因此在尝试配置分析以导入这些报表之前，您需要确保它们是正确生成的而不是空的。
- 测试执行报告必须符合JUnit XML格式。
- 必须使用JaCoCo生成代码覆盖率报告
- 您需要使用以下参数提供测试执行路径和代码覆盖率报告。
- 



> SonarJava不支持导入Cobertura覆盖数据。 存在一个sonar -cobertura插件，但维护和支持完全基于社区。 更一般地说，还要注意SonarQube提供了与工具无关的通用解决方案：通用测试数据。
