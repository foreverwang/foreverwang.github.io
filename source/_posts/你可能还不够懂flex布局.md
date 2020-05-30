---
title: flex
date: 2018-10-22 16:21:50
categories: 
- 布局 
tags: 
- flex

---


### flex

#### flex布局中基本概念

- 两种元素类型：flex容器(flex container)、flex项目（flex item 容器成员)。
- 两个轴：主轴（main axis）、交叉轴（cross axis）。
- 占据的空间：项目占据的主轴空间（main size）、项目占据的交叉轴空间（cross size）
- 注意：设为 Flex 布局以后，子元素的float、clear和vertical-align属性将失效。
 
 <!-- more -->
 
#### 属性

  - 容器属性6个
      - display: flex
      - flex-direction
      - justify-content
      - flex-wrap (wrap-reverse)
      - align-items (定义了项目在交叉轴上如何对齐，baseline | stretch)
      - align-content (定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。stretch（默认值）：轴线占满整个交叉轴。)
      - flex-flow (flex-direction和flex-wrap的简写，默认值： row nowrap)
  - 项目属性6个
      - align-self（可覆盖align-items属性）
      - flex-grow (默认值0)
      - flex-shrink （默认值1）
      - flex-basis（默认值为auto，即项目的本来大小）
      - order
      - flex (flex-grow、flex-shrink、flex-basis的简写，默认值：0 1 auto, 后两个属性可选。用两个快捷属性：auto(1 1 auto)、 none(0 0 auto),建议优先使用这两个属性，因为浏览器会推算相关值)

#### flex 布局核心属性
   - flex-basis
        - auto ： 优先设置的宽度，没设置则内容宽度
        - content： 一律内容宽度，不管有没有设置宽度
        - 长度或百分比：没啥好说的
   - flex-grow：剩余空间按比例分配，与项目自身所占空间无关
   - flex-shrink：超出空间按比例挤压项目，项目自身所占空间参与权重计算

#### 计算剩余空间
 > 剩余空间 = 容器所占主轴空间 - 所有项所占主轴空间总和

剩余空间 > 0,说明有剩余空间可分配,即可伸缩项`有条件扩展`,如何扩展看每一项的 `flex-grow`属性；
剩余空间 < 0 我们叫`超出空间`更方便理解,说明可伸缩项占主轴空间总和大于容器占主轴的空间，即可伸缩项目`需要收缩才内适应`容器空间，如何收缩看每一项的`flex-shrink`属性。

####  伸缩项扩展计算公式
> 伸缩项flex-grow权重值/各伸缩项flex-grow值总和 * 剩余空间

#### 伸缩项收缩计算公式（注意了，这里很多人搞错了）
> （伸缩项flex-shrink权重值 * 该伸缩项flex-basis）/ （各伸缩项flex-shrink * 各伸缩项flex-basis 总和)  * 超出空间

注意，与flex-grow在扩展时简单地按比例分配不同，除考虑flex-shrink本身，也要考虑flex-basis。假设每一项flex-shrink都是默认值1，那其实就是按照每一项flex-basis的占比进行收缩。

#### 易忘点
- align-self比align-items属性多了一个auto,其余一样。 align-self:auto(默认值)，继承父元素的align-items。
- flex属性是三个属性的简写：flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto； 
    - flex: 1 即 flex-grow:1; flex-shrink: 0; flex-basis: auto
    - 该属性有两个快捷值：auto (1 1 auto) 和 none (0 0 auto)
- flex-basis: 计算剩余空间是根据该属性值计算的。默认值：auto,项目的本来大小。
    - flex-basis:auto; 如果项目设置了width属性值，则flex-basis实际值等于width值。
    - flex-basis: length 优先级大于width/ height值

