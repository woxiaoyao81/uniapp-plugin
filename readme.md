## 由来

目前使用了uniUI和uViewUI两个系列组件，感觉最大问题一是他们都是优先考虑竖版，另一个就是几乎所有组件的通病，就是使用内部使用rpx定义尺寸导致不灵活，在PHP中文十三期培训中学习了父级影响子元素的两类单位:
- **em与rem** 通过em特殊的特性(font-size继承父级，本身margin、padding等不可继承属性则依据本身)来定义内部组件的margin、padding、position等。
- **百分比**  百分比通常定义width和height，再一个如背景图backgroud-size是依据块的大小来定义的，想实现 **宽高成比例自适应** ，可定义如width:60%;padding-bottom:10%来保证6:1，因为width、padding和margin都是依据父级宽度。具体可查MDN。

## 主要内容

1、基于现有插件修改和增强
2、自己编写的插件

## 主要特点

1、使用CSS3的Flex或Grid布局
2、组件由布局自适应容器，即横屏或竖屏自适应
3、尽量适配APP、H5和微信小程序端
