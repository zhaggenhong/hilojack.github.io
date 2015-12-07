---
layout: page
title:
category: blog
description:
---
# Preface
css/html5/js 兼容表
http://caniuse.com/

# flex
todo:
	阮一峰
Flex box现在越来越流行

	flex:
		 initial; 自适应

	.parent{ display:flex;}
	.children1{ flex:1;}
	.children2{ flex:2;} //宽度自适应 比例2

# lib

## bootstrap
http://getbootstrap.com/getting-started/

# autosize
如果想让网页宽度等于屏幕宽度（width=device-width），原始缩放比例（initial-scale=1）为1.0，即网页初始大小占屏幕面积的100%。

　　<meta name="viewport" content="width=device-width, initial-scale=1" />

width 也会受单词的影响，所以你还需要加上

	word-break: break-all;

# color
在计算机中经常使用rgb 三原色来表示所有的颜色。在做艺术设计时，经常使用另一种更多允观的HSL或者HSV 来表示。对于HSL 来说，

	H: Hue 色相
	S: Saration 饱和度,(0%是灰色,100%是饱和)
	V/L: 明度/亮度. (0%是黑色，50%是普通亮度，100%白色)

对于HSL(h,s,l)来说, 如果色相本身对应(hr,hg,hb):

	r = 50%+(hr-50%)*s + {-50% + s*(hr-50%) } *[(l-50%)/50%]
	r = 2 * {l + [s*(hr-0.5)+0.5] - l*[s*(hr-0.5)+0.5]} -1
	g = 2 * {l + [s*(hg-0.5)+0.5] - l*[s*(hg-0.5)+0.5]} -1
	b = 2 * {l + [s*(hb-0.5)+0.5] - l*[s*(hb-0.5)+0.5]} -1

色相表示人类能感知的颜色(rgb 两两组合的颜色)
饱和度表示接近灰色(50%,50%,50%)的程度
明亮度表示接近黑色(0，0，0), 或者白色(1,1,1) 的程度

![Have](/img/ria.color.hue.png)


# 响应式设计(Responsive Design)
http://zh.learnlayout.com/media-queries.html

	@media screen and (min-width:600px) {
	  nav {
		float: left;
		width: 25%;
	  }
	  section {
		margin-left: 25%;
	  }
	}
	@media screen and (max-width:599px) {
	  nav li {
		display: inline;
	  }
	}

# html5
<meta name="viewport" content="width=640">

# 位置

## margin

	Inherited: no

边距:
1. 两边间中距离, 取大边中的最大margin为实际margin.
2. 如果当前元素是float-right, 那么以margin-right为准, 而非margin-left.
3. 如果当前元素是float, 则margin就是距离其下元素的border外沿的距离.

## position

	Inherited: no
	position:
		static; 默认(left/top 不生效)
		relative; 以static为基准
		absolute; 以所在父元素(position!=static)左上角为基准
		fixed; 以窗口左上角为基准,不受鼠标滚动影响

		如果父结点为relative/absolute则以父结点为基准

### height

	height:100%; 要起作用的话，就得使 position: absolute;

padding border 都不占用width height

### middle window

	position:fixed;
    top: 50%;
    left: 50%;
    width:30em;
    height:18em;
    margin-top: -9em; /*set to a negative number 1/2 of your height*/
    margin-left: -15em; /*set to a negative number 1/2 of your width*/
    border: 1px solid #ccc;
    background-color: #f3f3f3;

### center block via block size

	.cent {
	  position:absolute;
	  top:50%;
	  left:50%;
	  margin-top:-50px; /* this is half the height of your div*/
	  margin-left:-100px; /*this is half of width of your div*/
	}

### center block via text-center

	.cent{
			text-align: center;
			text-align: -moz-center/-webkit-center
		}

### center block via margin

	.cent{
		margin: 0 auto;
		width: 80%;
	}


# Css3
CSS3 被划分为模块。 其中最重要的 CSS3 模块包括：

- 选择器
- 框模型
- 背景和边框
- 文本效果
- 2D/3D 转换
- 动画
- 多列布局
- 用户界面

# opacity透明度

	opacity: 0~1
	background: rgba(red,green,blue,opacity);

# background

	background:url x y no-repeat;//xy表示位置, 左上角是(0,0)
	background:rgba/hsla;

## background-color

	background-color:
		//rgba color
		rgba(0,255,0,0.5)
		//hsla
		rgba(0,100%,0,0.5)

