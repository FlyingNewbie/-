# gradle笔记

## 一、基础知识

### 1.1、groovy语言

​	Groovy适用于Java虚拟机的一种敏捷的动态语言，他是一种成熟的面向对象编程语言，既可以用于面向对象编程，又可以用作纯粹的脚本语言，使用该语言不必编写过多的代码，同时又具有闭包和动态语言的其他特性。

### 1.2、与java的区别

1.2.1 、编译器属性自动添加getter和setter方法

1.2.2 、属性可以直接用点号获取

1.2.3 、 最后一个表达式的值会作为返回值

1.2.4 、==等同于equals()，不会有NullPointerExceptions



### 1.3、groovy的特性

##### 1.3.1、assert语句

​	

```groovy
assert  version == 2
```

1.3.2、可选类型定义（弱类型）

​	

```groovy
def  version = 1
```



##### 1.3.3、可选的括号（括号可写可不写）

​	

```groovy
println(version)

println version
```



##### 1.3.4、字符串

​	

```groovy
def  s1 = 'string'      //单引号 ，纯粹是一个字符串

def  s2 = "gradle  version is  ${version}"  //可以插入变量

def  s3 = '''My  name

		    is gradle'''      //支持换行
```



##### 1.3.5、集合API

```
	//list

	def	buildTools = ['ant','maven']

	buildTools << 'gradle'

	assert	buildTools.getClass() == ArrayList

	assert	buildTools.size() == 3

	//map

	def	buildYears = ['ant':2000,'maven':2004]

	buildYears.gradle=2009

	

	println  buildYears.ant

	println	buildYears['gradle']

	println	buildYears.getClass()     //LinkedHashMap
```



##### 1.3.6、闭包

​	

```groovy
def	c1 = {

		v  ->   

				print	v

	}

	def	c2 = {

		print	'hello'

	}

	def	method1(Closure	closure){

		closure('param')

	}

	def	method2(Closure	closure){

		closure()

	}

	

	method1(c1);

	method2(c2);


```

## 二、使用gradle创建项目

### 2.1、使用Idea创建一个新的gradle项目

​	第一步：new Project

​			![1550315600650](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1550315600650.png)

​	第二步：输入GroupId和ArtifactId

​	![1550315684096](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1550315684096.png)

第三步：选择本地gradle和jdk版本

![1550315806646](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1550315806646.png)

### 2.2、项目中build.gradle

```groovy
plugins {
    id 'java'   //添加java插件
    id 'war'    //添加war插件 可打包为war包
} 

//添加插件的另外一种写法
//  apply  plugin:'java'

group 'com.gradle'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8   //指定jdk版本

//依赖仓库
repositories {
    mavenCentral()  //中央仓库
}

//依赖
dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

IDEA编译gradle项目在右侧栏目里面

![1550317191992](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1550317191992.png)

编译完成的jar包在build包的lib下面

![1550317433046](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1550317433046.png)

![image-20200602231802873](images/image-20200602231802873.png)

## 三、构建脚本

#### 3.1构件块

​	Gradle构建中的两个基本概念是项目（project）和任务（task），每个构建至少包含一个项目，项目中包含一个或多个任务。在多项目构建中，一个项目可以依赖于其他项目；类似的，任务可以形成一个依赖关系图来确保他们的执行顺序。

#### 3.2 项目（project）

​	一个项目代表一个正在构建的组件（比如一个jar文件），当构建启动后，Gradle会基于build.gradle实例化一个org.gradle.api.Project类，并且能够通过project变量使其隐式使用

​	项目属性：

​		1、group、name、version  。三个唯一确定一个项目

​		2、apply、dependencies、repositories、task

​		3、	属性的其他配置方式:ext、gradle.properties

#### 3.3、任务

​	任务对应org.gradle.spi.task。主要包括任务动作和任务依赖。任务动作定义了一个最小的工作单元。可以定义依赖其他任务、动作序列和执行条件

​	任务方法：

​		1、dependsOn

​		2、doFirst、doLast  <<

#### 3.4、自定义任务

​	

```
//创建闭包
def createDir = {
    path ->
        File file = new File(path);
        if(!file.exists()){
            file.mkdirs();
        }
}

task makeJavaDir(){
    def paths = ['src/main/java','src/main/resources','src/test/java','src/test/resources'];
    doFirst{
        paths.forEach(createDir);
    }
}

task makeWebDir(){
    dependsOn 'makeJavaDir'
    def paths = ['src/main/webapp','src/test/webapp']
    doLast{
        paths.forEach(createDir);
    }
}
```

​			

## 四、依赖管理

### 4.1、依赖阶段配置

- compile、runtime
- testCompile、testRuntime

### 4.2、依赖阶段关系

​	![1550331444681](C:\Users\GaoKang\AppData\Roaming\Typora\typora-user-images\1550331444681.png)

### 4.3、依赖仓库

```groovy
repositories {
	//多个仓库时按照顺序来寻找依赖
    mavenLocal()     //本地maven仓库
    maven{ url 'http://maven.aliyun.com/nexus/content/groups/public/'}   //私服仓库
    mavenCentral()  //远程仓库
}
```

### 4.4、解决版本冲突

- 发现版本冲突构建失败

  ```
  configurations.all {
      resolutionStrategy{
          failOnVersionConflict()
      }
  }
  ```

- 去除相应的依赖

  ```
  dependencies {
      compile (group:'',name:'',version:''){
          exclude group:'',module:''   //在相应的依赖中去除这个依赖
      }
      testCompile group: 'junit', name: 'junit', version: '4.12'
  }
  ```

- 强制使用某一个版本

  ```
  configurations.all {
      resolutionStrategy{
          failOnVersionConflict()
          force("junit:junit:4.13")   //强制指定一个版本
      }
  }
  ```

## 五、多项目构建

### 5.1、在根模块中的settings.gradle中添加

```
include '模块名'  //添加所有的子模块
```

### 5.2、一个模块依赖于另外一个模块

```
dependencies {
    compile project(:模块名)  //添加需要的依赖模块名
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

### 5.3、所有项目统一配置

```
//在根项目中配置统一配置
allprojects {
    sourceCompatibility = 1.8
}

//为所有子项目添加统一配置
subprojects {
    
}
```

### 5.4、统一配置 group和version

​	在根项目添加gradle.properties,在里面添加group和version;

## 六、发布

```
//添加maven发布插件
apply plugin 'maven-publish'

publishing{
    publications{
    	//定义一个发布任务，可以添加多个
        myPublish(MavenPublication){
            from components.java
        }
    }
    repositories {
        maven{
            name 'myRepo'
            url ''
        }
    }
}
```