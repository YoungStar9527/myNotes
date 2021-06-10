# 1 maven的前身

1、make：最原始的构建工具，不能跨平台

2、ant：曾经没有maven的时候，流行过一段时间，但是手工配置的语法繁琐，而且需要一次又一次的重复，另外依赖管理还要借助ivy来完成，工作量还是有点大。。。

3、maven：自动化。。。，目前是最有影响力的工程管理工具

4、gradle：google发布，不再基于xml来进行配置，而是基于DSL语言来进行构建管理，语法功能更加强大，android这块用的较多，同时国外一些开源项目，比如spring也开始用gradle来管理

# 2 maven基础

## 2.1 环境变量

首先普及一下，MAVEN_HOME是maven 1的写法、M2_HOME是maven 2的写法，但实际上这只是一种命名习惯，对实际作用没有任何影响。

  maven现在普遍的版本是3.5，3.6之后官网会推崇另外一种写法，不使用任何中间路径替代：

```shell
   export PATH=/opt/apache-maven-3.6.0/bin:$PATH
```


如果你的设置未生效，或者不对应，请按照上述进行设置（配置maven之前务必配置JAVA_HOME）

设置MAVEN_OPTS环境变量，就是设置maven的jvm参数，可以设置为-Xms128m -Xmx512m。

## 2.2 初始化

可选择运行mvn help:system

maven安装目录里的conf目录下的settings.xml配置文件拷贝到.m2目录里去，就ok了，作为以后maven全局唯一的配置文件

```shell
mvn help:system
#初始化，在用户目录下创建对应m2目录，并下载相关文件
```

## 2.3 pom介绍

```xml
<project>：pom.xml中的顶层元素

<modelVersion>：POM本身的版本号，一般很少变化

<groupId>：创建这个项目的公司或者组织，一般用公司网站后缀，比如com.company，或者cn.company，或者org.zhonghuashishan

<artifactId>：这个项目的唯一标识，一般生成的jar包名称，会是<artifactId>-<version>.<extension>这个格式，比如说myapp-1.0.jar

<packaging>：要用的打包类型，比如jar，war，等等。

<version>：这个项目的版本号

<name>：这个项目用于展示的名称，一般在生成文档的时候使用

<url>：这是这个项目的文档能下载的站点url，一般用于生成文档

<description>：用于项目的描述
```



## 2.4 相关命令

快速构建maven项目

```shell
mvn archetype:generate 
#快速构建maven项目 下面对应包名,id等需要指定，但是一起输入会pom文件错误·
-DgroupId=com.zhss.maven 
-DartifactId=maven-first-app 
-DarchetypeArtifactId=maven
-archetype
-quickstart 
-DinteractiveMode=false
##########################
java -cp target/maven-first-app-1.0-SNAPSHOT.jar com.zhss.maven.App
#执行jar包

```

## 2.5 maven流程图

![image-20210608070020341](maven小知识.assets/image-20210608070020341.png)



## 2.6 maven坐标

每个maven项目都有一个坐标

groupId + artifactId + version + packaging + classifier，五个维度的坐标，唯一定位一个依赖包

任何一个项目，都是用这五个维度唯一定位一个发布包

实际上后面两个维度较为少用，99%的场景下，唯一定位一个依赖的就是三个维度，groupId + artifactId + version

**groupId**：不是你的公司或者组织，但是以你的公司或者组织的官网的域名倒序来开头，然后加上项目名称

www.baidu.com，公司里任何一个项目的开头，就可以用com.baidu来打头

com.zhss + 项目名称，或者是系统名称，maven就叫com.zhss.maven

**artifactId**：项目中的某个模块，或者某个服务

com.zhss.oa，oa-organ，organ是缩写，organization

com.zhss.oa，oa-auth，auth是authorization

com.zhss.oa，oa-flow，flow就是流程的意思

 **version**：这个工程的版本号

**packaging**：这个工程的发布包打包方式，一般常用的就jar和war两种，java -cp执行一个jar包，war可以放到一个tomcat容器里去跑的web工程

**classifier**：很少用，定义某个工程的附属项目，比如hello-world工程的，hello-world-source工程，就是源码，可能是类似于hello-world-1.0-SNAPSHOT-source.jar这样的东西。

