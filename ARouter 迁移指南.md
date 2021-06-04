### 前言

由于 **ARouter** 过来的小伙伴会比较多, 特别提供这个迁移指南. 本以为使用了 **ARouter** 一段时间的用户会对路由框架比较了解. 可能我忘了用户是使用者, 不需要了解过多. 

### 配置迁移

[AndroidX 配置](https://github.com/xiaojinzi123/Component/wiki/%E4%BE%9D%E8%B5%96%E5%92%8C%E9%85%8D%E7%BD%AE-AndroidX)

[非 AndroidX配置](https://github.com/xiaojinzi123/Component/wiki/%E4%BE%9D%E8%B5%96%E5%92%8C%E9%85%8D%E7%BD%AE)

**配置上就主要看上面两个链接上的叙述, 这里再着重说一些容易错误或者不理解的地方**

配置迁移的时候. 主要是两点：

- 初始化

  - 初始化方面的下面会侧重的说明.

- 模块的配置

  - **ARouter** 有 **Gradle** 插件, 它是为了字节码修改代码加载模块用的. **Component** 提供字节码的方式和反射的方式. 如果你想要字节码的方式, 那么也请配置好 **Component** 的 [Gradle 插件](https://github.com/xiaojinzi123/Component/wiki/%E4%BE%9D%E8%B5%96%E5%92%8C%E9%85%8D%E7%BD%AE-AndroidX#%E5%8F%AF%E9%80%89%E7%9A%84-gradle-%E6%8F%92%E4%BB%B6), 如果你不想用字节码的方式, 那么你迁移的时候, 直接去掉有关 **ARouter** 的 **Gradle** 插件即可. **Component** 的 Gradle 插件也不需要配置了. 但是后续的初始化的时候也就没法用自动加载模块的功能了. 需要自己加载模块. 比如：

    ```java
    ModuleManager.getInstance().registerArr("module1", "module2",....);
    ```

  - 每一个业务模块你都需要配置[注解驱动器](https://github.com/xiaojinzi123/Component/wiki/%E4%BE%9D%E8%B5%96%E5%92%8C%E9%85%8D%E7%BD%AE-AndroidX#%E9%85%8D%E7%BD%AE%E6%AF%8F%E4%B8%80%E4%B8%AA%E4%B8%9A%E5%8A%A1%E7%BB%84%E4%BB%B6%E5%8C%85%E6%8B%ACapp). 特别注意, **kapt** 和 **annotationProcessor** 的使用. 如果你的模块是纯 **Java** 模块, 那么是 **annotationProcessor**, 否则就是 **kapt**

**ARouter** 没有提供主动加载模块的方法, 而 **Component** 提供了主动加载模块的方法. 比如在 **App** 中写了如下代码：

```java
Component.init(
                BuildConfig.DEBUG,
                Config.with(this)
                        .defaultScheme("router")
                        // ..........
                        .build()
        );
// 模块的名称需要和每一个 Module 中配置的 host 的值一致哦
ModuleManager.getInstance().registerArr("module1", "module2",....);
```

**Component** 同时也提供了自动注册的功能. 

自动注册功能, 需要依赖 **ASM** 字节码技术, 所以需要你正确配置 **Gradle** 插件, 并且下面的初始化中必须开启 **optimizeInit(true)**

- 方式1：初始化的时候, 传入自动注册的参数, 让框架去加载

  ```java
  Component.init(
                  BuildConfig.DEBUG,
                  Config.with(this)
                          .defaultScheme("router")
                          // ..........
                          .optimizeInit(true)
                          // 自动加载所有模块, 依赖上面的 optimizeInit(true)
                          .autoRegisterModule(true)
                          .build()
          );
  ```

- 方式2：初始化中不指定 autoRegisterModule(true), 自己调用加载的方法

  ```java
  Component.init(
                  BuildConfig.DEBUG,
                  Config.with(this)
                          .defaultScheme("router")
                          // ..........
                          .optimizeInit(true)
                          .build()
          );
  ModuleManager.getInstance().autoRegister();
  ```

如果你依赖的是 **RxJava** 的版本, 即 **-rx** 的包, 那么你还需要调用如下的代码：

```java
// 因为 RxJava 需要你实现 onError 的回调方法, 如果你不实现, 会造成崩溃, 
// 而这行代码的作用就是可以让你使用 RxJava 方法跳转的同时也不用实现 onError 回调
RxErrorIgnoreUtil.ignoreError();
```

### 模块的生命周期

**Component** 有[模块的生命周期](https://github.com/xiaojinzi123/Component/wiki/%E4%B8%9A%E5%8A%A1%E7%BB%84%E4%BB%B6%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)的概念.  这样可以让某一个模块的独有的一些 **Lib** 的初始化可以当到所属的模块. 真正的做到隔离.

下面是使用的一个例子. 你可以声明到任何的地方, 并且支持创建多个.

```java
@ModuleAppAnno()
public class Component1Application implements IApplicationLifecycle { // 老版是 IComponentApplication 接口

    @NonNull
    privat Application mApp;

    @Override
    public void onCreate(@NonNull final Application app) {
        mApp = app;
        // 你可以做一些当前业务模块的一些初始化
    }

    @Override
    public void onDestory() {
        // 你可以销毁有关当前业务模块的东西
    }
}
```

模块的生命周期, 一般我们会用到 **onCreate** 方法, 因为我们不会在 App 运行的时候去卸载一个模块. 虽然 **Component** 支持你任何时候去卸载. 但是坐着强烈建议你不要那么去做. 因为这样可能会出现未知的异常.

假如你想在运行时候下架某一个模块. 不要在运行的时候去做. 你可以在下次启动的时候. 不加载你想去掉的模块即可. 这样可保证你的 **app** 运行稳定没有 **bug**

### 跳转迁移

**ARouter** 的路由包含

- 路由 **Activity** 、
- 所谓的路由 **Fragment**
- 服务发现

实际上, 路由 **Fragment** 只是服务发现的一种便捷的使用方式. 因为它并不是帮你启动了一个 **Fragment**, 而是获取到一个 **Fragment**

所以 **ARouter** 的路由这个概念, 包含了上面的三个方面. 

#### 路由 Activity 迁移

**ARouter** 的跳转的核心参数是 **path**, **Component** 包含两个部分, **host** 和 **path**

标记目标界面, 参考：[RouterAnno的使用](https://github.com/xiaojinzi123/Component/wiki/RouterAnno-%E6%B3%A8%E8%A7%A3)

简单来说就是看到标记到 **Activity** 界面的 **@Route(path = "/xxx/xxx")**,  需要替换成 **@RouterAnno(hostAndPath = "/xxx/xxx")**

>  注意事项

- 值得注意的是 **Component** 中的 **hostAndPath** 属性是表示同时配置 **host**  和 **path**, 必须包含至少一个 / 分隔符, 如果没有分隔符, 请使用 **path** 属性, 那么 **host** 自动会采用模块配置的 **host** 
- **Component** 的路由仅支持系统支持的数据类型, 不会多支持. 因为 **Component** 的其中一个设计理念就是尽可能的贴近系统. 这也方便目标界面可以使用原生的参数获取方式, 不用做任何的更改. 也方便后续的维护, **ARouter** 是支持任意的数据类型的, 这点在 **Component** 中是被废弃的. 不支持的

**ARouter** 只支持标记应用内的界面. 而 **Component** 支持标记系统的、第三方的、和应用内的. 具体看: [此处](https://github.com/xiaojinzi123/Component/wiki/RouterAnno-%E6%B3%A8%E8%A7%A3#%E6%A0%87%E8%AE%B0%E5%9C%A8%E9%9D%99%E6%80%81%E6%96%B9%E6%B3%95%E4%B8%8A%E6%96%B9%E6%B3%95%E5%85%81%E8%AE%B8%E6%8A%9B%E5%87%BA%E4%BB%BB%E4%BD%95-exception)

替换 **ARouter** 的 跳转代码参考： [代码跳转](https://github.com/xiaojinzi123/Component/wiki/%E8%B7%B3%E8%BD%AC-%E4%BD%BF%E7%94%A8%E4%BB%A3%E7%A0%81%E8%B7%B3%E8%BD%AC)

**Component** 还支持类似于 **Retrofit** 的声明式的接口. [Wiki 文章](https://github.com/xiaojinzi123/Component/wiki/%E8%B7%B3%E8%BD%AC-%E6%8E%A5%E5%8F%A3%E8%B7%AF%E7%94%B1%E7%9A%84%E6%96%B9%E5%BC%8F)

举个例子：

```java
@RouterApiAnno()
public interface App {
      // 声明一个接口描述跳转到登录界面
      // @HostAndPathAnno 等同于 @HostAnno + @PathAnno
      @HostAndPathAnno("user/login")
      void toLoginView(Context context);
}
```

```java
// 执行跳转
Router.withApi(App.class).toLoginView(this);
```

这种方式适合那种某些界面的跳转入口太多. 参数太杂. 那么用声明式的跳转会让你的传递参数变得很带劲

**ARouter** 的自动注入, 使用的注解 **@Autowired**, 在 **Component** 中, 你需要区别属性的注入和 **Service** 的注入

**Service** 的注入需要使用 **@ServiceAutowiredAnno**, 属性的注入需要使用 **@AttrValueAutowiredAnno**

属性的注入的注解 **@AttrValueAutowiredAnno** 必须指注入的名称, **ARouter** 是默认使用参数的名字. 但是其实这样子虽然方便了, 但是你改了变量名是不会有任何的错误提示的, 及其容易造成后续维护的问题. 所以这也就是 **Component** 强制要求你写注入的名称的原因. 

属性的注入, **Component** 还支持注入多个属性, 框架会有顺序的注入, 所以这个特性方便你后续界面的传值的 **key** 改了之后还能支持老的. 

最后别忘了, 需要像 **ARouter** 一样, 调用 

```java
Component.inject(this);
```

#### "路由" Fragment 迁移

**Component** 提供了类似的方式: 

```kotlin
// 这是一个同步的方法, 即时返回对应的 Fragment
val fragment = Router
                .with(ModuleConfig.Module1.TEST_FRAGMENT)
                .navigate()
```

如何标记你的 **Fragment**, 还请移步[此处](https://github.com/xiaojinzi123/Component/wiki/%E8%B7%B3%E8%BD%AC-Fragment)

#### 服务发现迁移

服务发现在绝大多数的框架中表现就是 接口 + 实现类. 任何一个业务模块都可拿到接口 **class**, 但是访问不到实现类, 所以需要服务发现功能来帮助用户找到实现类, 返回接口的对象供使用者使用

**ARouter** 中的服务发现大概是下面的用法：

```java
// 服务发现帮你获取到 ISettingRouter 接口的实现对象
ISettingRouter executor = ARouter.getInstance().navigation(ISettingRouter.class);
```

**ISettingRouter** 的实现类也需要用路由的标记. 

而 **Component** 的服务发现彻底的脱离路由的范畴, 两者没有任何的关系

**Component** 使用 **@ServiceAnno** 标记实现类, 比如:

```java
@ServiceAnno(TestInter.class)
public class TestInterImpl implements TestInter {
	// .....
}
```

使用如下：

```java
TestInter tsetInter = ServiceManager.get(TestInter.class);
```

#### 路由概念的区别

**ARouter** 的路由概念不仅仅是你理解的跳转的那个路由. 它包含：跳转 **Activity**、获取 **Fragment**、获取接口实现

而 **Component** 的路由仅仅表示跳转 **Activity**, 但是跳转这块强大许多, 比如：

- 支持标记任何一个第三方或者系统的 **Intent**

- 支持自动取消(**Activity** 或者 **Fragment** 销毁)
- 支持自动取消路由
- 支持回调中直接拿到 **ActivityResult**
- 支持接口方式的声明式跳转
- 支持[页面拦截器](https://github.com/xiaojinzi123/Component/wiki/%E6%8B%A6%E6%88%AA%E5%99%A8)(后续还会解释这个的)

### 拦截器

**ARouter** 中的拦截器只有一种全局拦截器

**Component** 有三种维度的拦截器. 

- 单次跳转使用的拦截器

  ```java
  Router
  .with(activity)
  .hostAndPath("xxx/xxx")
  // 指定本次跳转的拦截器
  .interceptors()
  .forward();
  ```

- 全局拦截器(使用特定注解标记一个拦截器即可)

  ```java
  @GlobalInterceptorAnno(priority = 1000) // 可以指定优先级, 数字越大优先级越高
  public class WebViewInterceptor implements RouterInterceptor {
  	// ......
  }
  ```

- 页面拦截器

  - 经过所有拦截器后, 如果目标 **Intent** 上有标记拦截器, 这些拦截器就会被加载起来执行. 执行完毕再进行跳转
  - 换句话说就是此种拦截器和目标界面是一起的. 
  - 比如你将一个登录的拦截器配置到 A Activity, 那么 A Activity 就有了一个特性是, 所有跳转到 A Activity 的路由都会先完成登录的功能才会进来
  - 所以页面拦截器通常用来完成一些进入界面前的预处理工作. 十分的有用哦

  ```java
  // 只需这样子标记, 此界面就拥有了自动完成登录的功能啦. 当然了, 登录的功能是别名为 ”user.login” 的拦截器完成的. 
  // 至于登录拦截器
  @RouterAnno(
          path = "user/personCenter",
          interceptorNames = "user.login",
          desc = "用户个人中心界面"
  )
  public class PersonCenterAct extends AppCompatActivity {
  	// .......
  }
  ```

  

