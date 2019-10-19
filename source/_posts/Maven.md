---
title: Maven
date: 2019.06.24 15:32:42
tags: IDE
categories: IDE
---


##### 创建Maven模块
1.选择archetype进行创建
![123.jpg](/images/Maven/123.jpg)
2.添加属性
archetypeCatalog -> internal
![456.jpg](/images/Maven/456.jpg)

maven的pom.xml：
<packaging>war</packaging>  用于部署  
<packaging>jar</packaging>    默认打包成jar，用于dependency
<packaging>pom</packaging> 父pom


##### Maven命令
Maven package:完成了项目编译、单元测试、打包功能，**但没有把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库和远程maven私服仓库**
Maven install: 完成了项目编译、单元测试、打包功能，同时把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库，**但没有布署到远程maven私服仓库**
Maven deploy:完成了项目编译、单元测试、打包功能，同时把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库和远程maven私服仓库

**pom聚合工程的部署方法**
先打包父项目ps-parent: Maven install
再打包子项目ps-clientebase和ps-clientebase: Maven install
![image.jpg](/images/Maven/pom.jpg)

ps-parent的pom.xml添加插件
```
   <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <!-- 资源文件拷贝插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>2.7</version>
                <configuration>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <!-- java编译插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.2</version>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
        <pluginManagement>
            <plugins>
                <!-- 配置Tomcat插件 -->
                <plugin>
                    <groupId>org.apache.tomcat.maven</groupId>
                    <artifactId>tomcat7-maven-plugin</artifactId>
                    <version>2.2</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
```

ps-clientbase-web的pom.xml添加配置
```
  <!-- 配置tomcat热部署插件,默认支持到Tomcat7，但是对于目前最新的Tomcat9同样可以使用该插件（虽然插件的ArtifactId的名字为tomcat7很奇怪）-->
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <configuration>
          <path>/${project.artifactId}</path>                <!--部署到哪个目录下-->
          <port>8081</port>
          <url>http://XXX:8081/manager/text</url>  <!--部署到哪台服务器, 使用的是tomcat的默认项目manager进行deploy, 因此/manager/text是manager固定的接口。用户名密码使用的是tomcat的tomcat-users.xml配置文件中具备manager-script的用户名密码, 另外需要在maven的setting.xml配置同样的用户名密码-->
          <username>root</username>
          <password>1111</password>

          <server>tomcatServer</server>
          <update>true</update>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

ps-clientbase的pom.xml添加配置
```
    <!-- 配置tomcat热部署插件,默认支持到Tomcat7，但是对于目前最新的Tomcat9同样可以使用该插件（虽然插件的ArtifactId的名字为tomcat7很奇怪）-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <configuration>
                    <path>/${project.artifactId}</path>                <!--部署到哪个目录下-->
                    <port>8080</port>
                    <url>http://xxx:8080/manager/text</url>  <!--部署到哪台服务器, 使用的是tomcat的默认项目manager进行deploy, 因此/manager/text是manager固定的接口。用户名密码使用的是tomcat的tomcat-users.xml配置文件中具备manager-script的用户名密码, 另外需要在maven的setting.xml配置同样的用户名密码-->
                    <username>xxx</username>
                    <password>xxx</password>

                    <server>tomcatServer</server>
                    <update>true</update>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

最后使用命令tomcat7:deploy（注意不是maven:deploy - -）即可完成热部署
**maven的tomcat远程热部署**
资料：[https://www.cnblogs.com/xyb930826/p/5725340.html](https://www.cnblogs.com/xyb930826/p/5725340.html)
