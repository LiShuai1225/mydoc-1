#### building blocks

>每个gradle构建有三部分组成:projects,tasks和properties,每个gralde构建至少有一个project,一个project包含一个
>或多个tasks,project和tasks提供properties,可以被用来控制构建

#### projects

>在gradle术语里,一个project代表你构建的组件,例如一个jar文件,或者一个全局目标,像开发一个
>应用程序,你可能是来自maven的使用者,听到我前面说的可能很熟悉,gradle的build.gradle文件等价于maven的pom文件
>每个gradle构建脚本至少定义构建一个project,Gradle实例化org.gradle.api.Project类是基于你的在build.gradle的
>配置,并且通过project变量使他隐式可用

![](build.png)

>一个project可以创建新任务,加入依赖和配置,添加插件和其他构建脚本,很多属性类似name,descrition可以通过
>get和set方法得到,你不需要直接调用project变量,例如下面代码

	setDescriptin("myProject")
	prontln "Description of project $name: "+project.description

#### Tasks

>一个action是task执行时的最小单元,有时一个task的输入基于另一个task的输出,例如再打包一个jar文件之前需要编译
>java源码,下面是 org.gradle.api.Task 的接口

![](task.png)


#### Properties

> 每个project和task的实例提供的属性,可以使用getter和setter访问,一个property能够作为任务描述或者项目版本,
>你或许想声明一个变量引用一个文件,并且在构建脚本里使用,gradle允许你使用extra propertie.

##### EXTRA PROPERTIES

>许多gradle领域模型类支持特殊属性,属性被存储在一个键值对的hash里,加入属性需要使用ext命名空间,看下面例子

	project.ext.myProp = 'myValue'
	ext {
		someOtherProp = 123
	}
	assert myProp == 'myValue'
	println project.someOtherProp
	ext.someOtherProp = 567

##### GRADLE PROPERTIES

>在<USER_HOME>/.gradle目录下的gradle.properties文件中声明的变量可以直接在你的项目中使用,这些变量可以通过
>project实例访问这些属性,只能有一个gradle.properties文件,在<USER_HOME>/.gradle目录下,即使你有多个项目要处理
>看下面示例,我们假设下面的属性在properties中定义了

	exampleProp = myValue
	someOtherProp = 455
	
>你可以在项目中按照如下方式访问

	assert project.exampleProp == 'myValue'
	task printGradleProperty << {
		println "Second property: $someOtherProp"
	}
	

#### OTHER WAYS TO DECLARE PROPERTIES

>extra properties和gradle properties是你主要用来声明属性和值,gradle允许你许多其他方式提供属性用来构建

> Project property via the –P command-line option
> System property via the –D command-line option
> Environment property following the pattern
> ORG_GRADLE_PROJECT_propertyName=someValue

#### Working with tasks

>新定义的任务是org.gradle.api.DefaultTask类型.实现了org.gradle.api.Task接口,DefaultTask的所有属性都是私有
>只能使用public的setter和getter方法来访问,幸好groovy的语法糖可以使用属性名称访问


#### Managing the project version

![](properties.png)

>如上图所示,project的版本在构建期间从一个属性文件中读取,这个ProjectVersion数据类被初始化,版本的每一个分类被
>传递给数据类的属性,ProjectVersion类实例被分配给Project的version属性

>能够编程的方式控制版本结构,对于更多的自动化你的项目构建很重要.例如一个例子,你已经通过所有功能测试,准备发送代码,当前版本
>是1.2-SNAPSHOT. 在构建最终的war文件之前,你想变成release version 1.2,并且自动部署到生产服务器. 
>这个过程中可以通过创建一个任务:修改项目版本号和部署war文件

#### Declaring task actions

> task里的action就是存放构建逻辑最适合的地方， Task接口提供2个合适的方法来定义一个task的action，doFirst(闭包)
>和doLast(闭包),当一个task执行的时候，定义的action逻辑作为一个闭包,像参数那样被轮流执行
>可以定义一个简单的task叫做printVersion。这个task的目的打印当前project版本。 定义逻辑作为task的最后action，代码
>如下

	version = '0.1-SNAPSHOT'
	task printVersion {
		doLast {
			println "Version: $version"
		}
	}
	
>通过使用doFirst可以达成同样目的

	task printVersion {
		doFirst {
			println "Version: $version"
		}
	}
	
#### ADDING ACTIONS TO EXISTING TASKS
	
>前面，你仅仅是加入一个单独的action到task里面，要么是frist要么是last，但是你不会被限制于每个task只有一个action，
>你可以加入许多actions到已经建立的task里面，每个task都有一组actions，运行时，按顺序执行，看下面的版本

	task printVersion {
		doFirst {
			println "Before reading the project version"
		}
		doLast {
			println "Version: $version"
		}
		}
	printVersion.doFirst { println "First action" }
	printVersion << { println "Last action" } // <<符号doLast别名

> 上面的例子所示，你可以添加actions到task里，这十分有用。例如你想执行一个自定义逻辑
>但是这个task里没有你自定义的action,例如你想添加一个doFirst action到编译java的插件task里，来检查至少有一个java
>源文件存在


#### Accessing DefaultTask properties

>下一步我们改进输出本版号方式，gradle提供了logger的实现基于SLF4J,除了常用的日志级别(DEBUG,ERROR,TRACE,INFO,WARN)之外，还加入了一些扩展级别， logger实例可以直接通过task的一个方法访问，下面是打印版本号使用log level QUIET:

	task printVersion << {
		logger.quiet "Version: $version"
	}
	
> 还有2个属性，group和description.，都可以作为task的文档，
> description属性表示一个task的目的的简短描述,group定义task的逻辑分组，设置这两个属性作为task的参数

	task printVersion(group: 'versioning',description: 'Prints project version.') << {
		logger.quiet "Version: $version"
	}

>也可以调用set方法设置
	
	task printVersion {
		group = 'versioning'
		description = 'Prints project version.'
		doLast {
			logger.quiet "Version: $version"
		}
	}
	

#### Defining task dependencies

> dependsOn允许你声明基于一个或多个任务的依赖

	task first << { println "first" }
	task second << { println "second" }
	task printVersion(dependsOn: [second, first]) << {
		logger.quiet "Version: $version"
	}

    task third << { println "third" }
	third.dependsOn('printVersion')
	

#### Finalizer tasks

>实践过程中，你或许发现需要一个可靠的方案去清理一个任务，在它执行完之后，典型的场景就是web容器需要部署程序之前
>运行集成测试， gradle给这种场景一个finalizer task，即使这个finalizer task运行失败，gradle也会定时调用，下面就是一个例子

	task first << { println "first" }
	task second << { println "second" }
	first.finalizedBy second

> 你会发现first执行完，自动触发second任务
