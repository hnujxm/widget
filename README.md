# Widget 

---------

[![Build Status](https://travis-ci.org/aralejs/widget.png)](https://travis-ci.org/aralejs/widget)


Widget 是 UI 组件的基础类，约定了组件的基本生命周期，实现了一些通用功能。基于 Widget
可以构建出任何你想要的 Web 界面组件。

----------

**注**：Handlebars 依赖仅在混入 `Templatable` 后才产生。


## 使用说明


### extend `Widget.extend(properties)`

使用 `extend` 方法，可以基于 `Widget` 来创建子类。

```js
define(function(require, exports, module) {
    var Widget = require('widget');

    // 定义 SimpleTabView 类
    var SimpleTabView = Widget.extend({
        events: {
            'click .nav li': 'switchTo'
        },
        switchTo: function(index) {
            ...
        },
        ...
    });

    // 实例化
    var demo = new SimpleTabView({ element: '#demo' }).render();
});

```

详细示例请访问：[simple-tabview.html](http://aralejs.org/widget/examples/simple-tabview.html)


### initialize `new Widget([config])`

Widget 实例化时，会调用此方法。

```js
var widget = new Widget({
   element: '#demo',
   className: 'widget',
   model: {
       title: '设计原则',
       content: '开放、简单、易用'
   }
});
```

`config` 参数用来传入选项，实例化后可以通过 `get/set` 访问。`config`
参数如果包含 `element` 和 `model` 属性，实例化后会直接放到 `this` 上，即可通过
`this.element`、`this.model` 来获取。


在 `initialize` 方法中，确定了组件构建的基本流程：

```js
// 初始化 attrs
this.initAttrs(config, dataAttrsConfig);

// 初始化 props
this.parseElement();
this.initProps();

// 初始化 events
this.delegateEvents();

// 子类自定义的初始化
this.setup();
```

下面逐一讲述。


### initAttrs `widget.initAttrs(config, [dataAttrsConfig])`

属性的初始化方法。通过该方法，会将用户传入的配置与所继承的默认属性进行合并，并进行初始化操作。

子类如果想在 `initAttrs` 执行之前或之后进行一些额外处理，可以覆盖该方法：

```js
var MyWidget = Widget.extend({
    initAttrs: function(config) {
        // 提前做点处理

        // 调用父类的
        MyWidget.superclass.initAttrs.call(this, config);

        // 之后做点处理
    }
});

```

**注意**：一般情况下不需要覆盖 `initAttrs`。


### parseElement `widget.parseElement()`

该方法只干一件事：根据配置信息，构建好 `this.element`。

默认情况下，如果 `config` 参数中传入了 `element` 属性（取值可为 DOM element / selector），
会直接根据该属性来获取 `this.element` 对象。

`this.element` 是一个 jQuery / Zepto 对象。


### parseElementFromTemplate `widget.parseElementFromTemplate()`

如果 `config` 参数中未传入 `element` 属性，则会根据 `template` 属性来构建
`this.element`。 默认的 `template` 是 `<div></div>`。

子类可覆盖该方法，以支持 Handlebars、Mustache 等模板引擎。


### element `widget.element`

widget 实例对应的 DOM 根节点，是一个 jQuery / Zepto 对象。


### initProps `widget.initProps()`

properties 的初始化方法，提供给子类覆盖，比如：

```js
initProps: function() {
    this.targetElement = $(this.get('target'));
}
```


### delegateEvents `widget.delegateEvents([events])`

### delegateEvents `widget.delegateEvents(eventType, handler)`

注册事件代理。在 Widget 组件的设计里，推荐使用代理的方式来注册事件。这样可以使得对应的
DOM 内容有修改时，无需重新绑定。

`widget.delegateEvents()` 会在实例初始化时自动调用，这时会从 `this.events` 中取得声明的代理事件，比如

```js
var MyWidget = Widget.extend({
    events: {
        "dblclick": "open",
        "click .icon.doc": "select",
        "mouseover .date": "showTooltip"
    },
    open: function() {
        ...
    },
    select: function() {
        ...
    },
    ...
});
```

`events` 中每一项的格式是：`"event selector": "callback"`，当省略 `selector`
时，默认会将事件绑定到 `this.element` 上。`callback` 可以是字符串，表示当前实例上的方法名；
也可以直接传入函数。

`events` 还可以是方法，返回一个 events hash 对象即可。比如

```js
var MyWidget = Widget.extend({
    events: function() {
        var hash = {
            "click": "open",
            "click .close": "close"
        };

        return hash;
    },
    ...
});
```

`events` 中，还支持 `{{name}}` 表达式，比如上面的代码，可以简化为：

```js
var MyWidget = Widget.extend({
    events: {
        "click": "open",
        "click .close": "close",
        "mouseover {{attrs.trigger}}": "open",
        "mouseover {{attrs.panels}}": "hover"
    },
    ...
});
```

实例化后，还可以通过 `delegateEvents` 方法动态添加事件代理：

```js
var myWidget = new Widget();

myWidget.delegateEvents('click .move', function() {
  // ...
});
```


### undelegateEvents `widget.undelegateEvents([eventType])`

卸载事件代理。不带参数时，表示卸载所有事件。


### setup `widget.setup()`

提供给子类覆盖的初始化方法。可以在此处理更多初始化信息，比如

```js
var TabView = Widget.extend({
    ...
    setup: function() {
        this.activeIndex = getActiveIndex();
    },
    ...
});
```


### render `widget.render()`

提供给子类覆盖的初始化方法。render 方法只干一件事件：将 `this.element` 渲染到页面上。

默认无需覆盖。需要覆盖时，请使用 `return this` 来保持该方法的链式约定。


### $ `widget.$(selector)`

在 `this.element` 内查找匹配节点。


### destroy `widget.destroy()`

销毁实例。


### on `widget.on(event, callback, [context])`

这是从 Events 中自动混入进来的方法。还包括 `off` 和 `trigger`。

具体使用请参考 [events 使用文档](http://aralejs.org/events/)。


### autoRenderAll `Widget.autoRenderAll([root], [callback])`

根据 data-widget 属性，自动渲染找到的所有 Widget 类组件。


### query `Widget.query(selector)`

查询与 selector 匹配的第一个 DOM 节点，得到与该 DOM 节点相关联的 Widget 实例。


## Templatable

可混入的功能类，提供 Handlebars 模板支持。

```js
var Templatable = require('widget-templatable');

var MyWidget = Widget.extend({
    Implements: Templatable
});

var myWidget = new MyWidget({
    template: '<h3>{{title}}</h3><ol>{{#each list}}<li>{{item}}</li>{{/each}}',
    model: {
        'title': '标题',
        'list': [
            { 'item': '文章一' },
            { 'item': '文章二' }
        ]
    },
    parentNode: '#demo'
});

myWidget.render();
```


### renderPartial `widget.renderPartial(selector)`

局部渲染。根据传入的 `selector` 参数，刷新匹配的区域。

默认无需覆盖。需要覆盖时，请使用 `return this` 来保持该方法的链式约定。

