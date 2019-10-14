
## 取消默认操作
w3c 的方法是 e.preventDefault()，IE 则是使用 e.returnValue = false;

在支持 addEventListener() 的浏览器中，也能通过调用时间对象的 preventDefault() 方法取消时间的默认操作。不过，在 IE9 之前的 IE 中，可以通过设置事件对象的 returnValue 属性为 false 来达到同样的效果。下面的代码假设一个事件处理程序，它使用全部的三种取消技术：
```js
function cancelHandler(event){  
  var event = event || window.event;  //用于IE  
  if(event.preventDefault) event.preventDefault();  //标准技术  
  if(event.returnValue) event.returnValue = false;  //IE  
  return false;   //用于处理使用对象属性注册的处理程序  
}  
```
当前的 DOM 事件模型草案定义了 Event 对象属性 defaultPrevented。
## return false
javascript 的 return false 只会阻止默认行为，而是用 jQuery 的话则既阻止默认行为又防止对象冒泡。

下面这个使用原生 JS，只会阻止默认行为，不会停止冒泡:
```xml
<div id='div'  onclick='alert("div");'>  
 
　　<ul  onclick='alert("ul");'>  
	<li id='ul-a' onclick='alert("li");'>
		<a href="http://caibaojian.com/" id="testB" >caibaojian.com</a>
    </li> 
    </ul>  
 
</div>  
 
var a = document.getElementById("testB");  
 
　　a.onclick = function(){  
 
　　return false;  
 
}
```
## 阻止冒泡
w3c 的方法是 e.stopPropagation()，IE 则是使用 e.cancelBubble = true

在支持 addEventListener() 的浏览器中，可以调用事件对象的一个 stopPropagation() 方法已阻止事件的继续传播。如果在同一对象上定义了其他处理程序，剩下的处理程序将依旧被调用，但调用 stopPropagation() 方法可以在事件传播期间的任何时间调用，它能工作在捕获阶段、事件目标本身中和冒泡阶段。

IE9 之前的IE不支持 stopPropagation() 方法。相反，IE事件对象有一个 cancleBubble 属性，设置这个属性为 true 能阻止事件进一步传播。（ IE8 及之前版本不支持事件传播的捕获阶段，所以冒泡是唯一待取消的事件传播。）

当前的 DOM 事件规范草案在 Event 对象上定义了另一个方法，命名为stopImmediatePropagation()。类似 stopPropagation(),这个方法组织了任何其他对象的事件传播，但也阻止了在相同对象上注册的任何其他事件处理程序的调用。
```xml
<div id='div' onclick='alert("div");'>  
 
　　<ul onclick='alert("ul");'>  
 
　　　　<li onclick='alert("li");'>test</li>  
 
　　</ul>  
 
</div>
```
```js
function stopHandler(event)  
 
    window.event?window.event.cancelBubble=true:event.stopPropagation();  
 
}
```