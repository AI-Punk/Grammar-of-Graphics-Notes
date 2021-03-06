# 代数

本章主要介绍对数据的处理、重新存储与结构调整以便制作可视化图形。

## 5.1 语法

### 5.1.1 符号
符号用来表示代数中被操作的对象。变量集合(varset)代数中的符号即为变量集合。

### 5.1.2 操作符
操作符是代数中用来创建关系/关联的一种运算。图形系统中有三个操作符，`cross`, `nest`和`blend`。接下来将介绍这些操作的具体定义，我们使用 **A**, **B** 来表示变量集合(varset)，使用 **VA**, **VB** 来表示他们的值域。

#### 5.1.2.1 Cross
cross将右边的变量集合与左边的变量集合进行连接，生成一个元组集合。

![](../5.algebra/cross1.png)

cross可以理解为将两个表中列进行合并。

![](../5.algebra/cross2.png)

举例而言，这里可以理解为A中，1和4为red，2和3为blue（也可以简单理解为red和blue为color字段的取值，1，2，3，4为index或另一列的取值）

![](../5.algebra/cross3.png)

#### 5.1.2.2 Nest
nest操作与cross类似。nest一词含有within的意思。如teachers within school.

![](../5.algebra/nest1.png)

nest中的元组是“有颜色的”或“被标记的”，而cross中的元组则没有。

![](../5.algebra/nest2.png)

这里我理解为nest包含一种层级从属的意味，而cross仅仅是一种交际。所以nest中，我们通常会基于nest进行分组（或颜色与标记操作），如b1对应的a1/b1, a2/b1...

nest的值域需要被提前定义，为了实现这点，我们有三种方法

1. 基于已有的数据，统计对应字段组成的最小唯一元组集合。
2. 基于元数据给出的规则生成。
3. 基于OLAP中的聚合运算生成。

其实，从这里也可以认为，nest是cross中实际存在的元组。cross包含理论上可能存在但在数据中不一定存在的元组。

![](../5.algebra/nest3.png)

来看这个例子：

![](../5.algebra/nest4.png)

这里将A最终拆分为了两个区间，这是由于我们之前提出的nest操作只保留真实存在的值，而不包含理论上所有可能存在的值导致的。

#### 5.1.2.3 Blend
Blend操作为两个变量集合的并集。

![](../5.algebra/blend1.png)

定义

![](../5.algebra/blend2.png)

示例

![](../5.algebra/blend3.png)

### 5.1.3 规则
接下来将给出的每一个证明都分为两部分，一是我们会证明等式两边的变量集合相等。二是我们会证明两个变量集合的值域相等。

#### 5.1.3.1&2 结合律与交换律

![](../5.algebra/rule1.png)

![](../5.algebra/rule2.png)

![](../5.algebra/rule3.png)

#### 5.1.3.3 identity
blend操作的identity为空集；

![](../5.algebra/null.png)

Ω包含了所有的caseID，被称为单位变量集合(unity varset，不确定这样翻译是否合适)，其值域为单位变量(unity value)，在GPL中单位变量为 **1**

### 5.1.4 表达式
表达式是有符号与操作符构成的。项(term)是不含+(blend)的表达式。因子(factor)是不含*(cross)的表达式；单项式(monomial)是有一个项构成的，多项式(polynomial)是由多个项构成的

#### 5.1.4.1 代数形式(algebraic form)
代数形式是指拥有相同数量因子的多项式。代数形式的阶(order)是其项中的因子数量。A \* B + C \* D的阶为2，而 A * B + B 不是代数形式因为他的各个项的阶（因子数量）不同。

有时我们需要确定一个代数表达式中有多少维度，这是我们就需要将其转化为维度的代数形式。
```js
G +  A + B  * C/D
=>
G + A * C/D + B * C/D
=>
G * 1 + A * C/D+ B * C/D

```

#### 5.1.4.2 运算优先级(Operator Precedence)
```js
Nest > Cross > Blend
```