## background-size
背景尺寸

	background-size:
		cover; 全覆盖
		10px 20px;放缩
		50% 50%;放缩

## background-position(位置)

 	background-position:
		center center;//x y
		left;//x y

## background-origin
背景定位区域

	content-box
	padding-box
	border-box

## background-image:
	img1,img2, ...;

	background-repeat: no-repeat;
	background-attachment: fixed; //do not scroll with the page

# input

	input[type="submit"] {
		background: limegreen;
		color: black;
		border:0;
	}

# boder

	border-radius: 11px;//圆角半径
	box-shadow: 2px 4px 6px 8px #ccc;
	box-shadow: x  y 模糊值 延伸值 #ccc;//模糊值不能为负

## box-shadow
box-shadow: h-shadow v-shadow blur spread color inset;

	h-v: 位置
	blur: 模糊化
	spread: 尺寸
	color:颜色
	inset: 内部显示

## boder-image

	border-image: url top right bottom left  repeat|initial|inherit;
	object.style.borderImage="url(border.png) 30 30 round"
	 repeat|initial|inherit;
		round: 平铺 改变大小 整数个
		repeat	重复	不改变大小
		stretch		拉伸	改变大小 1个

## border-color
	color 颜色
	initial 默认值
	transparent 透明

# Table
set column width:

	th,td{width:100px;overflow:hidden}
	table{width:1000px;table-layout:fixed}

## border

	合并间隔
	border-collapse: collapse

# About Text

## text-shadow
	text-shadow: 5px 5px 5px #FF0000;
	text-shadow: 水平 垂直 模糊 #FF0000;

## text-transform
	text-transform:
		uppercase|lowercase|capitalize;

## wrap
	word-wrap:
		break-word; 对长单词强制换行, 短单词不受影响
	word-break: 	;控制`字符与单词`的换行
		break-all; 按字符换行(对中文无效)
		break-word;按单词换行, 不过长单词被强制割断.(等同于word-wrap: break-word)
		normal	按单词换行.长单词不换行 (initial)
	white-space: 控制空白(空格, 回车, 长句换行), 注意,它会控制长句换行, 但是不会影响单词换行
		//忽略回车
		nowrap; 合并空格| 忽略回车 | 长句不拆行
		normal; 合并空格| 忽略回车 | 长句要拆行
		//回车换行
		pre;	不合空格| 回车换行 | 长句不拆行
		pre-wrap;不合空格| 回车换行 | 长句要拆行
		pre-line;合并空格| 回车换行 | 长句要拆行

## overflow
overflow 控制元素内容不超出元素本身width/height.

	overflow:
		hidden;
		scroll;
		visiable;

但是同时只要有一个`overflow-x/y` 为hidden, 整个overflow 就会变成 hidden

	<div style=" width: 100px; height: 100px; background: green; top: 30px; left: 300px; position: absolute; ">
		<div style=" width: 150px; height: 150px; background: blue; position: absolute; top: 24px; left: 10px; overflow-y: hidden; ">
			<div style=" width: 300px; height: 300px; background: orange; "></div>
			<div style=" width: 10px; height: 10px; position: absolute; top: 1px; left: -20px; background: red; "></div>
		</div>
	</div>

以限制height 解决方法是：

*不加`overflow`*
只限制元素内的内容的高度 height(比如允许img 压缩高度)

*加`overflow-y:hidden` 或者 `overflow:hidden`*
但不限制元素内的内容的高度 height(比如不允许img 压缩高度)

	<div id="content" style=" width: 200px;   overflow-x: hidden; ">
		  <img src="http://www.hdwallpapers.in/walls/cute_dog_boo-wide.jpg" width="300">
	</div>

## text-overflow
超出长度时加省略号"..."

	div {
		overflow:hidden;
		text-overflow: ellipsis;
	}

# content
设定显示内容：

	content:none | normal |<string>	| url | open-quote | close-quote | no-open-quote | no-close-quote | attr(attribute) | counter(name[, style])

	a:after{
		content:"<" attr(href) ">";
	}

	h1 {
	  counter-increment: headers 10;
	  counter-reset: subsections 5;
	}

Refer to http://www.qianduan.net/css-content-counter-increment-counter-reset.html

# Font

	font-family:
	font-size:2em;
	font-weight:
		normal bold 100 200
	font-style:
		normal	italic

