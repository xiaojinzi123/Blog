## 前言

在我们的 Android App, [Kotlin flows](https://developer.android.com/kotlin/flow) 通常用来收集 UI 层需要展示的那些数据. 但是你在收集数据的时候, 你得确保它不会做很多额外的事情、不会浪费资源、不会因为视图层退到后台或者销毁而引起内存泄漏. 

正因为 Kotlin flows 和 RxJava 都可能有上述的问题, 所以官方的 LiveData 是一个比较好的选择. 但是 LiveData 的局限性比较大, 它缺少了 flows 和 RxJava 的可组合性, 也缺少了很多的好用的链式操作符的支持.

所以本文就是介绍如何利用 Kotlin flows 收集数据, 并且拥有 LiveData 的特性. 也支持各种组合和变换的能力. 

下文会介绍如何使用 **Lifecycle.repeatOnLifecycle** 和 **Flow.flowWithLifecycle** 的 API 来避免资源的浪费. 并且解释它为什么是一种 UI 层数据收集的好方式

## 资源浪费

在平常的编码中, 我们推荐从架构的较低的层面就开始提供 Flow 这种 Api 出来给上层使用. 但是你得保证正确的去收集/订阅他们.

一个由 [channel](https://kotlinlang.org/docs/channels.html) 或者使用操作符(比如：buffer, conflate, flowOn 或者 shareIn)做缓冲的冷流. 当使用一些存在的 Api(比如：CoroutineScope.launch, Flow.launchIn, 或者 LifecycleCoroutineScope.launchWhenX) 去收集是不安全的. 除非你手动的取消 Job, 当 Activity 进入到后台的时候. 这些 Api 会让 flow 底层的数据生产者一直在后台保持活跃, 并且浪费资源 

> Note: 一个冷的 flow 是一种 flow, 它在有新的订阅者收集/订阅的时候执行生产者的代码块来产生数据.

举例一个官方的例子：使用 callbackFlow 来不断发射位置更新的信息

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210604112709.png" />

从 UI 层收集/订阅这个 Flow, 使用上述说的其中一个 Api 去启动, 都会导致位置信息还会不断的更新即使 View 在屏幕上没有展示或者进入了后台.

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210604112729.png" />

为了解决这个问题, 你可能需要手动去取消当你的 View 到了后台. 避免位置信息的提供者一直在发送造成资源浪费. 比如, 你可能需要下面这样做：

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210604112748.png" />

这是一种好的处理方式. 但是这些代码过于模板和不友好了.  对于我们开发者来说, 我们是拒绝写模板代码的. 另外不写样板代码还有一个很大的好处就是. 少写了代码, 就一定可以制造更少的错误!

## Lifecycle.repeatOnLifecycle

现在我们需要来解决这些问题. 需要满足两点：

1. 使用足够简单
2. Api 友好, 并且易于理解和记忆
3. 最重要的一点是安全！它也应该支持任何场景的 Flow, 不管 Flow 的实现细节是怎么样的.

在 lifecycle-runtime-ktx 库中支持了 **Lifecycle.repeatOnLifecycle** 让我们来看一下下面的代码：

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210604112805.png" />

repeatOnLifecycle 是一个 suspend 标记的挂起方法, 他有一个 [Lifecycle.State](https://developer.android.com/reference/android/arch/lifecycle/Lifecycle.State) 参数

当 lifecycle 的状态达到了指定的状态, 它会自动创建并且启动一个新的协程去执行指定的 block. 并且在状态扭转低于所给的状态后自动取消协程.

这样子可以很好的避免写模板代码. 你可以猜得到的是, 这 Api 的需要在 activity 的 `onCreate` 或者  fragment 的 `onViewCreated` 方法中去执行. 这样可以避免产生位置的异常行为. 在 Fragment 中可以这样写. 

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210604112822.png" />

重要提示：Fragment 应该始终使用 viewLifecycleOwner 去触发 UI 更新. 但是这不适用于 DialogFragment. 因为它可能没有 View. 对于 DialogFragment. 你应该使用 lifecycleOwner

## 简单说下原理

repeatOnLifecycle 是一个挂起方法. 当状态达到了指定的值, 内部会启动一个 协程去执行 block 中的代码. 当状态低于给定的状态, 会取消协程从而取消 Flow 的收集/订阅. 因为内部一直需要监测状态来启动和取消. 所以 repeatOnLifecycle 方法下方的代码只有状态到达了 destroyed, 那么才能得到执行. 具体实现代码如下：

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210603192524.png" />

## 用图来描述一下

repeatOnLifecycle 可以防止你浪费资源并且防止 app 崩溃(因为在不合适的 lifecycle 的状态下收到了数据更新的时候)

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210603192927.png" />

## Flow.flowWithLifecycle

你也可以使用 Flow.flowWithLifecycle 操作符当你只有一个 Flow 需要收集/订阅的时候. 这个 API 使用 repeatOnLifecycle 作为底层的实现. 

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210604112853.png" />

## 底层的生产者

就算你使用这些 Api, 如果你收集/订阅到了 Hot Flow, 那么即使你没有人去收集/订阅它, 它还是会浪费资源. 

但是这也是有好处的, 在后续的收集/订阅发生之后, 订阅者总能拿到最新的数据去显示, 而不是拿到老旧的数据. 但是如果你确实不必要它一直保持活跃. 那么这里有一些措施可以防止一下. 

MutableStateFlow 及其 Api 提供了一个 subscriptionCount 的字段, 你可以根据这个字段去判断是否生产数据.

## 和 LiveData 的对比

你可能发现了这些 Api 的行为和 LiveData 真是太像了. 这是事实! LiveData 能感知生命周期, 它的行为是一种理想的方式从 UI 层去订阅数据流. 和 `Lifecycle.repeatOnLifecycle` 和 `Flow.flowWithLifecycle` APIs 类似

使用这些 Api 去代替 LiveData 是 Kotlin 项目独有的. 如果你使用这些 Api 去收集, LiveData 没有任何优势, 相比于协程和 Flow. 毕竟 Flow 可以有更多的组合性和变换性, 还可以从任何一个 Dispatcher 去收集数据. 并且支持很多的操作符去满足各种场景的需求. 反观 LiveData, 只有极少的操作符可用. 并且只能从 UI 线程去订阅.

### data binding 支持了 StateFlow 

另外的, 你使用 LiveData 的一个可能的原因是因为它支持 data binding. 那么现在, StateFlow 它也支持啦. 更多的相关信息可以[查看这里](https://developer.android.com/topic/libraries/data-binding/observability#stateflow)

## 总结

使用 Lifecycle.repeatOnLifecycle 或者 Flow.flowWithLifecycle 的 Api 可更安全的去收集 UI 层想要的数据

看都看完了. 关注一下公众号呗

<img width="200" height="200" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/IMG_8339(20210603-215157).JPG" />

有任何问题, 欢迎留言

[参考链接](https://medium.com/androiddevelopers/a-safer-way-to-collect-flows-from-android-uis-23080b1f8bda)

