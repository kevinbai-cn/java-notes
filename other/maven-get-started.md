Maven，为 Java 项目打造的管理和构建工具，主要功能有

- 提供了一套标准化的项目结构
- 提供了一套标准化的构建流程（编译、测试、打包、发布...）
- 提供了一套依赖管理机制

# 安装 Maven

根据平台下载相应的压缩包，解压后将 bin 目录添加到环境变量 Path 中

下载地址：https://maven.apache.org/download.cgi

以 macOS 平台为例

```
$ unzip apache-maven-3.6.3-bin.zip
$ mv apache-maven-3.6.3 /opt/apache-maven-3.6.3
$ export PATH="/opt/apache-maven-3.6.3/bin:$PATH"
$ mvn --version
```

上面的 export 语句可以放到 `~/.bashrc` 文件里，这样就不用每次手动 export 了

# 创建项目

创建项目 my-app

```
$ mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

目录结构为

```
my-app
|-- pom.xml
`-- src
    |-- main
    |   `-- java
    |       `-- com
    |           `-- mycompany
    |               `-- app
    |                   `-- App.java
    `-- test
        `-- java
            `-- com
                `-- mycompany
                    `-- app
                        `-- AppTest.java
```

其中

- my-app，项目名
- pom.xml，项目描述文件
- src/main/java，存放 Java 源码
- src/test/java，存放测试源码

看下 pom.xml 文件

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

其中，

- groupId，类似于 Java 的包名，通常是公司或组织名称
- artifactId，类似于 Java 的类名，通常是项目名称
- version，版本

一个 Maven 工程就是由 groupId、artifactId 和 version 作为唯一标识。我们在引用其他第三方库的时候，也是通过这 3 个变量确定

# 理解依赖中的 scope

- compile，表示编译时需要用到该 jar 包（默认），比如 commons-logging
- test，表示仅在测试时使用，正常运行时并不需要，比如 junit
- runtime，表示编译时不需要，但运行时需要，比如 mysql
- provided，依赖表示编译时需要，但运行时不需要，比如 servlet-api

# 构建流程

Maven 通过生命周期（lifecycle）、阶段（phase）和 goal 来提供标准的构建流程。

Maven 的生命周期由一系列阶段构成，以内置的生命周期 default 为例，它包含以下phase：

- validate
- initialize
- generate-sources
- process-sources
- generate-resources
- process-resources
- compile
- process-classes
- generate-test-sources
- process-test-sources
- generate-test-resources
- process-test-resources
- test-compile
- process-test-classes
- test
- prepare-package
- package
- pre-integration-test
- integration-test
- post-integration-test
- verify
- install
- deploy

如果我们运行 mvn package，Maven 就会执行 default 生命周期，它会从开始一直运行到 package 这个 phase 为止：

- validate
- ...
- package

如果我们运行 mvn compile，Maven 也会执行 default 生命周期，但这次它只会运行到 compile，即以下几个 phase：

- validate
- ...
- compile

Maven 另一个常用的生命周期是 clean，它会执行 3 个phase：

- pre-clean
- clean（注意这个 clean 不是 lifecycle 而是 phase）
- post-clean

**所以，我们使用 mvn 这个命令时，后面的参数是 phase，Maven 自动根据生命周期运行到指定的 phase。**

更复杂的例子是指定多个 phase，例如，运行 mvn clean package，Maven 先执行 clean 生命周期并运行到 clean 这个 phase，然后执行default 生命周期并运行到 package 这个 phase，实际执行的 phase 如下：

- pre-clean
- clean（注意这个 clean 是 phase）
- validate
- ...
- package

在实际开发过程中，经常使用的命令有：

- mvn clean：清理所有生成的 class 和 jar
- mvn clean compile：先清理，再执行到 compile
- mvn clean test：先清理，再执行到 test，因为执行 test 前必须执行 compile，所以这里不必指定 compile
- mvn clean package：先清理，再执行到 package

大多数 phase 在执行过程中，因为我们通常没有在 pom.xml 中配置相关的设置，所以这些 phase 什么事情都不做。

经常用到的 phase 其实只有几个：

- clean：清理
- compile：编译
- test：运行测试
- package：打包

执行一个 phase 又会触发一个或多个 goal：

- compile，对应执行 goal compiler:compile
- test，对应执行 goal compiler:testCompile 和 surefire:test

goal 的命名总是 abc:xyz 这种形式。

其实我们类比一下就明白了：

- lifecycle 相当于 Java 的 package，它包含一个或多个 phase
- phase 相当于 Java 的 class，它包含一个或多个 goal
- goal 相当于 class 的 method，它其实才是真正干活的

大多数情况，我们只要指定 phase，就默认执行这些 phase 默认绑定的 goal，只有少数情况，我们可以直接指定运行一个 goal。例如，启动 Tomcat 服务器

```
mvn tomcat:run
```

# 小结

本文简单介绍了 Maven 的作用，然后通过创建一个项目帮助大家了解了 Maven 项目的布局，最后对 Maven 中的依赖和构建流程做了个大致的说明。

# 参考

- https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html
- https://www.liaoxuefeng.com/wiki/1252599548343744/1255945359327200
