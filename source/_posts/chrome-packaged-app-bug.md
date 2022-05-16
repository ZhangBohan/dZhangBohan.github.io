title: Chrome Packaged App的一些坑
date: 2014-05-19 10:47:48
tags:
- Chrome
- CSS
---
最近在写[Http Craft][http-craft]——一个开源`HTTP`请求模拟的`Chrome App`过程中遇到了一个头疼的问题，作为一个`CSS`挫人，记录一下

## 无法对应用内的文本进行选择

无法对应用内的文本进行选择（`input`和`textarea`可以选择）

解决方法：需要向`css`样式中添加如下样式

	body {
	    -webkit-touch-callout: all;
	    -webkit-user-select: all;
	    -khtml-user-select: all;
	    -moz-user-select: all;
	    -ms-user-select: all;
	    user-select: all;
	}

其中样式可选值：

* auto——默认值，用户可以选中元素中的内容
* none——用户不能选择元素中的任何内容
* text——用户可以选择元素中的文本
* element——文本可选，但仅限元素的边界内(只有IE和FF支持)
* all——在编辑器内，如果双击/上下文点击发生在子元素上，改值的最高级祖先元素将被选中。
* -moz-none——firefox私有，元素和子元素的文本将不可选，但是，子元素可以通过text重设回可选。


## 当内容过长时无法向下滚动

解决方法：

	html {
	    overflow: auto;
	}

[http-craft]: https://chrome.google.com/webstore/detail/http-craft/afnnmplamedeifhiecpmjajocogjdinp
