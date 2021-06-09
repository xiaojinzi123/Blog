<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210531193822.png" />

## 前言

你的项目中, 是否写了很多的工具类? 比如 StringUtils, SPUtils, SystemUtils 等等

那么此篇文章, 我来介绍一个关键词：inline, 我翻译为内联. 那么让我们来看一下内联函数的实现原理及其使用它的注意点

## 普通扩展函数

这里写了一个 SharedPreferences 编辑的扩展函数

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210609113913.png" />

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210609114004.png" />

那么让我们看看底层发生了什么. 你可以通过 (Tools > Kotlin > Decompiled Kotlin to Java) 操作, 可以看到生成的 Java 代码是什么样的.

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210609114015.png" />

我们可以看到其实扩展方法也是生成了一个我们平常写的一个工具方法. 然后用户使用的地方的代码, 被翻译成了一个需要创建一个 Function1 来执行用户业务逻辑的代码. 可以看到原理还是很简单的.

## 内联函数

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210609114036.png" />

如果我们的代码加了一个 inline, 那么, 再让我们看看生成的 java 代码是怎么样的

函数定义处没有啥变化. 但是使用的地方, 发生了很大的变化

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210609114046.png" />

因为 inline 这个关键字的原因. 使用该方法的地方, 编译器都会复制方法内的实现到使用的地方. 

## 内联函数的注意点

#### 内联函数的代码行数

因为内联函数会让使用处的代码膨胀. 所以官方极力建议, 内联函数的代码不建议超过 3 行. 控制在 1~3 行代码是最佳的. 

#### 你不能传递函数的引用给非内联函数

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210609114058.png" />

你会得到一个错误信息:

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210609110744.png" />

其实也很容易想明白. 内联函数的代码会被生成到使用的地方. 

我们在最上面不使用内联的时候, 我们可以看到系统生成一个 Function1 的内部类. 

<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210609114116.png" />

如果函数内联了, 那么这个 Function1 的内部类也没有了. 那么后续你用于传递的 Function1 参数自然也就不存在了. 所以报错了.

你有两种解决方式：

1. 内联函数引用的函数也标记为内联.

   <img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210609114135.png" />
   
2. 内联函数的参数使用 **noinline** 标记. 让编译器强制生成 Function1 的内部类.

   <img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210609114147.png" />
   
   ## 总结
   
   原文链接：[https://medium.com/androiddevelopers/inline-functions-under-the-hood-12ddcc0b3a56](https://medium.com/androiddevelopers/inline-functions-under-the-hood-12ddcc0b3a56)
   
   本文中, 介绍了 inline 和 noinline 的用法. 也介绍了 inline 的原理. 同时也介绍了 inline 使用的时候的注意点. 
   
   看都看完了, 如果你喜欢我的文章, 可以订阅我的公众号哦
   
<img width="300" height="300" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/IMG_8339(20210603-215157).JPG" />