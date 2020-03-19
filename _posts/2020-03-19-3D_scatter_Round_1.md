# 科技感十足的3D散点图绘制 

### 上万个散点绘制在同一坐标系下

## 密恐慎入！！！

最近呢 Yool得到了一些气象雷达相关的数据集(晚些时候我会放一些在我的github仓库里 欢迎品尝)

然后呢 就一直在想怎么把那些数字给具象化地展示出来 这不 在经过仨天仨夜的艰苦奋斗之下 终于初尝成果

赶紧就来分享给小伙伴们来了(干货在第二篇内容中)

因为是处理大宗数据 那么就不得不考虑到各种语言 的数据处理能力 同时呢也要考虑到画面的渲染性能 所以python是一个很好的选择（人库资源多 还新 尤其数据处理的便捷性 可靠性 没得说）

ok！ 

### Round 1 python 3.7 + matplotlib 3.2.1
s
按照惯例 先上结果图

![]{./images/2020_03_19/Figure_1.png}
![]{./images/2020_03_19/Figure_2.png}

因为这里不能调取可视化工具 所以看不了它实际的3D旋转和放缩的效果 实际上它是可以旋转放缩滴(你问我这张图有啥用 emm 没什么用 可能可以用于进一步理解python的matplotlib吧 手动狗头)

选择matplotlib的原因呢 很简单 很简单 因为我用过 而且安装起来很方便 用起来也很方便 而且呢 这是一个已经更新维护了十多年近20年的库 拥有不可比拟的健壮性 要想调用函数实现我们的作图 首先要考虑的问题 就是 函数需要什么样的数据 我们拥有什么样的数据 这两者之间需要一个协调的过程 

emm先要找到能够满足我们需要的一个绘图函数-plot3d-scatter3d

通过查阅matplotlib的官方文档中的[scatter3d]{https://matplotlib.org/gallery/mplot3d/scatter3d.html#sphx-glr-gallery-mplot3d-scatter3d-py}

得到一个官方样例

```python
import matplotlib.pyplot as plt
import numpy as np

# Fixing random state for reproducibility
np.random.seed(19680801)


def randrange(n, vmin, vmax):
    '''
    Helper function to make an array of random numbers having shape (n, )
    with each number distributed Uniform(vmin, vmax).
    '''
    return (vmax - vmin)*np.random.rand(n) + vmin

fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')

n = 100

# For each set of style and range settings, plot n random points in the box
# defined by x in [23, 32], y in [0, 100], z in [zlow, zhigh].
for m, zlow, zhigh in [('o', -50, -25), ('^', -30, -5)]:
    xs = randrange(n, 23, 32)
    ys = randrange(n, 0, 100)
    zs = randrange(n, zlow, zhigh)
    ax.scatter(xs, ys, zs, marker=m)

ax.set_xlabel('X Label')
ax.set_ylabel('Y Label')
ax.set_zlabel('Z Label')

plt.show()
```

简单解析一下这段代码：

*第31行到第40行的内容 就是在写一个生成伪随机数的方法 而我们用到的数据呢是已经确定的数据集 因此这段代码直接删掉*

*从第42行开始 才是真正的绘图过程 和matplotlib其它绘图函数基本一致 首先调用`plot.figure`函数 生成一块画布 然后在这块画布上添加一个`projection='3d'`的绘图 `111`是标定绘图位置的参数 它的三个1的含义依次是 我要在画布上添加**1**张图 这张图在画布上的相对位置是 第**1**行第**1**列的位置*

*接下来的n是声明散点的个数 (我们不需要)*

*最后是把点画到图上去 这里的for循环很有意思 它的涵义是 定义了两种格点(('0',-50,-25)和('^',-30,-5)) 一个是圆形一个是三角形 是不是很形象 不过这都和我们的数据无关 我们只要圆的 后面的那俩参数是三维坐标里面z坐标的取值上下限*

*ok 也就是说 这个for循环其实只是循环了两次 一次把原点打出来 一次把三角形的点打出来 最后面四行代码 应该就完全没啥可说的了*

分析完这些的话 我们可以得到以下几点信息:

1. 我们调用的绘图函数是`ax.scatter()`

2. 需要我们提前准备的必要数据元素有:x,y,z 

ok 听着就很简单 实际上也确实很简单 这和普通的二维plot散点图有啥区别 没有区别 只是多了一个维度 多了一个你可以旋转放大的功能(hhh)

听上去 我们的工作好像差不多了 不过 python的库版本匹配问题从来不会让你失望 是的 你会报错 而且 不止一种报错

先把我最后的实现代码给大伙康康 

![]{./images/2020_03_19/code.jpg}

第一个是在import的时候 需要添加一条

    from mpl_toolkits.mplot3d import Axes3D

虽然 执行之前 编辑器会把它当成未使用状态 但是没有这行代码 我的python3.7.3 + matplotlib3.2.1就没法运行

第二个问题是 GUI的选择matplotlib默认的绘图GUI呢是agg 但是它很有可能不能和你的电脑很好的适配 有时呢它也会和你调用的方法不适配 因此 需要去选择一款你不会报错的GUI选项 通过`matplotlib.use('')`即可

第三个问题是 如果我想给我的点做一个颜色的区分 我的解决方案是 分批次绘制点 这个方法也许看上去很捞 但是真的有效 但是在这里需要注意一下 当你在为你的scatter函数设置color这个参数的时候 需要构造一个与你的数据点个数相等的一维数组(注意 数组元素是char类型哦)

而关于如何构造它 看代码就知道了

emmm 关于我所使用的数据部分 因为是其实是项目内甲方给的数据 暂时还没有获得甲方爸爸的许可 等晚些时候去说一下 应该就可以发一些在我的git仓库里给大伙儿玩玩儿了

ok 这样我们就能做出一个看上去还不错的 基于python-matplotlib的三维数据可视化 但是如果你和我一样去做了的话 不难发现 一个很大的问题就是 它太卡了 而且 功能较为单一 而且 我的结果也没办法保存下来 这在我们后续研究它的时候是一个巨大的不幸 那么如何改进呢 看第二篇 嘿嘿~
