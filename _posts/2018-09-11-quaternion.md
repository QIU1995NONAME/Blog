---
layout: docs
title:  "四元数与旋转"
date:   2018-09-11
categories: Math Quaternion
enable_mathjax: true
enable_mermaid: false
enable_comments: true
---

所有的引用均来自<https://en.wikipedia.org/wiki/Quaternion>以及其相关联的页面，所以不再列出详细引用。

# Quaternion (四元数) 与旋转
如果一句话介绍四元数，那么它就是一个活在四次元，一个很难被想明白的玩意儿。

## 四元数基本定义(主要来源于[维基百科](https://en.wikipedia.org/wiki/Quaternion))

$$ \boldsymbol{q} = a + 
b\ \boldsymbol{\vec{i}} + 
c\ \boldsymbol{\vec{j}} + 
d\ \boldsymbol{\vec{k}}\ ,\ 
\boldsymbol{q} \in \boldsymbol{H}
$$

其中 $$ a, b, c, d $$ 是实数，$$ 
\boldsymbol{\vec{i}}, 
\boldsymbol{\vec{j}}, 
\boldsymbol{\vec{k}} 
$$ 是三个虚数单位。这三个虚数单位两两垂直，且遵从右手螺旋定则：

0. $$ 
\boldsymbol{\vec{i}}^2 = 
\boldsymbol{\vec{j}}^2 = 
\boldsymbol{\vec{k}}^2 = 
\boldsymbol{\vec{i}}\ \boldsymbol{\vec{j}}\ \boldsymbol{\vec{k}} = 
-1$$
0. $$ 
\boldsymbol{\vec{i}}\ \boldsymbol{\vec{j}} = \boldsymbol{\vec{k}}\ ,\ \ \ 
\boldsymbol{\vec{j}}\ \boldsymbol{\vec{k}} = \boldsymbol{\vec{i}}\ ,\ \ \ 
\boldsymbol{\vec{k}}\ \boldsymbol{\vec{i}} = \boldsymbol{\vec{j}}
$$
0. $$ 
\boldsymbol{\vec{j}}\ \boldsymbol{\vec{i}} = -\boldsymbol{\vec{k}}\ ,\ \ \ 
\boldsymbol{\vec{k}}\ \boldsymbol{\vec{j}} = -\boldsymbol{\vec{i}}\ ,\ \ \ 
\boldsymbol{\vec{i}}\ \boldsymbol{\vec{k}} = -\boldsymbol{\vec{j}}
$$

另一种四元数的表示方法是：

$$ \boldsymbol{q} = (r,\ \boldsymbol{\vec{v}})\ ,\ 
\boldsymbol{q} \in \boldsymbol{H}\ ,\ 
r \in \boldsymbol{R}\ ,\ 
\boldsymbol{\vec{v}} \in \boldsymbol{R}^3
$$