**坐标作用**：

​	**工程写好了，都有版本，groupId+artifactId+version就成了这一次这个工程目前这个状态的唯一的标识和定位，就可以提供给别人使用(jar包方式打包提供)**，

## 2.6 dependency 引入依赖

```xml
<dependency>
	<groupId></groupId>
    <!--包路径-->
	<artifactId></artifactId>
    <!--服务名-->
	<version></version>
    <!--版本-->
	<type></type>
    <!--极少使用。。。。-->
	<scope></scope>
    <!--使用范围-->
	<optional></optional>
    <!--是否传递依赖-->
</dependency>
```

三要素：groupId、artifactId、version 常用(声明了就会定位依赖，本地没有就网上下载到本地)

type、scope、optional不常用

### 2.6.1 scope依赖范围

**PS:三种范围，编译、测试、运行**

compile：默认，对编译、测试和运行的classpath都有效

test：仅仅对于运行测试代码的classpath有效，编译或者运行主代码的时候无效（juit依赖可以使用）

provided：编译和测试的时候有效，但是在运行的时候无效(servlet-api，容器已提供)

runtime：测试和运行classpath有效，但是编译代码时无效(mysql驱动类，正常代码不需要直接引用到驱动类内部api)

### 2.6.2 传递性依赖

​	**PS:传递性依赖就是一个依赖包中还依赖其他的包，maven会自动传递解析**

​	**maven的传递性依赖，就是说会自动递归解析所有的依赖**，然后负责将依赖下载下来，接着所有层级的依赖，都会成为我们的项目的依赖，不需要我们手工干预。所有需要的依赖全部下载下来，不管有多少层级。这个就是maven的传递性依赖机制，自动给我递归依赖链条下载所有依赖的这么一个特性。

传递性依赖对应范围

| 一级\二级 | compile  | test | provided | runtime  |
| --------- | -------- | ---- | -------- | -------- |
| compile   | compile  |      |          | runtime  |
| test      | test     |      |          | test     |
| provided  | provided |      | provided | provided |
| runtime   | runtime  |      |          | runtime  |

### 2.6.3 依赖调节

传递性依赖会出现依赖冲突，maven根据**两大就近原则**解决：

**（1）就近原则：**

比如A->B->C->X(1.0)，A->D->X(2.0)，A有两个传递性依赖X，不同的版本

此时就会依赖调解，就近原则，离A最近的选用，就是X的2.0版本

**(2) 优先声明原则：**

如果A->B->X(1.0)和A->D->X(2.0)，路径等长呢？

那么会选择第一声明原则，哪个依赖在pom.xml里先声明，就用哪个

### 2.6.4 optional可选依赖

```xml
<optional>true</optional>
<!--此时依赖传递失效，不会向上传递-->
```

## 2.7 解决依赖冲突问题

**问题：**

maven会自动依赖调解，说对已给项目不同的版本选择一个版本来使用

但是如果**maven选择了错误的版本呢**

A包引用了C1.0(路径短)，B包引用了C2.0(路径长)，根据依赖调节选用了C1.0版本，这个时候B包调用C2.0中的方法就可能会产生ClassNotFound等异常

**解决：**

首先使用 

```shell
mvn dependency:tree
#查找当前项目pom的所有依赖链条
```

![image-20210608210124820](maven小知识.assets/image-20210608210124820.png)

再根据报错是哪个包，在信息中找到相关的包，然后对应包中排除低版本的包即可

```xml
  		<dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-core</artifactId>  
            <version>5.2.15.RELEASE</version>
            <!--排除spring-jcl-->
            <exclusions>
                <exclusion>
                    <groupId>org.springframework</groupId>
                    <artifactId>spring-jcl</artifactId>
                </exclusion>
            </exclusions>
        </dependency>  
```

（1）maven工作中最常见的依赖冲突问题的现象？

不同版本的包冲突

（2）产生的原因是什么？

传递依赖和依赖调节导致项目使用了低版本的包

（3）解决的思路

找出低版本的包排除

（4）具体用什么命令和配置去解决

用mvn dependency:tree找出所有依赖关系，再用exclusion将低版本包排除
