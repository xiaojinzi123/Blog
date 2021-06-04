## 视频讲解(比较久之前的了, 建议看下文文档)

- [youtube资源](https://youtu.be/sH1vbjAahkA)
- [bilibili资源](https://www.bilibili.com/video/av54974120)

## 特殊说明!!!

- `app` **模块在项目中既是一个壳工程也是一个业务模块.这点务必注意**
- `App` **会包含其他所有的业务模块**
- **而一般工程中会有一个 "BaseModule" 的基础业务模块,每一个业务模块都会依赖 "BaseModule"**

## 依赖

![](https://img.shields.io/github/release/xiaojinzi123/Component.svg?label=LastVersion&color=%233fcd12)

**下面所有的 `<version>` 请替换成上面的版本号**

> 如果你需要的不是 java8, 那就去掉 -java8
>
> 如果你也不需要 androidx, 那就去掉 -androidx 即可

在 `(工程)Project` 级别的 build.gradle 中添加 maven 地址：

```
maven { url 'https://jitpack.io' }
```

在基础业务模块 `BaseModule` 或者每一个业务模块中添加 **Component** 的依赖使用.

**下面三个选择适合自己的进行依赖!**

```java

// -------------------------------------- 基础版本 --------------------------------------
api 'com.github.xiaojinzi123.Component:component-impl:<version>'
  
// ------------------------------------ RxJava2版本 ------------------------------------
// RxJava2 版本. 需要特别注意：RxJava2 的依赖需要自行在项目中添加依赖. 内部依赖是 compileOnly 的方式
api 'com.github.xiaojinzi123.Component:component-impl-rx:<version>'
api io.reactivex.rxjava2:rxjava:<2.x.x> // 这句记得在自己项目中依赖
  
// -------------------------------------- 协程版本 --------------------------------------
// 协程版本. 需要特别注意：协程的依赖需要自行在项目中添加依赖. 内部依赖是 compileOnly 的方式
api 'com.github.xiaojinzi123.Component:component-impl-ktx:<version>'
api 'org.jetbrains.kotlinx:kotlinx-coroutines-core:x.x.x' // 这句记得在自己项目中依赖
```

## 配置每一个业务组件(包括`app`)

### 配置每一个业务组件的 Host 名称和模块的加载优先级

下面配置的 `HOST` 的值可以是随便的一个名字,你只要保证每一个 `Module` 的名称是唯一的就可以了.<br/>
不一定和 `Module` 名称一样 

```
defaultConfig {
        ......
        javaCompileOptions {
            annotationProcessorOptions {
                // 配置业务模块的模块名称
                arguments += [
                		"HOST"			: project.getName(),
                		"Priority"  : "0", // 这个字符串内必须是数字类型的. 表示模块的优先级, 值越大, 越先加载
                ]
            }
        }
    }
```

### 可选的配置生命周期类

[点我配置,您也可以跳过本步骤](https://github.com/xiaojinzi123/Component/wiki/%E4%B8%9A%E5%8A%A1%E7%BB%84%E4%BB%B6%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)

### 每个业务组件中添加注解驱动器

**请格外注意, 如果你的 build.gradle 配置文件中配置了 apply plugin: 'kotlin-android' 下面的注解驱动器请使用 kapt**

```java
annotationProcessor "com.github.xiaojinzi123.Component:component-compiler:<version>"
// 如果是 kotlin 模块请使用如下的代码
// kapt "com.github.xiaojinzi123.Component:component-compiler:<version>"
// kapt "com.github.xiaojinzi123.Component:component-compiler:<version>"
```

## 壳工程 `App` 的 `Application` 配置如下

特别注意：使用字节码技术加载模块(和 Google 的 App Bundle 不兼容. 如果你使用了 **App Bundle** 请使用下面的反射的方式)

1. 字节码加载方式

```
// 初始化
Component.init(
                BuildConfig.DEBUG,
                Config.with(this)
                      .defaultScheme("router")
                      // 使用内置的路由重复检查的拦截器, 如果为 true, 
                      // 那么当两个相同的路由发生在指定的时间内后一个路由就会被拦截
                      .useRouteRepeatCheckInterceptor(true)
                      // 1000 是默认的, 表示相同路由拦截的时间间隔
                      .routeRepeatCheckDuration(1000)
                      // 是否打印日志提醒你哪些路由使用了 Application 为 Context 进行跳转
                      .tipWhenUseApplication(true)
                      // 这里表示使用 ASM 字节码技术加载模块, 默认是 false
                      // 如果是 true 请务必配套使用 Gradle 插件, 下一步就是可选的配置 Gradle 插件
                      // 如果是 false 请直接略过下一步 Gradle 的配置
                      .optimizeInit(true)
                      // 自动加载所有模块, 打开此开关后下面无需手动注册了
                      // 但是这个依赖 optimizeInit(true) 才会生效
                      .autoRegisterModule(true) // 1.7.9+
                      .build()
);
// 如果你依赖了 rx 版本,需要配置这句代码,否则删除这句
RxErrorIgnoreUtil. ignoreError(); 
// 自动加载所有模块, 此功能需要打开开关 optimizeInit(true). 
// 如果你同时也打开了开关 autoRegisterModule(true), 那么这句代码也可省略了, 因为初始化的时候自动帮你注册了
// 当然你也可以设置 autoRegisterModule(false), 然后自己选择时机去调用自动注册
// ModuleManager.getInstance().autoRegister(); // 1.7.9+
// Debug 的包开启检查
if (BuildConfig.DEBUG) {
      // 框架还带有检查重复的路由和重复的拦截器等功能,在 `debug` 的时候开启它
      ModuleManager.getInstance().check();
}
```

2. 使用反射的方式加载模块

```
// 初始化
Component.init(
                BuildConfig.DEBUG,
                Config.with(this)
                      .defaultScheme("router")
                      // 使用内置的路由重复检查的拦截器, 如果为 true, 
                      // 那么当两个相同的路由发生在指定的时间内后一个路由就会被拦截
                      .useRouteRepeatCheckInterceptor(true)
                      // 1000 是默认的, 表示相同路由拦截的时间间隔
                      .routeRepeatCheckDuration(1000)
                      // 是否打印日志提醒你哪些路由使用了 Application 为 Context 进行跳转
                      .tipWhenUseApplication(true)
                      .build()
);
// 如果你依赖了 rx 版本,需要配置这句代码,否则删除这句
RxErrorIgnoreUtil. ignoreError(); 
// 注册其他业务模块,注册的字符串是上面各个业务模块配置在 build.gradle 中的 HOST
ModuleManager.getInstance().registerArr("component1","component2","user","help");
// 让框架在 Debug 的时候检查.
if (BuildConfig.DEBUG) {
      // 框架还带有检查重复的路由和重复的拦截器等功能,在 `debug` 的时候开启它
      ModuleManager.getInstance().check();
}
```

## 可选的 Gradle 插件

如果你上一步 `optimizeInit(true)` 传入的是 true, 
那么你必须配置此 `Gradle` 插件

在工程级别的 build.gradle 中添加 classpath (java7 或者  java8 选择其一)

```gradle
classpath "com.github.xiaojinzi123.Component:component-plugin:<version>"
```

在壳工程 app 的 builde.gradle 中配置插件 

```
apply plugin: 'com.xiaojinzi.component.plugin'
```

## 混淆配置

```
# 小金子组件化框架 不要警告
-dontwarn com.xiaojinzi.component.**
# 所有本包下面的类和接口都不混淆
-keep class com.xiaojinzi.component.** {*;}
-keep interface com.xiaojinzi.component.** {*;}
#这些是让被标记的类不被混淆
-keep @com.xiaojinzi.component.anno.InterceptorAnno class * {*;}
-keep @com.xiaojinzi.component.anno.GlobalInterceptorAnno class * {*;}
-keep @com.xiaojinzi.component.anno.ConditionalAnno class * {*;}
-keep @com.xiaojinzi.component.anno.FragmentAnno class * {*;}
-keep @com.xiaojinzi.component.anno.ServiceAnno class * {*;}
-keep @com.xiaojinzi.component.anno.support.ComponentGeneratedAnno class * {*;}
-keep @com.xiaojinzi.component.anno.router.RouterApiAnno interface * {*;}
#保留生成的 Router Api
-keep class **.**RouterApiGenerated {*;}
#几个用户自定义或者自动生成到其他包下的应该不混淆
-keep class * implements com.xiaojinzi.component.impl.RouterInterceptor{*;}
-keep class * implements com.xiaojinzi.component.support.IBaseLifecycle{*;}
-keep class * implements com.xiaojinzi.component.application.IApplicationLifecycle{*;}
-keep class * implements com.xiaojinzi.component.service.IServiceLifecycle{*;}
-keep class * implements com.xiaojinzi.component.application.IComponentApplication{*;}
-keep class * implements com.xiaojinzi.component.support.Inject{*;}
```