# transform(2D)
	-webkit-transform:
		rotate(30deg); 顺时针30度旋转
		rotateX();
		rotateY();
		translate(xpx, ypx); 移动
		scale(2, 4); 水平扩大两倍, 垂直扩大4倍
		skew(30deg,20deg) 围绕 X 轴把元素倾斜 30 度，围绕 Y 轴倾斜 20 度。//-webkit-transform-origin:0px 0px ;倾斜的基点
		#
		matrix() 方法把所有 2D 转换方法组合在一起[matrix].
			matrix(1, 0, 0, 1, e, f) -> (1*x+0*y+e, 0*x+1*y+f) -> (x+e, y+f) -> translate(epx, fpx)
			matrix(cosθ,sinθ,-sinθ,cosθ,0,0) -> (x*cosθ-y*sinθ, x*sinθ+y*cosθ) -> rotate(θ);
			matrix(1,tan(θy),tan(θx),1,0,0) -> (x+y*tan(θx), x*tan(θy)+y) -> skew(θxdeg, θydeg);

# transition
CSS3 过渡是元素从一种样式逐渐改变为另一种的效果。
要实现这一点，主要规定：

	transition: property time action;
		property 规定您希望把效果添加到哪个 CSS 属性上
		time 规定效果的时长
		action:
			linear	规定以相同速度开始至结束的过渡效果（等于 cubic-bezier(0,0,1,1)）。
			ease	规定慢速开始，然后变快，然后慢速结束的过渡效果（cubic-bezier(0.25,0.1,0.25,1)）。
			ease-in	规定以慢速开始的过渡效果（等于 cubic-bezier(0.42,0,1,1)）。
			ease-out	规定以慢速结束的过渡效果（等于 cubic-bezier(0,0,0.58,1)）。
			ease-in-out	规定以慢速开始和结束的过渡效果（等于 cubic-bezier(0.42,0,0.58,1)）。
			cubic-bezier(n,n,n,n)	在 cubic-bezier 函数中定义自己的值。

	transition:width 2s, height 2s linear;
	transition:2s; //所有的属性
	transition:1s width, 1s 2s height cubic-bezier(.8,.9,.1,2);//2s是延迟
	img:hover{} 经常用

## cubic-bezier 贝塞尔曲线
上文提到的 cubic-bezier 公式为

	 B(t) = f(p0, p1, p2, p3) = (1-t)^3*p0 + 3*(1-t)^2*t*p1 + 3*(1-t)*t^2p2 + t^3p3
	 当p0=p1=0, p2=p3=1 时，
	 	3(1-t)
	 其中 P0, P1 ,P2, P3 都为两维 xy 向量

# Animation 动画
设定每个时间点的样式集.

	@keyframes myfirst
	{
		0%   {background:red; left:0px; top:0px;}
		25%  {background:yellow; left:200px; top:0px;}
		50%  {background:blue; left:200px; top:200px;}
		75%  {background:green; left:0px; top:200px;}
		100% {background:red; left:0px; top:0px;}
	}
	div {
		width:100px;
		height:100px;
		background:red;
		position:relative;
		animation:myfirst 5s;
		animation:myfirst 5s backwards;//动画结束后返回第一桢
		animation:myfirst 5s ;//动画结束后返回每一桢
		animation:myfirst 5s 3;//播放3次
		animation:myfirst 5s infinite;//播放无限次
		-moz-animation:myfirst 5s; /* Firefox */
		-webkit-animation:myfirst 5s; /* Safari and Chrome */
		-o-animation:myfirst 5s; /* Opera */
	}

## animation-fill-mode属性。
动画保持的结束状态

	none：默认值，回到动画没开始时的状态。
	backwards：让动画回到第一帧的状态。
	both: 循环时, 轮流应用forwards和backwards规则

## animation-direction
动画循环播放时，每次都是从结束状态跳回到起始状态，再开始播放。animation-direction属性，可以改变这种行为。(浏览器对alternate的支持不好, 请慎用)

	normal: 正常播放
	reverse: 倒序动画
	alternate: 渐变, 循环播放时, 从结束到开始要平滑的过渡(实现规则是:step1, step2-reverse, step3, step4-reverse ...)
	alternate-reverse: 对倒序动画做渐变

## animation-play-state
有时，动画播放过程中，会突然停止。这时，默认行为是跳回到动画的开始状态。

	animation-play-state: running;
	animation-play-state: paused; //暂停

