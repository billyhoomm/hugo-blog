
### 1.水平居中
行级元素设置其父元素的text-align center，块级元素设置其本身的left 和 right margins为auto即可，下面主要介绍下容器内的垂直居中和完全居中
### 2.line-height
单行文本垂直居中
```html
<div id="parent">
  <div id="child">Text here</div>
</div>
```
```css
#child {
  line-height: 200px;
  text-align:center;
}
```
图片垂直居中：
```xml
<div id="parent">
  <img src="">
</div>
```
```css
#parent {
  line-height: 200px;
}
#parent img {
  vertical-align: middle;
}
```
### 3.table & table-cell
通用方法（vertical-align只对行内元素或者行内快起效，table-cell默认为行内块）
```xml
<div id="parent">
  <div id="child">Content here</div>
</div>
```
```css
#parent {display: table;}
#child {
  display: table-cell;
  vertical-align: middle;
}
```
### 4.absolute-position & negative-margin
```xml
<div id="parent">
  <div id="child">Content here</div>
</div>
```
```css
#parent{
	width:600px;
	height:200px;
	position:relative;
}
#child{
	width:300px;
	height:100px;
	left:50%;
	top:50%;
	margin-left: -150px;/*负的元素宽度的一半*/
	margin-top: -50px;/*负的元素高度的一半*/
	position:absolute;
}
```
### 5.absolute-position & margin-auto
实现的效果和上面的相同，这种在做响应式中用起来比较方便（此特性只支持IE8+）
```xml
<div id="parent">
  <div id="child">Content here</div>
</div>
```
```css
#parent {
  position: relative;
  width:600px;
  height:200px;
}
 
#child {
  width: 50%;
  height: 50%;
  overflow: auto;
  margin: auto;
  position: absolute;
  top: 0; left: 0; bottom: 0; right: 0;
}
```