其他基本概念请参阅[维基百科](https://en.wikipedia.org/wiki/Quaternion)，
这里着重提一下四元数的加法和乘法：

0. $$ (r_1,\ \boldsymbol{\vec{v}}_1) + (r_2,\ \boldsymbol{\vec{v}}_2) = 
(r_1+r_2,\ \boldsymbol{\vec{v}}_1 + \boldsymbol{\vec{v}}_2) $$
0. $$ (r_1,\ \boldsymbol{\vec{v}}_1) (r_2,\ \boldsymbol{\vec{v}}_2) = 
(r_1r_2 - \boldsymbol{\vec{v}}_1 \cdot \boldsymbol{\vec{v}}_2 ,\ 
r_1\boldsymbol{\vec{v}}_2 + r_2\boldsymbol{\vec{v}}_1 +
\boldsymbol{\vec{v}}_1 \times \boldsymbol{\vec{v}}_2) $$

## 开始旋转
问题来了：如果在三维空间中，有一个点 $$ P $$ 要环绕某
单位向量 $$ \boldsymbol{\vec{n}} $$ 按照右手螺旋定则
正向(逆时针)旋转 $$ \theta $$ 弧度，问新点 $$ P' $$ 的位置。

先给出答案： 设点 $$ P' $$ 位置:

$$ \boldsymbol{p'} = 0 + 
x\boldsymbol{\vec{i}} + 
y\boldsymbol{\vec{j}} + 
z\boldsymbol{\vec{k}}
$$

已知单位向量:

$$ \boldsymbol{\vec{n}} = 0 + 
n_x\boldsymbol{\vec{i}} + 
n_y\boldsymbol{\vec{j}} + 
n_z\boldsymbol{\vec{k}}
$$

已知点 $$ P $$ :

$$ \boldsymbol{p} = 0 + 
x_0\boldsymbol{\vec{i}} + 
y_0\boldsymbol{\vec{j}} + 
z_0\boldsymbol{\vec{k}}
$$

则有:

$$ \boldsymbol{q} = cos\left(\frac{\theta}{2}\right) + 
n_x sin\left(\frac{\theta}{2}\right)\boldsymbol{\vec{i}} + 
n_y sin\left(\frac{\theta}{2}\right)\boldsymbol{\vec{j}} + 
n_z sin\left(\frac{\theta}{2}\right)\boldsymbol{\vec{k}}
$$

即：

$$ \boldsymbol{q} = \left(cos \frac{\theta}{2} \ ,\ 
\boldsymbol{\vec{n}} sin\frac{\theta}{2} \right)
$$

使得：

$$ \boldsymbol{p'} = \boldsymbol{q} \boldsymbol{p} \boldsymbol{q}^{-1} $$

答案看上去就是这么奇怪。问题是：为什么是这样？为什么是半角？为啥要乘两次？为啥一个在左一个在右，为啥……？如果只是凑出来的要我硬背公式那是不可能的。没有一个合理的想法或解释，那我是不会认可这个公式的，即使它正确。

## 胡乱分析 (本段为纯粹的原创！)
这段是真正的，纯粹的，胡乱分析，因为谁都没见过四维空间，一切只靠猜测。

### 第四坐标轴？第一坐标轴？
在四维空间中，除了三维中的三只坐标轴以外，多了一根与他们三个都垂直的线，它的出现我们可能观测不到，甚至想象不到是什么样子，就当这根神奇的线存在吧。有了这跟神奇的线，空间升级为超体空间，平面升级为超平面。称之为第四坐标轴一点都不过分。但是当我们使用四元数的时候，却惊恐的发现，我们存在的整个三维，竟是使用三条虚轴来表示的！而与第四坐标轴做的任何垂直超平面，都是一个普通的三维空间，我们身处这个普通的三维空间中，却只能通过坐标原点，窥探到第四坐标轴的一个点。所谓的第四坐标轴恐怕不应该被理解为第四轴，而应该被理解为第一坐标轴！同时，三维中的坐标原点并不代表着它也是第一轴的原点，那恐怕只是第一轴中随意的一个切点。在胡乱分析的这段中，
我们使用 $$ \boldsymbol{\vec{1}}_{Axis} $$ 来表示第一坐标轴。

### 使用四元数表示"点"
在上面的段落中，所有表示"点"的四元数，无一例外的，
第一轴坐标都取了 $$ 0 $$ ，因为我们不关心第一轴上的任何事情。
但是如果使用四元数表示一般的"点"的话，第一轴坐标不再默认为 $$ 0 $$ 。;

点 $$ P $$ 的四元数:

$$ \boldsymbol{p} = p_0 + 
p_x\boldsymbol{\vec{i}} + 
p_y\boldsymbol{\vec{j}} + 
p_z\boldsymbol{\vec{k}}
$$

或者：

$$ \boldsymbol{p} = \left(\ p_0 ,\ \boldsymbol{\vec{v}}_p \right) $$

代表着，在 $$ \boldsymbol{\vec{1}}_{Axis} $$ 的 $$ p_0 $$ 坐标处，
作了一个与 $$ \boldsymbol{\vec{1}}_{Axis} $$ 垂直的超平面(立体空间)，
并以交点作为此空间的坐标原点，点 $$ P $$ 位于此三维空间的 
$$ \boldsymbol{\vec{v}}_p = \left( p_x,\ p_y,\ p_z \right)$$ 处。

### 使用单位四元数表示"旋转"
我们的旋转轴 $$ \boldsymbol{\vec{n}} $$ 位于这个立体空间中，
由于 $$ \boldsymbol{\vec{1}}_{Axis} $$ 与整个空间都垂直，
所以也肯定与 $$ \boldsymbol{\vec{n}} $$ 垂直。
我们通过 $$ \boldsymbol{\vec{1}}_{Axis} $$ 的原点，
做一条与 $$ \boldsymbol{\vec{n}} $$ 平行的坐标轴，
并将这条新轴标记为 $$ \boldsymbol{\vec{n}}_{Axis} $$ ，
单位为 $$ \boldsymbol{\vec{n_0}} $$ 。
注意，既然 $$ \boldsymbol{\vec{1}}_{Axis} $$ 身为第一坐标轴，且把整个三维空间视为"虚"，
则新创建的任何轴都应该在 $$ \boldsymbol{\vec{1}}_{Axis} $$ 逆时针垂直的方向。

<div style="text-align:center;">
<svg height="600" width="600" style="margin: auto;">
 <!-- 1 axis -->
 <line x1="50" y1="400" x2="550" y2="400" stroke="#8888FF" stroke-width="3"/>
 <line x1="530" y1="395" x2="550" y2="400" stroke="#8888FF" stroke-width="3"/>
 <line x1="530" y1="405" x2="550" y2="400" stroke="#8888FF" stroke-width="3"/>
 <line x1="150" y1="400" x2="250" y2="400" stroke="#88FF88" stroke-width="4"/>
 <line x1="240" y1="395" x2="250" y2="400" stroke="#88FF88" stroke-width="3"/>
 <line x1="240" y1="405" x2="250" y2="400" stroke="#88FF88" stroke-width="3"/>
 <text x="500" y="430" fill="#FFFF88" font-size="20px">1_Axis</text>
 <text x="240" y="430" fill="#88FF88" font-size="20px">1</text>
 <!-- n axis -->
 <line x1="150" y1="50" x2="150" y2="550" stroke="#8888FF" stroke-width="3"/>
 <line x1="150" y1="50" x2="145" y2="70" stroke="#8888FF" stroke-width="3"/>
 <line x1="150" y1="50" x2="155" y2="70" stroke="#8888FF" stroke-width="3"/>
 <line x1="150" y1="400" x2="150" y2="300" stroke="#FF8888" stroke-width="4"/>
 <line x1="145" y1="310" x2="150" y2="300" stroke="#FF8888" stroke-width="3"/>
 <line x1="155" y1="310" x2="150" y2="300" stroke="#FF8888" stroke-width="3"/>
 <text x="70" y="80" fill="#FFFF88" font-size="20px">n_Axis</text>
 <text x="120" y="320" fill="#FF8888" font-size="20px">n</text>
 <text x="130" y="325" fill="#FF8888" font-size="12px">0</text>
 
 <!-- O -->
 <circle cx="150" cy="400" r="12" stroke-width="3" stroke="#FFFFFF"/>
 <circle cx="150" cy="400" r="6" fill="#FFFFFF"/>
 <text x="120" y="440" fill="#FFFFFF" font-size="30px">O</text>
</svg>
</div>

上图就是一个最基本（手写SVG累死）的坐标图了。
注意：$$ \boldsymbol{\vec{1}}_{Axis} $$ 是实轴，
$$ \boldsymbol{\vec{n}}_{Axis} $$ 是虚轴，
有 $$ \boldsymbol{\vec{n_0}}^2 = -1 $$ 。

我们把需要研究的四条轴，挑了两条出来研究。这两条轴，事实上和虚数中的两条轴可以类比了。用一个单位虚数表示平面内绕原点正向旋转 $$ \theta $$ 弧度的公式是：

$$ \boldsymbol{z} = cos\theta + \boldsymbol{i}sin\theta $$

所以，使用(单位)四元数表示绕虚轴正向旋转 $$ \theta $$ 弧度的方法就是：

$$ \boldsymbol{q} = \left(cos\theta ,\ \boldsymbol{\vec{n}}sin\theta\right) $$


接下来，以 $$ \boldsymbol{\vec{1}}_{Axis} $$ 轴的坐标 $$ p_0 $$，
做平行于 $$ \boldsymbol{\vec{n}}_{Axis} $$ 的垂"线"，
与 $$ \boldsymbol{\vec{1}}_{Axis} $$ 相交于点 $$ O' $$，
而新作的这条"线"，事实上代表着整个立体空间。但是在这个图中，立体空间仅剩一个维度：就是那条旋转轴！
也就是$$ \boldsymbol{\vec{n}} $$ 所在的那条直线！
而这条线上的每一个点，都代表着三维空间中，与旋转轴垂直并相交于对应点的整个平面！

<div style="text-align:center;">
<svg height="600" width="600" style="margin: auto;">
 <!-- 1 axis -->
 <line x1="50" y1="400" x2="550" y2="400" stroke="#8888FF" stroke-width="3"/>
 <line x1="530" y1="395" x2="550" y2="400" stroke="#8888FF" stroke-width="3"/>
 <line x1="530" y1="405" x2="550" y2="400" stroke="#8888FF" stroke-width="3"/>
 <line x1="150" y1="400" x2="250" y2="400" stroke="#88FF88" stroke-width="4"/>
 <line x1="240" y1="395" x2="250" y2="400" stroke="#88FF88" stroke-width="3"/>
 <line x1="240" y1="405" x2="250" y2="400" stroke="#88FF88" stroke-width="3"/>
 <text x="500" y="430" fill="#FFFF88" font-size="20px">1_Axis</text>
 <text x="240" y="430" fill="#88FF88" font-size="20px">1</text>
 <!-- n axis -->
 <line x1="150" y1="50" x2="150" y2="550" stroke="#8888FF" stroke-width="3"/>
 <line x1="150" y1="50" x2="145" y2="70" stroke="#8888FF" stroke-width="3"/>
 <line x1="150" y1="50" x2="155" y2="70" stroke="#8888FF" stroke-width="3"/>
 <line x1="150" y1="400" x2="150" y2="300" stroke="#FF8888" stroke-width="4"/>
 <line x1="145" y1="310" x2="150" y2="300" stroke="#FF8888" stroke-width="3"/>
 <line x1="155" y1="310" x2="150" y2="300" stroke="#FF8888" stroke-width="3"/>
 <text x="70" y="80" fill="#FFFF88" font-size="20px">n_Axis</text>
 <text x="120" y="320" fill="#FF8888" font-size="20px">n</text>
 <text x="130" y="325" fill="#FF8888" font-size="12px">0</text>

 
 <!-- Space -->
 <line x1="390" y1="70" x2="390" y2="530" stroke-width="3"
   stroke-dasharray="30,10" stroke="#44DDDD"/>
 <line x1="150" y1="400" x2="390" y2="220" stroke-width="3"
   stroke-dasharray="10,10" stroke="#DD44DD"/>
 <circle cx="390" cy="220" r="6" fill="#FFFFFF"/>
 <text x="396" y="250" fill="#FFFFFF" font-size="24px">P (P)</text>
 <text x="404" y="256" fill="#FFFFFF" font-size="18px">0</text>
 <circle cx="390" cy="400" r="6" fill="#FFFFFF"/>
 <text x="396" y="430" fill="#FFFFFF" font-size="24px">O'</text>

 <!-- O -->
 <circle cx="150" cy="400" r="12" stroke-width="3" stroke="#FFFFFF"/>
 <circle cx="150" cy="400" r="6" fill="#FFFFFF"/>
 <text x="120" y="440" fill="#FFFFFF" font-size="30px">O</text>
</svg>
</div>

### 所以把一个平面抽象到一个点到底有什么意义？
很明显，单位四元数代表的"旋转"并没有想象中的那么简单。把平面抽象成点的意义就是：我们先暂时舍弃掉我们能轻易看得见的东西，只有把它们舍弃掉，那些平时根本无法看到的东西，才能些微地从图中窥探到。

### 所以一个单位四元数表示的"旋转"到底造成了什么意想不到的变化？
<div style="text-align:center;">
<svg height="600" width="600" style="margin: auto;">
 <!-- 1 axis -->
 <line x1="50" y1="400" x2="550" y2="400" stroke="#8888FF" stroke-width="3"/>
 <line x1="530" y1="395" x2="550" y2="400" stroke="#8888FF" stroke-width="3"/>
 <line x1="530" y1="405" x2="550" y2="400" stroke="#8888FF" stroke-width="3"/>
 <line x1="150" y1="400" x2="250" y2="400" stroke="#88FF88" stroke-width="4"/>
 <line x1="240" y1="395" x2="250" y2="400" stroke="#88FF88" stroke-width="3"/>
 <line x1="240" y1="405" x2="250" y2="400" stroke="#88FF88" stroke-width="3"/>
 <text x="500" y="430" fill="#FFFF88" font-size="20px">1_Axis</text>
 <text x="240" y="430" fill="#88FF88" font-size="20px">1</text>
 <!-- n axis -->
 <line x1="150" y1="50" x2="150" y2="550" stroke="#8888FF" stroke-width="3"/>
 <line x1="150" y1="50" x2="145" y2="70" stroke="#8888FF" stroke-width="3"/>
 <line x1="150" y1="50" x2="155" y2="70" stroke="#8888FF" stroke-width="3"/>
 <line x1="150" y1="400" x2="150" y2="300" stroke="#FF8888" stroke-width="4"/>
 <line x1="145" y1="310" x2="150" y2="300" stroke="#FF8888" stroke-width="3"/>
 <line x1="155" y1="310" x2="150" y2="300" stroke="#FF8888" stroke-width="3"/>
 <text x="70" y="80" fill="#FFFF88" font-size="20px">n_Axis</text>
 <text x="120" y="320" fill="#FF8888" font-size="20px">n</text>
 <text x="130" y="325" fill="#FF8888" font-size="12px">0</text>

 
 <!-- Space -->
 <line x1="390" y1="70" x2="390" y2="530" stroke-width="3"
   stroke-dasharray="30,10" stroke="#44DDDD"/>
 <line x1="330" y1="100" x2="330" y2="398" stroke-width="3"
   stroke-dasharray="20,10" stroke="#DD4444"/>
 <text x="250" y="300" fill="#FFFFFF" font-size="24px">θ</text>
 <!-- Line 1 -->
 <line x1="150" y1="400" x2="390" y2="220" stroke-width="3"
   stroke-dasharray="10,10" stroke="#DD44DD"/>
 <circle cx="390" cy="220" r="6" fill="#FFFFFF"/>
 <text x="396" y="250" fill="#FFFFFF" font-size="24px">P (P)</text>
 <text x="404" y="256" fill="#FFFFFF" font-size="18px">0</text>
 <!-- Line 2 -->
 <line x1="150" y1="400" x2="330" y2="160" stroke-width="3"
   stroke-dasharray="10,10" stroke="#DD44DD"/>
 <circle cx="330" cy="160" r="6" fill="#FFFFFF"/>
 <text x="296" y="160" fill="#DD4488" font-size="24px">P' (P')</text>
 <text x="304" y="166" fill="#DD4488" font-size="18px">0</text>
 <!-- Foot -->
 <circle cx="390" cy="400" r="6" fill="#FFFFFF"/>
 <text x="396" y="430" fill="#FFFFFF" font-size="24px">O'</text>

 <!-- O -->
 <circle cx="150" cy="400" r="12" stroke-width="3" stroke="#FFFFFF"/>
 <circle cx="150" cy="400" r="6" fill="#FFFFFF"/>
 <text x="120" y="440" fill="#FFFFFF" font-size="30px">O</text>
</svg>
</div>

首先肯定是分析我们看不到的东西啦！如上图所示：亮蓝色的虚线代表着当前被舍弃了两个维度的三维空间，
仅剩下旋转轴一维在这里，$$ O' $$ 是三维空间的坐标原点，$$ OP_0 $$ 是旋转轴所在的直线，
方向与 $$ \boldsymbol{\vec{n_0}} $$ 相同。因为上图损失了两个与旋转轴垂直的维度，
所以我们原本准备分析的点 $$ P $$ 也和它本来应该环绕的 $$ P_0 $$ 重合了。但这都不是重点。

重点是，从上图可以轻易的看到，一个普通的单位四元数，
会变换 $$ P $$ 点的 $$ \boldsymbol{\vec{1}}_{Axis} $$ 坐标！
这意味着什么？上面的介绍已经说明，每一个 $$ \boldsymbol{\vec{1}}_{Axis} $$ 坐标
都代表了一个三维超平面，也就是三维空间，修改了这个坐标，说明这个点被变换到了
另一个三维超平面（图中红色虚线）！也就是说，除非是特殊情况，否则一个普通的单位四元数绝对会把这个点
从三维空间变"没"了。如果我们强行把 $$ \boldsymbol{\vec{1}}_{Axis} $$ 坐标改回来呢？
那会损失 $$ \boldsymbol{\vec{1}}_{Axis} $$ 轴的分量，这是不合乎逻辑的。

有了上面这幅图，我们看到了平时看不到的 $$ \boldsymbol{\vec{1}}_{Axis} $$ 轴，看到了它的影响力，同时也注意到了，普通的单位四元数会做出什么让我们意想不到的事情。

### 那是不是只能使用特殊的，不会让点穿越的单位四元数值才可以呢？
当然不是！我们刚刚分析的都是我们平时看不到的现象，接下来分析一些相对简单地多的东西。

首先我们要把被舍弃的两条轴找回来。这两条轴应该同在一个三维空间中，互相垂直，且垂直于旋转轴。四条轴恢复之后，我们就要重建 $$ P $$ 点坐标。
令新创建的两条轴为 $$ \boldsymbol{\vec{r}}_{Axis}, \boldsymbol{\vec{s}}_{Axis} $$ ,
同时满足 $$ \boldsymbol{\vec{n}}_{Axis}, 
\boldsymbol{\vec{r}}_{Axis}, 
\boldsymbol{\vec{s}}_{Axis} $$ 顺序的右手螺旋定则。
则他们的单位 $$ \boldsymbol{\vec{n_0}}, \boldsymbol{\vec{r_0}}, \boldsymbol{\vec{s_0}}$$ 将对应 $$ \boldsymbol{\vec{k}}, \boldsymbol{\vec{i}}, \boldsymbol{\vec{j}} $$ 的运算规则。

设 $$ \mid P_0P \mid $$ 的长度为 $$ l $$ ，$$ \vec{P_0P} $$ 的方向
是 $$ \boldsymbol{\vec{r_0}} $$ 按 $$ \boldsymbol{\vec{n_0}} $$ 正方向
旋转 $$ \phi $$ 弧度，$$ \boldsymbol{p_n} $$ 是 $$ \boldsymbol{p} $$ 
在 $$ rOs $$ 坐标中的分量(我们这回只看能看到的)，

则：

0. $$ \boldsymbol{q} = cos\theta + \boldsymbol{\vec{n_0}}sin\theta $$
0. $$ \boldsymbol{p_n} = \boldsymbol{\vec{r_0}}lcos\phi + 
\boldsymbol{\vec{s_0}}lsin\phi $$

由于四元数乘法不具有交换率，所以：

0. $$ \boldsymbol{qp_n} = \boldsymbol{\vec{r_0}}lcos\left(\phi+\theta\right) + 
\boldsymbol{\vec{s_0}}lsin\left(\phi+\theta\right) $$
0. $$ \boldsymbol{p_nq} = \boldsymbol{\vec{r_0}}lcos\left(\phi-\theta\right) + 
\boldsymbol{\vec{s_0}}lsin\left(\phi-\theta\right) $$

### 答案已经显而易见了！
接下来的推导过程省略。本文的重点，仅仅是展示和解释平时看不到的那部分图像。