## 分布过渡step(time)
看看这个[typing](http://dabblet.com/gist/1745856)

	@-webkit-keyframes typing { from { width: 0; } }
	-webkit-animation: 10s typing infinite steps(10); //打字的效果

# 多列布局

	-webkit-column-count:3; //列数/* Safari and Chrome */
	-webkit-column-gap: 30px;//列间隔
	-webkit-column-rule:3px outset #ff0000;//列边宽度及样式

# 用户界面属性：

1. resize
1. box-sizing
1. outline-offset


## inline-block
请使用 inline-block 而不是float
http://zh.learnlayout.com/inline-block-layout.html

可以使用 inline-block 来布局。有一些事情需要你牢记：

1. vertical-align 属性会影响到 inline-block 元素，你可能会把它的值设置为 top 。
1. 你需要设置每一列的宽度
1. 如果HTML源代码中元素之间有空格，那么列与列之间会产生空隙


## box-sizing

	box-sizing: 
		border-box;
			width = width + pad + border
		content-box:

box-sizing 属性允许您以确切的方式定义width 是否包含padding + border

	<head>
		<style>
			div.container {
				width:30em;
				border:1em solid;
			}
			div.box {
				box-sizing:border-box;// 不加的话, box不会并列(默认width+border>50%)
				width:50%;height:100px;
				border:1em solid red;
				float:left;
			}
		</style>
	</head>
	<body>
		<div class="container">
			<div class="box">这个 div 占据左半部分。</div>
			<div class="box">这个 div 占据右半部分。</div>
			<div class="" style=" background: blue; height: 100px; clear: left; ">abc</div>
		</div>
	</body>

float 使得block 变成漂浮(不过z-index 不变哦)滑块, div 不会被撑大，除非有：`clear:both` 或者`文字`撑大

	clear: both
		清理float 的漂浮动，但是不扩充div 高宽度
	div .clear: overflow: auto
		清理本div 内的float 的漂浮，且填扩充div 高宽度

## float
float 会受到自身`margin`同样基于margin 准则

	margin 是定义border 外边界之间的距离
	如果两元素的margin 距离定义重合，则取最大margin

float 块的margin 是在不浮动块的基础上叠加的：margin-top = 50px + 45px

	<div style="width: 100%;height: 50px;background: yellow;float:left;"></div>
	<div style="margin-top: 45px; "></div>

如果div 的height 为0，则距离是值与其相邻元素的最小min(margin-top,margin-botom....)

	<div style="margin-top: 100px; "></div>
	<div style="margin-top: 45px; height:1px"></div>

## outline-offset
outline-offset 属性对轮廓进行偏移，并在超出边框边缘的位置绘制轮廓。
	轮廓不占用空间

	div {
		border:2px solid black;
		outline:2px solid red;
		outline-offset:15px; //轮廓偏移
	}

# image

	Formal grammar: linear-gradient(  [ <angle> | to <side-or-corner> ,]? <color-stop> [, <color-stop>]+ )
									  \---------------------------------/ \----------------------------/
										Definition of the gradient line         List of color stops

						  where <side-or-corner> = [left | right] || [top | bottom]
							and <color-stop> = <color> [ <percentage> | <length> ]?
	linear-gradient( 45deg, blue, red );           /* A gradient on 45deg axis starting blue and finishing red */
	linear-gradient( to left top, blue, red);      /* A gradient going from the bottom right to the top left starting blue and
													  finishing red */

	linear-gradient( 0deg, blue, green 40%, red ); /* A gradient going from the bottom to top, starting blue, being green after 40%
													  and finishing red */

# Reference
- [matrix]
- [css animation]
- [cubic-bezier]
- [css-book]
- [css-book-zh]

[css-book]: http://learn.shayhowe.com/advanced-html-css/performance-organization/#strategy-structure
[css-book-zh]: http://zh.learnlayout.com/box-model.html
[matrix]: http://www.zhangxinxu.com/wordpress/2012/06/css3-transform-matrix-%E7%9F%A9%E9%98%B5/
[css animation]:http://www.ruanyifeng.com/blog/2014/02/css_transition_and_animation.html
[cubic-bezier]: http://yiminghe.iteye.com/blog/1762706
[fighting-the-space-between-inline-block-elements]:http://css-tricks.com/fighting-the-space-between-inline-block-elements/
