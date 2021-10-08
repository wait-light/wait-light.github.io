---
title: Maven记录
date: 2021-10-01 00:00:00
---

# 安装

1. 检查JAVA_HOME环境变量

   ![image.png](https://i.loli.net/2021/10/08/x69jbLTE2d8ZtG5.jpg)

2. 解压核心程序压缩包

   放在一个非中文，无空格的文件夹中。

3. 配置Maven环境变量

   - MAVEN_HOM（M2_HOME）

     地址为刚才解压出的文件的根目录。

     ![image.png](https://i.loli.net/2021/10/08/Wq39JRDbjNUg1MP.jpg)

   - 添加path

     ![image.png](https://i.loli.net/2021/10/08/uHqeAPJWxlGIoD5.jpg)
   
4. 验证是否安装成功
   在cmd中执行mvn -v，若是有相关版本输出，说明安装成功了。
   ![image.png](https://i.loli.net/2021/10/08/fmhue5d9DLkT1rp.jpg)



# Maven工程目录结构
```
		Hello
		|---src
		|---|---main
		|---|---|---java
		|---|---|---resources
		|---|---test
		|---|---|---java
		|---|---|---resources
		|---pom.xml
```
根目录：工程名
src目录：源码
pom.xml：Maven的核心配置文件
main目录：存放主程序
test目录：存放测试程序
java目录：存放Java源文件
resources目录：存放框架或者其他工具的配置文件



# Maven常用命令

**执行与构建过程相关的Maven命令，必须进入pom.xml所在的目录。**

**构建过程主要环节**

①**清理**：删除以前的编译结果，为重新编译做好准备。
②**编译**：将 Java 源程序编译为字节码文件。
③**测试**：针对项目中的关键点进行测试，确保项目在迭代开发过程中关键点的正确性。
④**报告**：在每一次测试后以标准的格式记录和展示测试结果。
⑤**打包**：将一个包含诸多文件的工程封装为一个压缩文件用于安装或部署。Java 工程对应 jar 包，Web工程对应 war 包。
⑥**安装**：在 Maven 环境下特指将打包的结果——jar 包或 war 包安装到本地仓库中。
⑦**部署**：将打包的结果部署到远程仓库或将 war 包部署到服务器上运行

## 命令

- mvn clean 清理
- mvn compile 编译主程序
- mvn test-compile 编译测试程序
- mvn test 执行测试
- mvn package 打包
- mvn install 安装当前Maven工程到仓库
- mvn site 生成站点

更多命令可以到本文**生命周期**中查看



# Maven setting配置

**maven默认配置setting.xml位置**：[用户家目录]\\.m2，即C:\Users\\[用户名]\\.m2

## 指定Maven工程JDK版本

```xml
<profile>
    <id>jdk-1.8</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
    
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
</profile>
```



## 仓库

Maven的核心程序中仅仅定义了抽象的生命周期，但是具体的工作必须由特定的插件来完成。而插件本身并不在Maven的核心程序中。当执行Maven命令需要用到某些插件时，Maven首先会到本地仓库中寻找，若是不存在则会到中央仓库下载，若是无法连接到中心仓库，或者插件不存在，则构建失败。

**本地仓库默认位置**：[用户家目录]\\.m2\\repository ，即C:\Users\\[用户名]\\.m2\\repository

### 仓库保存的内容

- Maven自身所需要的插件
- 第三方框架或者工具的jar包
- 自己开发的Maven工程

### 修改仓库

打开Maven根目录下的conf目录下的setting.xml

#### 本地仓库

修改\<localRepository\>\</localRepository\>

```xml
<!-- xxxx替换为指定的目录 -->
<localRepository>xxxx</localRepository>
```

#### 远程仓库

修改\<mirrors\>\</mirrors\>，添加mirror或者修改mirror。由于Maven中心仓库在中国下载速度慢，可以替换为国内仓库镜像。如下为阿里云仓库。

```xml
<mirror>
    <id>alimaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
</mirror>
```

## 完整配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <localRepository>D:\apache-maven-3.6.3\maven-repository</localRepository>

  <pluginGroups>
  </pluginGroups>
    
  <proxies>
  </proxies>

  <servers>
  </servers>
    
  <mirrors>
    <mirror>
      <id>alimaven</id>
      <mirrorOf>central</mirrorOf>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
    </mirror>
  </mirrors>

  <profiles>
    <profile>
      <id>jdk-1.8</id>
      <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
      </activation>

      <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
      </properties>
    </profile>
  </profiles>
</settings>

```



# POM

含义：Project Object Model（项目对象模型）

## 坐标

1. 使用下面三个向量唯一定位一个Maven工程

- groupid：公司或组织域名倒序 + 项目名

  ```xml
  <groupid>com.atguigu.maven</groupid>
  ```

- artifactid：模块名

  ```xml
  <artifactid>Hello</artifactid>
  ```

- version：版本

  ```xml
  <version>1.0.0</version>
  ```

2. 仓库与坐标的对应关系

   仓库中的文件路径与坐标一一对应。如上的坐标对应的仓库路径为：com/atguigu/maven/Hello/1.0.0

## 依赖

**样例**

```xml
<dependencies>
    <!-- 单个依赖 -->
    <!-- dependency必须在dependencies之中 -->
    <dependency>
        <!-- 公司或组织域名倒序 + 项目名 -->
        <groupId>org.slf4j</groupId>
        <!-- 模块名 -->
        <artifactId>slf4j-log4j12</artifactId>
        <!-- 版本 -->
        <version>1.6.6</version>
        <!-- 依赖范围 -->
        <scope>compile</scope>
    </dependency>
</dependencies>
```



### 依赖范围

**默认类型**：compile

样例中的\<scope\>compile\</scope\>说明这个依赖是compile类型的。

**类型**

- compile
- test

![compile和test范围依赖](https://i.loli.net/2021/10/08/iqcR1t2DsSmlIJ5.jpg)

- provided

  *下图为Servlet-API参与的编译范围，其编译范围为provided*
  
  ![provided范围依赖](https://i.loli.net/2021/10/08/BPnEv5twx82RWQj.jpg)

**三种类型对比**

  ![image.png](http://ww1.sinaimg.cn/large/008c2CqBly1gmypklf74yj30uy054glw.jpg)

解读：compile说明在编译主程序、测试程序、部署的时候都会参与进编译，而test只有在编译测试程序的时候会进行编译，provided不参与在部署的时候的编译。

### 依赖的传递性

**优点**：只需要在被需要的工程中添加依赖即可，无需在父工程中重复添加。有Maven统一管理。

**注意**：只有依赖范围为compile的依赖才有传递性，其他类型不具有传递性。

### 依赖的排除

 ```xml
<exclusions>
    <exclusion>
        <groupId>avax.transaction</groupId>
        <artifactId>javax.transaction-api</artifactId>
    </exclusion>
</exclusions>
 ```

### 依赖原则

- 就近原则

  ![image.png](https://i.loli.net/2021/10/08/4PQvI6XUihzWLEw.jpg)

- 先声明着优先（在dependencies的声明顺序）

  ![image.png](https://i.loli.net/2021/10/08/gm6Hek9rslLBv8q.jpg)

### 统一管理依赖版本号

1. 设置properties
 ```xml
<properties>
    <!-- 自定义标签名 -->
    <diy.version>5.0.2.RELEASE</diy.version>
</properties>
 ```

2. 使用 ${自定义的标签名} 统一管理

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <!-- 填入统一管理的自定义标签 -->
    <version>${diy.version}</version>
</dependency>
```

### 依赖继承

1. 建立Maven父工程

   ```xml
   <groupId>org.example</groupId>
   <artifactId>father</artifactId>
   <version>1.0-SNAPSHOT</version>
   ```

2. 子工程中声明父工程

   ```xml
   <parent>
       <groupId>org.example</groupId>
       <artifactId>father</artifactId>
       <version>1.0-SNAPSHOT</version>
       <!--以当前文件路径为基准的父工程的pom.xml路径-->
       <relativePath>../father/pom.xml</relativePath>
   </parent>
   ```

3. 子工程删除与父工程重复的内容(可以不删除)

4. 在父工程中统一对依赖进行管理

   ```xml
   <dependencyManagement>
       <dependencies>
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.13.1</version>
           </dependency>
       </dependencies>
   </dependencyManagement>
   ```

5. 删除子工程中父工程统一管理的依赖的版本信息

   ~~\<version\>xxxx\</version\>~~

**注意** : 配置继承后若是执行安装命令 , 则要先安装父工程

### 聚合

**作用**

一键安装各个模块工程

**配置**

在聚合工程配置各个子工程的相对路径

```xml
<modules>
    <!-- 子工程的相对路径 -->
    <module>./child</module>
</modules>
```

**使用**

在聚合工作的pom.xml下执行mvn install

### 构建

**自动化构建**

```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-invoker-plugin</artifactId>
            <version>1.6</version>
            <configuration>
                <debug>true</debug>
                <pomIncludes>
                    <pomInclude>app-web-ui/pom.xml</pomInclude>
                    <pomInclude>app-desktop-ui/pom.xml</pomInclude> 
                </pomIncludes>
            </configuration>
            <executions>
                <execution>
                    <id>build</id>
                    <goals>
                        <goal>run</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
<build>
```

[自动化构建 - 菜鸟教程](https://www.runoob.com/maven/maven-build-automation.html)

#### 静态资源构建

以下配置会使Maven构建时,将src/main/java下及其子目录下的所有properties和xml一起构建 , 即生成的target中会包含这些文件

```xml
<build>
    <resources>
        <resource>
            <!-- 位置 -->
            <directory>src/main/java</directory>
            <!-- 包括哪些文件 -->
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <!-- 关闭过滤，开启过滤的作用用指定的参数替换directory下的文件中的参数(eg. ${name}) -->
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```

### 查找依赖信息

[mvnrepositor](https://mvnrepository.com)

## 打包方式

- war包

  一个war包可以理解成一个web项目，里面是项目的所有东西

- jar包

  用于压缩和发布，打成包利于管理，主要存放项目需要的工具类

- pom包

  用在父级工程或聚合工程中，用来做jar包的版本控制

```xml
  <packaging>war</packaging>
```

## 完整配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 公司或组织域名倒序 + 项目名 -->
    <groupId>org.example</groupId>
    <!-- 模块名 -->
    <artifactId>project_library</artifactId>
    <!-- 版本 -->
    <version>1.0-SNAPSHOT</version>

    <!-- 打包方式 -->
    <packaging>war</packaging>
    <!-- 依赖 -->
    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.6.6</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${diy.version}</version>
            <!-- 依赖排除 -->
            <exclusions>
                <exclusion>
                    <groupId>avax.transaction</groupId>
                    <artifactId>javax.transaction-api</artifactId>
                </exclusion>
            </exclusions>  
        </dependency>
    </dependencies>

    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>
</project>
```



# 生命周期

## 标准生命周期

- **clean**：项目清理的处理

  - pre-clean：执行一些需要在clean之前完成的工作
  - clean：移除所有上一次构建生成的文件
  - post-clean：执行一些需要在clean之后立刻完成的工作

- **default(或 build)**：项目部署的处理

  一个典型的 Maven 构建（build）生命周期是由以下几个阶段的序列组成的：

  ![img](https://i.loli.net/2021/10/08/WDCtwgk3VBEXazx.png)

  | 生命周期阶段                                | 描述                                                         |
  | :------------------------------------------ | :----------------------------------------------------------- |
  | validate（校验）                            | 校验项目是否正确并且所有必要的信息可以完成项目的构建过程。   |
  | initialize（初始化）                        | 初始化构建状态，比如设置属性值。                             |
  | generate-sources（生成源代码）              | 生成包含在编译阶段中的任何源代码。                           |
  | process-sources（处理源代码）               | 处理源代码，比如说，过滤任意值。                             |
  | generate-resources（生成资源文件）          | 生成将会包含在项目包中的资源文件。                           |
  | process-resources （处理资源文件）          | 复制和处理资源到目标目录，为打包阶段最好准备。               |
  | compile（编译）                             | 编译项目的源代码。                                           |
  | process-classes（处理类文件）               | 处理编译生成的文件，比如说对Java class文件做字节码改善优化。 |
  | generate-test-sources（生成测试源代码）     | 生成包含在编译阶段中的任何测试源代码。                       |
  | process-test-sources（处理测试源代码）      | 处理测试源代码，比如说，过滤任意值。                         |
  | generate-test-resources（生成测试资源文件） | 为测试创建资源文件。                                         |
  | process-test-resources（处理测试资源文件）  | 复制和处理测试资源到目标目录。                               |
  | test-compile（编译测试源码）                | 编译测试源代码到测试目标目录.                                |
  | process-test-classes（处理测试类文件）      | 处理测试源码编译生成的文件。                                 |
  | test（测试）                                | 使用合适的单元测试框架运行测试（Juint是其中之一）。          |
  | prepare-package（准备打包）                 | 在实际打包之前，执行任何的必要的操作为打包做准备。           |
  | package（打包）                             | 将编译后的代码打包成可分发格式的文件，比如JAR、WAR或者EAR文件。 |
  | pre-integration-test（集成测试前）          | 在执行集成测试前进行必要的动作。比如说，搭建需要的环境。     |
  | integration-test（集成测试）                | 处理和部署项目到可以运行集成测试环境中。                     |
  | post-integration-test（集成测试后）         | 在执行集成测试完成后进行必要的动作。比如说，清理集成测试环境。 |
  | verify （验证）                             | 运行任意的检查来验证项目包有效且达到质量标准。               |
  | install（安装）                             | 安装项目包到本地仓库，这样项目包可以用作其他本地项目的依赖。 |
  | deploy（部署）                              | 将最终的项目包复制到远程仓库中与其他开发者和项目共享。       |

- **site**：项目站点文档创建的处理

  - pre-site：执行一些需要在生成站点文档之前完成的工作
  - site：生成项目的站点文档
  - post-site： 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
  - site-deploy：将生成的站点文档部署到特定的服务器上

## 执行

Maven核心程序为了更好的实现自动化构建，按照如下特点执行：不论要执行生命周期的哪一个阶段，都是从这个生命周期的最初的位置开始执行。

例如：

- 执行mvn clean，将运行以下两个生命周期阶段：pre-clean, clean
- 执行mvn post-clean则运行以下三个生命周期阶段：pre-clean, clean, post-clean

 



参考资料来源：[Maven构建生命周期-菜鸟教程](https://www.runoob.com/maven/maven-build-life-cycle.html)

参考资料来源：[尚硅谷视频](http://www.atguigu.com)