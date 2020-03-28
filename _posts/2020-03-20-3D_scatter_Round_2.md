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

通过代码的缩进 不难看出 这段代码分为两块

前16行代码是第一块代码 这一块儿完成了数据的预处理 后面的代码是绘图函数

先看数据的预处理部分 前9行没什么可说的 构造了一个数据框(R语言的一种数据格式 类似于excel表格数据)

而在10-16行这里 是有关数据分类汇总的一段代码 当初也是迷惑了yool很久才看弄明白
