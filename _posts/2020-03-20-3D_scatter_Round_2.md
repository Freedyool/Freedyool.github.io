# Echarts大法好(滑稽)

在yool将他的需求告诉他的小伙伴的时候 他的小伙伴告诉他 不妨试一试用前端的js包来画出想要的图 而且前端的美化功力也是值得肯定的

这里就不得不安利一下由百度前端团队开发的[Echarts](https://www.echartsjs.com/zh/index.html)项目了

此处省略亿点赞美之词

ok 下面正式进入我们的Round2 本文将不再给出完整代码 而是针对其核心代码进行详细说明 以及一些适用场景的介绍

## Round 2 和Echarts相关的绘图解决方案

很明显 Echarts是个前端js包 这对我这种毫无前端开发经验 只会写个html的yool来说 有点超纲

花了小半天的时间 只是得到了一个模糊的逻辑概念 而没有得出一个具体可实现的做法

接着我注意到了一个名为[Recharts](https://madlogos.github.io/recharts/index_cn.html#-en)的项目 这个项目是百度Echarts2的一个R语言接口 不过它的版本十分陈旧 无法满足我需要的功能 在这个项目的文档里 我发现了recharts2的开发情况 点击它给我的链接 发现了一个悲伤的故事

![](/images/2020_03_20/Figure_1.jpg)

不过它依然给我们指明了方向 echarts4r yes'

R语言是什么 建议百度 大致上来说就是一个功能强悍的数据科学软件 我使用的是基于R语言的IDE-[Rstudio](https://rstudio.com/) 确实很好用(感觉和pycharm差不多)

话不多说 [链接给你](https://echarts4r.john-coene.com/index.html)

只要在你的IDE中选择适合你的方式安装即可

在说明文档中找到3Dscatter相关的案例 从案例出发 解析一下它的用法

```R
v <- LETTERS[1:10]
matrix <- data.frame(
  x = sample(v, 300, replace = TRUE), 
  y = sample(v, 300, replace = TRUE), 
  z = rnorm(300, 10, 1),
  color = rnorm(300, 10, 1),
  size = rnorm(300, 10, 1),
  stringsAsFactors = FALSE
) %>% 
  dplyr::group_by(x, y) %>% 
  dplyr::summarise(
    z = sum(z),
    color = sum(color),
    size = sum(size)
  ) %>% 
  dplyr::ungroup() 
  
matrix %>% 
  e_charts(x) %>% 
  e_scatter_3d(y, z, size, color) %>% 
  e_visual_map(
    size,
    inRange = list(symbolSize = c(1, 30)), # scale size
    dimension = 3 # third dimension 0 = x, y = 1, z = 2, size = 3
  ) %>% 
  e_visual_map(
    color,
    inRange = list(color = c('#bf444c', '#d88273', '#f6efa6')), # scale colors
    dimension = 4, # third dimension 0 = x, y = 1, z = 2, size = 3, color = 4
    bottom = 300 # padding to avoid visual maps overlap
  )
```

代码分为数据准备部分和绘图部分 这也是我们脑中需要清楚的 即我们要用什么样的数据传入绘图函数 以及绘图函数的调用方法

数据准备部分使用的是R语言常用的dataframe数据框以及针对数据框的函数包[`dplyr`](https://www.rdocumentation.org/packages/dplyr/versions/0.7.8)

dataframe部分将多个向量拼接成一个名为matrix的数据框 列名包括`x`,`y`,`z`,`color`,`size`五个部分 每一行作为一个空间点的完整信息参数

绘图部分使用了`echarts4r`和`dplyr`中的管道函数`%>%`(一个非常实用的工具) 解读一下绘图模块 

`e_charts(x)`:初始化画布

`e_scatter_3d(y, z, size, color)`:绘图函数

`e_visual_map`:左侧可视化选择工具(可根据点的颜色和大小部分显示)

接下来是一个我的示例 数据集可以在我的gihub项目仓库里找到

```R
library(echarts4r)
# 预定义部分
{
  colormap = c('#0000f6','#01a0f6','#00ecec','#01ff00',
               '#00c800','#019000','#ffff00','#e7c000',
               '#ff9000','#ff0000','#d60000','#c00000',
               '#ff00f0','#780084','#ad90f0')
  center_lat = 38.35
  center_lon = 114.72
}
# 数据载入
tmp = read.csv("./8-12-120906.csv", header = TRUE,stringsAsFactors = FALSE)
# 数据集制作
{
  data <- data.frame(lon= numeric(), lat= numeric(),height= numeric(), dbz=numeric(),color=numeric(),stringsAsFactors=FALSE)

  dat = subset(tmp, dbz!='--')
  rm(tmp)
  dat$dbz = as.numeric(dat$dbz)
  dat = subset(dat, dbz>0)
  # 分级设置点的颜色
  temp = subset(dat, dbz > 70)
  temp$color = rep(15,times=length(temp$dbz))
  data = rbind.data.frame(data,temp)
  for(i in c(1:14)){
    temp = subset(dat, dbz > (5*i-5) & dbz <= (5*i))
    temp$color = rep(i,times=length(temp$dbz))
    data = rbind.data.frame(data,temp)
  }
  rm(dat)
  rm(temp)
 }
 # 数据集整合
  data <- data.frame(
    x = data$lon-center_lon,
    y = data$lat-center_lat,
    z = data$height,
    size = data$height,
    color = data$color,
    stringsAsFactors=FALSE
  )
 # 绘制图像
 data %>%
    e_charts(x) %>%
    e_scatter_3d(y, z, size, color) %>%
    e_visual_map(
      min = 0.15, max = 16,
      inRange = list(symbolSize = c(3, 5)),
      dimension = 2
    ) %>%
    e_visual_map(
      min = 1, max = 15,
      inRange = list(color = colormap), 
      dimension = 4,
      bottom = 300
    )
```

OK 关于如何用echarts4r绘制巨量三维散点图就说到这里了 有一些不得不说的东西 就是没有什么语言或者包是完美的 总会有不足 相比于matplotlib 基于百度echarts的echarts4r显然在性能和视觉效果上提高了一个层次 但是也有图层穿模 无法支持同时渲染更大批量数据 以及视角交互能力有较多局限性这样的问题 此外添加地图插件似乎也会遇到一些问题 本人也尚未解决

有机会的话 我会讲讲如何在python环境下实现echarts的调用 希望能解决上述提及的部分问题 不过大概率不会有很大的改进 毕竟都是基于echarts的包 所以我也是比较期待是否有更多功能更为强悍的可视化工具 欢迎评论区告知我
