## 前言

Jetpack Compose ScrollableTabRow 在使用的时候, 会发现无论怎么样, 最小的宽度始终不是自己设置的. 每个 Tab 之间间隔的很大. 而我们的 UI 上的效果是比较紧凑的. 所以这里给出解决方案

## 问题原因

在 ScrollableTabRow 的实现中, 使用了一个叫做 ScrollableTabRowMinimumTabWidth 的值, 他的值是：90.dp 

由于这个值外部没法修改. 导致任何情况下你的 Tab 最小的宽度都是 90.dp

![](https://raw.githubusercontent.com/xiaojinzi123/images/master/202203181459448.png)

![](https://raw.githubusercontent.com/xiaojinzi123/images/master/202203181457619.png)

## 如何解决

目前的解决方式就只有一种, 那就是复制整个 TabRow 的源码. 然后对其中的 ScrollableTabRowMinimumTabWidth 的值进行修改!!!

Google 在不修改源码的情况下也没有其他方式. 所以认命吧. 该死的 Compose. 踩坑达人就是我！