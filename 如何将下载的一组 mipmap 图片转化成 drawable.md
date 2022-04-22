## 干就完了

当我们使用蓝湖 UI 设计的时候, 我发现下载的 png 的一组图都是 mipmap 命名的. 

下载之后. 我需要对五个目录重命名, 并且将内部的文件统一重命名

这个十分浪费我的时间, 所以博主必须解决！！！

项目 [mmtdAndroid](https://github.com/xiaojinzi123/mmtdAndroid) 就是解决这个问题的. 内部帮助你重命名了目录和文件. 使用很方便. 

下图就是我下载一组图. 我们进入图二的命令行, 输入：

mmtdAndroid res_alipay

mmtdAndroid 是命令的名称, 事先配置了环境变量, res_alipay 是文件的新名称



![](https://s2.loli.net/2022/03/23/WJ7poFPUh9Sl4vq.png)![](https://s2.loli.net/2022/03/23/Nm6PjC7GYQSOvge.png)![](https://s2.loli.net/2022/03/23/IZJTMq1k3OvyUaS.png)

运行之后的效果为：

![](https://s2.loli.net/2022/03/23/I13igJGTnxbvYWo.png)![](https://s2.loli.net/2022/03/23/ZTQdDia4Smy6jsG.png)![](https://s2.loli.net/2022/03/23/1dcIBNr5HPn7KbZ.png)

然后我们就可以进入我们项目的 drawable 所在的目录. 复制全部的目录. 进行合并

![](https://s2.loli.net/2022/03/23/7qUFSelTALW8OEN.png)

完美. 比自己一个一个复制进去重命名快多了. Nice!!!

## 源码及其使用说明

https://github.com/xiaojinzi123/mmtdAndroid
