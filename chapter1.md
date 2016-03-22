# js事件

从一个简单的事件封装说起：
### 使用addEventListener或attachEvent
```html
<div id="d4">Div4 Element</div>
<script type="text/javascript">
var d4 = document.getElementById('d4');
function clk(){alert(4)}
if(d4.addEventListener){
d4.addEventListener('click',clk,false);	
}
if(d4.attachEvent){	
d4.attachEvent('onclick',clk);
}
</script>
```
这是目前推荐的方式，可以为元素添加多个事件handler，支持事件冒泡和捕获*（IE6/7/8仍然没有遵循标准而使用了自己专有的attachEvent，且不支持事件捕获。）
*
我们把上面的方法封装一下：
```js
/**
 *@param {object}el HTML Element
 *@param {object} type 事件类型
 *@param {object} fn事件handler
 */
 function addEvent(el, type, fn) {
   if(el.addEventListener){
       el.addEventListener(type, fn, false);
   }else{
     el.attachEvent('on'+type, fn);
   }
 }
```
用一下这个函数
```js
addEvent(document, 'click', function () {
  console.log(this, arguments[0]);
});
//在标准浏览器下面，this就是dom本身而ie678却是window对象
```
改进
```js
function addEvent(el, type, fn){
  if(el.addEventListener){
    el.addEventListener(type, fn, false);
  }else{
    el['e'+fn] = function() {
      fn.call(el, window.event);
    }
    el.attachEvent('on'+type, el['e'+fn]);
  }
}
```
上面我们封装了一个addEvent，解决了IE678handler中this只想window的问题，并且同意了事件对象作为事件handler的第一个参数传入。
补上删除事件函数：
```js
  E={
    add: function (el, type, fn) {
      if(el.addEventListener){
        el.addEventListener(type, fn, false);
      }else{
        el['e'+fn] = function() {
          fn.call(el, window.event);
        }
        el.attachEvent('on'+type, el['e'+fn]);
      }
    }
  },
  remove: function(el, type, fn){
    if(el.removeEventListener){
      el.removeEventListener(type, fn, false);
    }else{
      el.removeEvent('on'+type, el['e'+fn]);
    }
  }
```
到这里我们已经拥有了天价删除事件两个方法，并且解决了哥哥浏览器下的部分差异，现在天价一个主动的触发事件的方法（*该方法模拟用户行为，如点击操作*）标准的使用dispatchEvent方法，IE678使用fireEvent方法，因为可能出线异常，使用了try catch

```js
  E={
      add: function (el, type, fn) {
        if(el.addEventListener){
          el.addEventListener(type, fn, false);
        }else{
          el['e'+fn] = function() {
            fn.call(el, window.event);
          }
          el.attachEvent('on'+type, el['e'+fn]);
        }
      }
    },
    remove: function(el, type, fn){
      if(el.removeEventListener){
        el.removeEventListener(type, fn, false);
      }else{
        el.removeEvent('on'+type, el['e'+fn]);
      }
    },
    dispatch: function(el, type){
      try{
        if(el.dispatchEvent){
          var evt = document.creatEvent('Event');
          evt.iniEvent(type, true, true);
          el.dispatchEvent(evt);
        }else if(el.fireEvent){
          el.fireEvent('on'+ type);
        }
      }catch(e){}
    }
```
fixEvent
```js
function _fixEvent( evt, el ) {
	var props = "altKey attrChange attrName bubbles button cancelable charCode clientX clientY ctrlKey currentTarget data detail eventPhase fromElement handler keyCode layerX layerY metaKey newValue offsetX offsetY originalTarget pageX pageY prevValue relatedNode relatedTarget screenX screenY shiftKey srcElement target toElement view wheelDelta which".split(" "),
		len   = props.length;
	function now() {return (new Date).getTime();}
	function returnFalse() {return false;}
	function returnTrue() {return true;}
	function Event( src ) {
		this.originalEvent = src;
		this.type = src.type;
		this.timeStamp = now();
	}
	Event.prototype = {
		preventDefault: function() {
			this.isDefaultPrevented = returnTrue;
			var e = this.originalEvent;
			if( e.preventDefault ) {
				e.preventDefault();
			}
			e.returnValue = false;
		},
		stopPropagation: function() {
			this.isPropagationStopped = returnTrue;
			var e = this.originalEvent;
			if( e.stopPropagation ) {
				e.stopPropagation();
			}		
			e.cancelBubble = true;
		},
		stopImmediatePropagation: function() {
			this.isImmediatePropagationStopped = returnTrue;
			this.stopPropagation();
		},
		isDefaultPrevented: returnFalse,
		isPropagationStopped: returnFalse,
		isImmediatePropagationStopped: returnFalse
	};
	var originalEvent = evt;
	evt = new Event( originalEvent );
	
	for(var i = len, prop; i;) {
		prop = props[ --i ];
		evt[ prop ] = originalEvent[ prop ];
	}
	if(!evt.target) {
		evt.target = evt.srcElement || document;
	}
	if( evt.target.nodeType === 3 ) {
		evt.target = evt.target.parentNode;
	}
	if( !evt.relatedTarget && evt.fromElement ) {
		evt.relatedTarget = evt.fromElement === evt.target ? evt.toElement : evt.fromElement;
	}
	if( evt.pageX == null && evt.clientX != null ) {
		var doc = document.documentElement, body = document.body;
		evt.pageX = evt.clientX + (doc && doc.scrollLeft || body && body.scrollLeft || 0) - (doc && doc.clientLeft || body && body.clientLeft || 0);
		evt.pageY = evt.clientY + (doc && doc.scrollTop  || body && body.scrollTop  || 0) - (doc && doc.clientTop  || body && body.clientTop  || 0);
	}
	if( !evt.which && ((evt.charCode || evt.charCode === 0) ? evt.charCode : evt.keyCode) ) {
		evt.which = evt.charCode || evt.keyCode;
	}
	if( !evt.metaKey && evt.ctrlKey ) {
		evt.metaKey = evt.ctrlKey;
	}
	if( !evt.which && evt.button !== undefined ) {
		evt.which = (evt.button & 1 ? 1 : ( evt.button & 2 ? 3 : ( evt.button & 4 ? 2 : 0 ) ));
	}		
	if(!evt.currentTarget) evt.currentTarget = el;
	return evt;
}	

```




















































