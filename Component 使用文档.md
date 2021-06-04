## 前言

**Component** 是一个强大又灵活的框架. 主要分为 服务发现 和 路由两大模块

下面以 androidx 的 java8 版本为例讲解. 如果不是 Androidx 或者 java8 去掉相应的描述就好了

## 添加项目配置

### 1. 添加依赖

这个是框架的实现. 你只要保证想用的地方能引用到就好了. 但是我是基本放在 BaseModule 中的. 这样其他 Module 就都能引用到了

![](https://img.shields.io/github/release/xiaojinzi123/Component.svg?label=JitPack&color=%233fcd12)

```groovy
// Androidx + java8
implementation com.github.xiaojinzi123.Component:component-impl:1.x.x-androidx-java8

// 非 Androidx + java8
// implementation com.github.xiaojinzi123.Component:component-impl:1.x.x-java8
// Androidx + java7
// implementation com.github.xiaojinzi123.Component:component-impl:1.x.x-androidx
// 非 Androidx + java7
// implementation com.github.xiaojinzi123.Component:component-impl:1.x.x
```

### 2. 为每一个业务模块添加注解驱动器

**请注意!!!! 这个是每一个业务模块都必须配置的!!!**

在每一个业务模块的 build.gradle 中的 defaultConfig 中的代码块中加入如下配置

```groovy
defaultConfig {
        ......
        javaCompileOptions {
            annotationProcessorOptions {
                // 配置业务模块的模块名称, 当然你也可以自己写死某个字符串
                arguments = ["HOST": project.getName()]
            }
        }
}
```

如果你的模块中配置了 Kotlin. 

```groovy
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-kapt'
```

```groovy
// 如果配置了 kotlin 
kapt com.github.xiaojinzi123.Component:component-compiler:1.x.x-androidx-java8 // 注意和上面依赖的版本号保持一致
// 如果没配置 Kotlin
annotationProcessor com.github.xiaojinzi123.Component:component-compiler:1.x.x-androidx-java8 // 注意和上面依赖的版本号保持一致
```

### 3. 配置 Gradle 插件(如果你要手动加载模块, 可以跳过这个步骤)

Gradle 插件主要的作用就是后续加载模块的时候可以自动加载

在**项目根目录下**的 **build.gradle** 中, 添加依赖：

```groovy
buildscript {
  		......
      dependencies {
	        ......
          classpath "com.github.xiaojinzi123.Component:component-plugin:1.x.x-androidx-java8"  // 注意和上面依赖的版本号保持一致
      }
 		
}
```

然后在 **app** 的 **build.gradle** 中应用这个插件

```groovy
apply plugin: 'com.android.application'
apply plugin: 'com.xiaojinzi.component.plugin'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-kapt'
......
```

### 4. 在 Application 中初始化

```java
Component.init(
                BuildConfig.DEBUG,
                Config.with(this)
                        // 表示是否采用 Gradle 插件配置的方式加载模块
                        .optimizeInit(true)
                        // 自动加载所有模块, 依赖上面的 optimizeInit(true)
                        .autoRegisterModule(true)
                        // 执行构建
                        .build()
);
// 如果你没有配置 Gradle 插件, 你需要手动加载模块
// ModuleManager.getInstance().registerArr("module1", "module2", ......);
```

## 开始使用

### 路由

**标记一个目标**

```java
@RouterAnno(
        host = "app", // host 是可选的,如果不写默认采用 build.gradle 中配置的 host
        path = "info"
)
public class InfoAct extends AppCompatActivity {
		// ......
}

@RouterAnno(
  			// 这个就代表上面的两个属性. 但是最少得包含一个 /
        hostAndPath = "app/info"
)
public class InfoAct extends AppCompatActivity {
		// ......
}
```

**发起跳转**

```java
Router.with().hostAndPath("app/info").forward();
```

### 路由使用进阶

- 详细的 [文档在这里](https://github.com/xiaojinzi123/Component/wiki/%E8%B7%B3%E8%BD%AC-%E4%BD%BF%E7%94%A8%E4%BB%A3%E7%A0%81%E8%B7%B3%E8%BD%AC)
- 使用 [Api 的方式跳转文档在这里](https://github.com/xiaojinzi123/Component/wiki/%E8%B7%B3%E8%BD%AC-%E6%8E%A5%E5%8F%A3%E8%B7%AF%E7%94%B1%E7%9A%84%E6%96%B9%E5%BC%8F)

### 服务发现

在 **BaseModule** 中写一个接口 

```java
public interface Service1 {
			String test();
}
```

在业务模块中去实现它, 并用注解 @ServiceAnno 标记实现了哪个接口

```java
@ServiceAnno(Service1.class)
public interface ServiceImpl1 implements Service1 {
			public String test() {
					return "hello world";
			}
}
```

使用它

```java
// 寻找服务
Service1 service1 = ServiceManager.get(Service1.class)
// 调用方法
String content = service1.test()
```

更详细的 [文档在这里](https://github.com/xiaojinzi123/Component/wiki/%E8%B7%A8%E6%A8%A1%E5%9D%97%E6%9C%8D%E5%8A%A1%E8%B0%83%E7%94%A8)

