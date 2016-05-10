>在前端，一个简单的UI控件可以拆封为HTML + CSS + JS三个部分，其中CSS和JS都是可选的。 主站上的JS一直欠缺有效的组织形势，因而经常出现漏加载JS文件，或者重复执行了某些JS文件。 因此在2013年初的新UI改版中，我们使用了Widget形式的代码结构来组织我们UI相关的代码。 这里我们详细介绍一下widget实现的相关细节

## 简单入门 
***

以一个简单的例子来介绍一下data-widget
我们现在需要实现一个左侧图片中展示的下拉列表，要求鼠标进入操作区时下拉列表，鼠标移出时隐藏下拉。 首先我们先完成大概如下的HTML结构，注意这里的data-widget属性是一个url，标识着这个widget需要载入这个url所指的js模块进行初始化，如果这个url中含有`#`，那么会载入这个模块下指定的属性来初始化，这个属性名由#后面的字符串指定：

```html
<div data-widget="test_page.js#hoverWidget"> 
<span>当前的标题</span> 
<ul class="ui-hover-list"> 
<li>标题1</li> 
<li>标题2</li> 
<li>标题3</li> 
</ul> 
</div>

```

效果如图：
![image](https://git.youwei.me/backend/dajia365-web/uploads/75b2e3d97ea8dbff1478226ee0e23779/image.png)

然后我们实现最简单的css样式, 我们通过一个名为active的class来控制下拉列表的显示：

```css
.ui-hover-list{ display: none; } 
.active .ui-hover-list{ display: block }

```

最后，我们新建一个名为test_page.js的js文件，并且导出一个名为hoverWidget的接口(与data-widget属性相对应)：

```javascript
var $ = require('jquery');   
exports.hoverWidget = function (config) {
            this.config.$el
                        .addClass('active')
                        .on('mouseleave', function () {
                            $el.removeClass('active');
                        });
        };
});
```

至此，开发工作基本结束了，最后一步我们需要在引入这段HTMl的页面里加上一句JS来初始化所有的widget：
```js
G.use('widget.js', function (Widget) {
    Widget.initWidgets();
    });

```

## 参数设置

    在刚才的示例文件中，我们看到test_page.js模块导出了一个函数，这个函数接受一个config参数，接下来我们还是以一个例子来罗列一下config中的字段，注意'data-' 开头的属性：
```html
<div data-widget="test_page.js#hoverWidget" data-foo="1" data-bar="2" data-pub-sub="3"> 
    <ul data-role="itemList">
     <li data-role="item">1</li>
     <li data-role="item">2</li>
     <li data-role="item">3</li>
    </ul>
</div>
```

在这样一个widget中，config会有以下字段

- config.$el

 $el 指的是该widget的根元素，也就是标记着data-widget的那个元素。

- config.$xxx

 $el元素的子节点中如果出现data-role属性标记的子元素，那么这个子元素将会被自动收集起来作为一个config的一个以$开头的属性。这个属性是一个jquery对象。

 例如上例中的HTMl结构，config中会出现一个config.$itemList的属性，指向ul元素。以及另一个config.$item，指向li元素集合。

- config.xxx

$el元素的上如果出现data-开头的属性，那么这些属性会被自动收集作为config的属性出现。HTML中的data-属性采用连字符格式命名风格，收集到config中时会自动转变为小驼峰风格。

例如上例中的HTML结构，config会出现有以下字段:

 ```js
  config.foo === 1;
  config.bar === 2; 
  config.pubSub === 3;
 ```

***

## 初始化
***

有了data-widget，在页面的开发过程中不再需要关心何时载入js、载入什么js、什么参数这些问题。只要将HTML准备好，js就会自动加载，而完成这个功能也仅仅只需在页面的底部嵌入三行代码：
```js
G.use('widget.js', function (Widget) {
    Widget.initWidgets();
    });

```

上面这段代码会自动处理页面内所有的data-widget元素。 如果一个Widget对应的dom元素在页面加载是并不出现在DOM树内，而是在未来的某个用户动作触发时才加载，例如在一个panel内展现一个widget，那么我们可以使用另外一个接口来初始化：

```js
<script> 
    $('#foo-btn').click(function () { 
    var panel = GJ.panel({ 
        content: '<div data-widget="app/ms_v2/xxx/xxx.js">...</div>' ... 
        });   
    var $el = $(panel.bd).find('[data-widget]');   
    GJ.use('widget.js', function (Widget) { 
        Widget.initWidget($el); }); 
    }); 
</script>
```

## Widget.define
在之前的例子中，我们的初始化函数只是一个简单的函数，这个函数接受一个config，然后进行初始化。面对简单的widget的时候，我们可以这么做，但是如果面对一个data-role众多，逻辑复杂的widget时，这个简单的函数就会变得巨大无比。而我们希望所有的函数不要超过80行。因此我们引入Widget.define。

同样以一个简单的例子来说明：

```js
var $ = require('jquery');   
exports.hoverWidget = Widget.define({
    events:{
        'mouseenter': function () {
            this.config.$el
                        .addClass('active')
                        .on('mouseleave', function () {
                            $el.removeClass('active');
                        });
        }
    },
    init: function (config) {
        this.config = config;
    },
    xxx: function () {
        // some function of this Widget
    }
});
```

### events

这个字段用于定义事件的绑定，通用的格式为:
```
'事件名^[自元素选择符]':回调
```

### init

这个函数用于初始化widget，接受一个config参数，该参数与之前简单版本的widget保持一致。函数内的this指向widget实例。

### 成员属性

所有的其他key都可以通过成员属性的方式访问到。