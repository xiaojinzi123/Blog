### 前言

**Android** 中的 **ViewModel** 出来已经有不少的时间了. 我作为一个忠实的使用者. 从开始用它, 到用错它, 到理解它. 还是走了不少冤枉路. 所以本次的内容会围绕 **ViewModel** 的正确使用. 并且如何融入到 **MVVM** 架构中

### 官方 **ViewModel** 的作用解释

[Google ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel) 的解释. 如果喜欢看官方的, 可以点链接去看. 这里我做一个总结：

1. **ViewModel** 可以让你分离你 UI 上的数据. 比如某个视图的是否可见、比如 Adapter 的数据、比如某个 **TextView** 的内容等等
2. **ViewModel** 避免 **Activity** 或者 **Fragment** 加载数据的异步调用导致的内存泄漏.
3. 可以在 **Activity** 及其挂载的所有 **Fragment** 之间共享数据
4. 可以在 **Activity** 发生旋转屏幕导致的重建后还能不销毁

以上就是 **ViewModel** 的功能. 那其实简单一点说就是. 

**ViewModel** 可以帮你存储所有 **UI** 上显示的数据. 并且和外界的数据交互都可以我来干.

### ViewModel 在架构中的角色

有两种可能

- **ViewModel** 只是视图层的一部分. 它负责视图层的 UI 数据的持有. 具体的业务代码由下一层实现. 
  - **View** <==> **ViewModel** <==> 业务层 <==> 数据层(比如数据库、网络请求等等、SP)
- **VIewModel** 不是视图层的一部分, 它负责业务代码的实现.
  - **View** <==> **ViewModel** <==> 数据层(比如数据库、网络请求等等、SP)

在现在, 很多公司或者小伙伴都基本使用了第二种, 当成了业务层来使用. 我之前也是直接当成业务层来使用了. 

随着时间的推移, 我发现第二种的方式, 它好像是错误的！原因有以下几点：

- 一般业务层可能会互相创建或者引用. **ViewModel** 的创建是系统帮忙创建的, 没法在一个 **ViewModel** 中去用另一个 **ViewModel**
- 不在 **Fragment** 或者 **Activity** 的地方, 没法使用 **ViewModel**, 如果要使用, 发现需要自己 **new** 一个
- 如果 **ViewModel** 我们自己创建, 那么就失去了 **ViewModel** 的特性

综上所述, **ViewModel** 不是一个写业务代码的地方. 所以上面的第一种方式是正确的! **ViewModel** 并不是业务层

**View** <==> **ViewModel** <==> 业务层 <==> 数据层(比如数据库、网络请求等等、SP)

但是这个方式有一个比较大的弊端：

如果我要享受 **ViewModel** 的特性, 那么所有的业务方法都得通过 **ViewModel** 转发到业务层. 比如 **View** 层触发一个登录操作. 你会发现 **ViewModel** 写的特别多的业务的代理方法. 虽然不实现业务逻辑. 但是写起来非常的冗余. 我相信没有几个人会喜欢这种写法的. 虽然它是对的一种架构

那么有没有办法既能享受到 **ViewModel** 的特性, **ViewModel** 又不会写很多的冗余的业务代理方法. 答案是有的. 使用 **Kotlin** 的委托

比如写一个业务类. 

```kotlin
interface MainUseCase{
    fun login()
}

class MainUseCaseImpl: MainUseCase {
    override fun login() {
        // 实现登录
    }
}
```

我写一个 **ViewModel**

```kotlin
class MainViewModel : ViewModel(), MainUseCase by MainUseCaseImpl() {
}
```

在 **ViewModel** 中我没写一句代码, 但是 **ViewModel** 已经拥有业务的方法了. 利用这种方式. 如果你的架构是 MVP, 那么你的 P 就是这里的 UseCase, 

那么你的 P 就能在视图层的各个 Fragment 中共享, 并且可以在 Activity 重建后还依然存在. 

如果你的架构是 **MVVM**, 对应到这里, 就是 **MainViewModel** 可以取订阅各种业务层 **UseCase** 的数据. 

通过组合或者变换来转化成 UI 上的数据. 用 **LiveData** 去装载. **ViewModel** 融入到 **MVVM** 架构中, 一个很好的官方项目是 [Google IO](https://github.com/google/iosched)

### 总结

这篇文章, 我的目的很简单, 就是说明 **ViewModel** 并不是直接写业务代码的, 正确的基础架构应该是 

**View** <==> **ViewModel** <==> 业务层 <==> 数据层(比如数据库、网络请求等等、SP)

但是由于此架构所有的业务方法都得 **ViewModel** 来代理, 所以分享一下如何简化这个代理. 就是使用 **Kotlin** 的委托. 