### 5.1.5 SQL等价性
在SQL中，我们仍可以使用这些代数方法：

#### 5.1.5.1 Cross
cross操作可以由内联实现（如果是理论上的所有值的话）
> 问题： 如果是理论上的所有值的话为何不是外联（类似于笛卡尔积）

```SQL
select a.*, b.*
   from a inner join b;
   on a.key = b.key;
```

#### 5.1.5.2 Nest
```SQL
select a.*, b.*
   from a inner join b;
   on a.key = b.key;
```
得到这部分数据后，我们需要将右表的数据作为左表数据的tag

#### 5.1.5.3 Blend
```SQL
select * from X
   union all
   select * from Y;
```

## 5.2 案例
这里使用一个1980年各个城市的人口作为数据集。

![](../5.algebra/figure5.1.png)

### 5.2.1 Cross
图5.2所示为二维散点图，这里加上了2000年各个城市的人口数量；

![](../5.algebra/figure5.2.png)

如果其中一个corss的变量集合是类别类型的，那样我们就可以把图形分为小区间如图5.3；

![](../5.algebra/figure5.3.png)

我们注意到图5.3中有些城市会对应两个点；这是因为在历史上，美国的一些城市名称是仿照其他地方的城市名取得。

我们在图5.4中将这些城市分为两组，美国与其他；借助分组，我们可以得到一张表意更为准确的可视化：
![](../5.algebra/figure5.4.png)

这张图是将三个维度变量集合进行cross操作得到的三维可视化，但我们将group维度转化为切面，使得他们可以被绘制在一张平面上。

我们发现图中芝加哥看起来十分突兀，这是基于其较大的人口；我们期望将这些城市以有序的形态展示出来，但是代数表达式并不允许我们这样做，这是由于group已经与其他字段进行了cross操作；两个切面是共享一个城市顺序的，其中一个有序则会导致另一个无需；这时我们就需要nest操作了。

### 5.2.2 Nest
nest操作可以视为将cross操作的得到的结果删减部分结果得到的。

![](../5.algebra/nextorigin.png)

这次，我们使用`city/group*pop`来制作图表

![](../5.algebra/figure5.5.png)

另外在图5.4中，两个Paris被理解为一个name在不同环境下的不同表现结果，而在5.5中，则被理解为两个不同的城市拥有着相同的名字。虽然这种区别比较微妙，但在很多场景下会变得很重要。

### 5.2.3 Blend

![](../5.algebra/figure5.6.png)

我们使用颜色来区分blend操作合并的两个集合；由于blend操作，使得position仍是一个二维集合，并不会像之前一样产生三维的切片（如果产生切片的话是对两个度量的切片），而是将两个不同的度量在一张图表中展示出来。

`blend`操作与`nest`操作经常在一起使用。这里我们使用`(city/group)*(pop1980+pop2000)`来作为position；在用颜色将两个度量区分出来；

![](../5.algebra/figure5.8.png)

注：图5.8中使用了`color(p1980+p2000)`

有时我们对只有一个变量成员值的变量进行nest，这是为了方便我们在blend后将不同的度量尺度(scale)分离出来。当我们执行了blend操作，pop1980与pop2000由于度量尺度相同，会共享一套度量尺度，也就会被绘制在统一坐标系中。为了避免这种事情发生，我们可以改为将pop1980/p1980与pop2000/p2000进行blend，这样，二者值得含义会变得不同，从而被分离到两个坐标系下。此时，我们发现两个坐标系下的度量尺度变得不同，有些场景下，我们需要再调整度量尺度使得他们在视觉上具有可比性。

![](../5.algebra/figure5.9.png)

## 5.3 其他类型的代数
+ 设计代数(design algebra)
+ 关系代数(relational algebra)
+ 函数代数(Functional algebra)
+ 表格代数(table aglebra)
+ 查询代数(query algebra)
+ 展示代数(display algebra)
