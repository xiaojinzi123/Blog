<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210604155412.png" />

## 前言

Flow 的 shareIn 和 stateIn 操作符可以转化一个 Cold Flow 到 Hot Flow：它可以把从上游 Cold Flow 中收到的数据广播给所有的订阅者(collectors/subscriber). 它通常用来提升性能, 甚至内部有缓存机制.

知识点普及：Cold Flow 一被订阅或者被观察的时候, 就会产生数据. 通常订阅者可以观察到所有的数据. 而 Hot Flow 是不管有没有订阅者订阅, 它都保持活跃, 并且可能发射数据.

在这个博客中, 你可以通过例子熟悉 shareIn 和 stateIn 操作符的用法. 你会学到如何配置它们去满足一些使用场景, 也会学到如何避免常见的陷阱.

## 底层数据生产者

继续之前[文章的例子](https://blog.csdn.net/u011692041/article/details/117552958). 底层的 Flow 生产者是发射定位更新数据的. 它是一个 Cold Flow, 用 callbackFlow 实现. 每一个订阅者来订阅的时候, 否会触发 Flow 生产者的 block, 从而导致每次都会产生一个新的 callback 添加到 FusedLocationProviderClient 中

<img  width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210604161514.png" />

那让我们看看如何使用 shareIn 和 stateIn 操作符去优化上面的代码去满足不同的场景.

## shareIn or stateIn?

先说明一个知识点. shareIn 和 stateIn 是不一样的. stareIn 操作符会返回一个 sharedFlow, 而 stateIn 会返回一个 StateFlow.

详细信息可以查看这里的文档: https://developer.android.com/kotlin/flow/stateflow-and-sharedflow

StateFlow 是一个特别的 SharedFlow. StateFlow 是 SharedFlow 派生类, StateFlow 的行为和 LiveData 的行为是类似的. 会发射最后一个状态给订阅者.

两者最大的区别是, StateFlow 允许你访问最后发射的状态. SharedFlow 是不可以的.

如果熟悉 LiveData 或者 RxJava 的 BehaviorSubhect, 你会瞬间明白 StateFlow 的行为. 

## 提升性能

这些 Api 可以通过共享同一个 Flow 实例来提升性能. 当订阅者来订阅的, 不会多次创建 flow, 也不会和上述的情况一样每次都会让数据生产者每次执行内部的 block.

下面的例子中, LocationRepository 消费了 locationsSource 的 flow, 转化成了一个共享的 Flow 对象, 以便订阅者可以订阅同一个 Flow. 

<img  width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210604163326.png" />

WhileSubscribed 策略是在没有订阅者的时候, 会取消上游的 Flow, 避免浪费资源. 但是有时候视图层可能发生一些系统行为导致的配置发生变化. 这里就会触发取消和重新订阅上游的 Flow. 所以官方十分推荐使用 **WhileSubscribed(5000)** 的策略. 表示 5 秒内如果都没有订阅者, 就取消, 这很好的避免了因为配置变化导致的资源浪费.

# 事件缓存

这个例子中. 我们的需求发生了变化. 有两点要求：

1. 我们希望订阅者能一直收到位置的更新
2. 新的订阅者能收到最后的 10 个位置信息数据. 

 width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210604164953.png" />

我们可以使用 replay 参数, 设置 10 来缓存最后 10 个位置数据. 当订阅者来订阅了, 就会收到这十个值. 为了让底层的 Flow 生产者一直保持活跃. 我们可以使用 SharingStarted.Eagerly 策略. 即使没有任何的订阅者.

## 数据缓存

现在新的需求是, 我们只需要当订阅者订阅的时候, 能收到最后一个数据. 这时候我们可以使用 StateIn 操作符.
<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210604174527.png" />
EmptyLocation 是传的一个默认值. 因为 StateFlow 需要一个默认值. 

## 需要避免的问题

千万千万不要在一个方法中去使用 shareIn 或者 stateIn 去创建一个新的 Flow. 因为这会导致你创建无数个 Hot Flow, 在对应的 scope 取消的时候才能被销毁.  错误示范如下:
<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210604174712.png" />

## 总结
shareIn 和 stateIn 操作符可以把 Cold Flow 转化为 Hot Flow, 从而提升性能. 但是使用上要特别注意, 不要在多次执行的方法或者代码块中使用 stateIn 和 shareIn 操作符去创建 Hot Flow. 这样会导致严重的资源浪费.

看都看完了, 关注一下我的公众号吧
<img align="left" width="300" height="300" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/IMG_8339(20210603-215157).JPG" />