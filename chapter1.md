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
