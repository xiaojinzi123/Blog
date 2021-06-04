<img width="auto" height="auto" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210531193822.png" />

这么好看的你, 来都来了. 关注我. 了解更多最新知识点, 每天学习一点点

## 前言

自从 Google在 2017 宣布支持 Kotlin, 我们收到了很多 Android 上关于 Kotlin 的问题. 比如：

1. 我应该从什么时间开发学习它
2. 什么是学习 Kotlin 的最好的课程或者文档
3. 是否 Google 内部已经使用了 Kotlin
4. Google 对 Java 语言的计划是什么.

这篇文章中, 我来回答这些问题.

## 问题1：我应该学习 Kotlin 在 Android 中使用吗?

相关的一些问题比如：

1. 作为一个初学者, 我应该如何选择 Kotlin 语言和 Java 语言
2. 作为一个掌握了 java 基础的开发者, 现在开始学习 Kotlin Android 开发是否是正确的.
3. 作为一个资深的 Java 开发, 如果我从事 Android 开发, 你是推荐我使用 Java 还是 Kotlin 开发

简短的回答：

**你应该立马就开始学习和使用 Kotlin**

长回答：

从 2017 年开始推荐使用 Kotlin 开始, 我们第一步就是确保我们的 Api、文档和例子对 Kotlin 是友好的. 2019 年, Kotlin 成为了 Android 开发的首选语言, 我们也渐渐的开始依赖一些 Kotlin 特性. 比如：协程是解决异步工作的一种推荐的方式. 下面是我们做的额外的事情：

### 库会优先支持 Kotlin 

在 Android Jetpack API 中, 我们优先支持了 Kotlin 的协程. 比如 Room, LiveData, ViewModel 和 WorkManager 等等.

再比如 Firebase Android 的 SDK和很多的 Jetpack 的 library 都会使用 Kotlin 的扩展(ktx) 来让编码更加的顺畅.

现在, 很多库比如 Paging 3.0 和 DataStore 都是优先使用 Kotlin 开发. 特别是 [Jetpack Compose](https://developer.android.com/jetpack/compose), 最新开发的一个 UI 工具包, 就是基于 Kotlin 开发的

### 工具

开发生产力来自伟大的工具, 比如：我们对 Kotlin 的编译工具链做了很多的优化, 包括增强 Kotlin JVM 的编译器, 针对 Kotlin 的 R8 优化, 甚至新开发了一个 [注解/符号驱动器](https://github.com/google/ksp).

我们添加了内置的 Android Kotlin 模板(Live templates), 它可以让你快速的添加一些固定结构的代码. 与此同时, 新的 Kotlin Lint 检查工具也会让你的代码更加的符合 Kotlin 的规范. 这是非常有用的, 特别是从 Java 往 Kotlin 过度的阶段. 

## 问题2：是否 Google 内部使用 Kotlin 开发

在 Google 内部, 我们加倍的使用 Kotlin, 超过 60 个我们的 App(比如：Google Home, Drive, Maps...) 已经添加 Kotlin 到项目中. 

我们内部的代码仓库, 超过两百万行代码是 Kotlin

## 问题3：我是否需要迁移我的 App 到 Kotlin

我们经常收到这个问题, 但是这个问题的答案其实取决于你. 如果你喜欢你目前的技术栈, 并且熟练的使用现有的解决方案去解决各种复杂的场景. 比如异步的任务. 并且有高效的方式去发现和解决错误. 那么可能迁移不太适合. 

如果你喜欢通过 Kotlin 尝试一些新特性, 也喜欢使用 Jetpack 最新对 Kotlin 支持的特性. 那么你应该考虑添加 Kotlin 到你的项目. 

毕竟 Kotlin 和现有的 Java 代码可以很好的共存和交互. 你可以一点一点的小范围的使用它, 然后在一些新功能上你可以试着转化一些旧代码到 Kotlin. 

如果你还没有添加 Kotlin 到项目, 你可以参考这个资料：[Converting to Kotlin codelab](https://codelabs.developers.google.com/codelabs/java-to-kotlin#0)

## 问题4：Java 语言对于 Android 来说, 是个什么计划?

目前 Java 和 Kotlin 编译之后都是相同的字节码, 是共存的. 虽然我们喜欢 Kotlin 编码的表现力和安全性. 但是我们依然会维护和发展 Java. 比如在 Android 11 中, 我们添加了很多 OpenJDK 中多个 API, 让你可以使用这些 Api 在所有的设备中. 

## 问题5：学习 Kotlin 最好的方式是什么

采用一个新语言本就不是一个容易的事情, 但是我们努力让它变得尽可能的简单.

1. 通过训练的[课程](https://developer.android.com/kotlin/campaign/learn), 它针对所有水平的开发者. 会提升你在 Android 开发中的技巧.  [Android Basics in Kotlin](https://developer.android.com/courses/android-basics-kotlin/course) 这是另一个线上的课程, 适合没有开发经验的小伙伴. 从基础到教你如何使用[协程](https://codelabs.developers.google.com/codelabs/advanced-kotlin-coroutines/index.html?index=..%2F..index#0)
2. 我们所有的文档页面都支持 Kotlin 的片段. 所以你可以轻松的比对代码是如何工作的. 目前所有的 [samples](http://github.com/android) 都有 Kotlin 的版本.
3. 查看我们的 [文章](http://goo.gle/kotlin-posts) 和 [视频](http://goo.gle/kotlin-videos)可以教你很多 Kotlin 的知识
4. 在[官网](https://developer.android.com/kotlin/learn)的 [Kotlin]([developers.android.com/kotlin](http://developers.android.com/kotlin)) 文章中, 学习如何切换到 Kotlin

## 最后的总结

从官方从2017开始支持 Kotlin, 我们和 JetBrains 一起抓紧支持了这个很棒的语言和系统. 我们会确保 Kotlin 的发展是稳步的. 我们发布的库虽然不仅仅支持 Kotlin, 但却是 Kotlin 优先的. 我们非常坚定会让 Kotlin 在 Android 上的开发是令人愉快的!

关注我, 学习更多小知识. 

<img width="160" height="160" src="https://raw.githubusercontent.com/xiaojinzi123/images/master/20210531193410.png"/>

 