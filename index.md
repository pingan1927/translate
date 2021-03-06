# 用AMD，CommonJS 和 ES Harmony编写模块化的JavaScript #

## 模块化 ##
> 解耦你的程序的重要性

当我们说一个程序是模块化的，通常我们指它是由一组存放在模块中的高度解耦的独立功能片段所组成。正如你所了解的，[松耦合](http://arguments.callee.info/2009/05/18/javascript-design-patterns--mediator/)通过尽可能的减少依赖从而使得维护应用程序变得更简单。当松耦合被高效地实现时，可以很容易地观察到系统的一部分变化是如何影响到另一部分的。

然而与传统编程语言不同的是，当前版本的JavaScript（[ECMA-262](http://www.ecma-international.org/publications/standards/Ecma-262.htm)）规范并没有提供一个清晰、有条理的导入模块代码的方式。这个问题之前并不引人注意，直到最近几年，人们对更有条理地组织JavaScript程序代码的需求变得愈加强烈。

然而，当前开发人员仍然只能使用[模块](http://www.adequatelygood.com/2010/3/JavaScript-Module-Pattern-In-Depth)或者[对象字面量](http://blog.rebeccamurphey.com/2009/10/15/using-objects-to-organize-your-code)模式的一些变体模式。以这样的方式，模块脚本会在Dom中通过一个全局变量命名空间的方式组织，但这依然很容易导致你的架构命名冲突。并且如果你不进行人为的处理或借助第三方工具，处理模块的依赖关系还是很繁琐的。

虽然含有这些问题原生解决方案的ES Harmony还有一些遥遥无期，不过好消息从今天起我们已经能够轻松地编写模块化的JavaScript代码了。

在这篇文章中，我们会一起看看三种编写模块化JavaScript的格式：**AMD**、**CommonJS**以及下一个JavaScript版本的提案，**Harmony**。

## 前奏 ##

> 关于脚本加载器的说明

讨论AMD和CommonJS模块之前，我们还是有必要谈谈[脚本加载器](http://msdn.microsoft.com/en-us/scriptjunkie/hh227261)。目前，为了使模块化的JavaScript能在当今的应用程序中广泛的发挥作用，脚本加载是至关重要的一点。为了能从本文中获得最大收益，我建议大家先对流行的脚本加载工具是如何工作的做一个**基本了解**，从而方便对本文中对模块格式解释的理解。

目前有很多很好的采用AMD或CJS格式的加载器用来处理模块加载，而我的个人选择是 [RequireJS](http://requirejs.org/) 和[curl.js](https://github.com/unscriptable/curl)。关于这些工具的完整教程不在本文讨论的范围之内，但如果你想更多地了解，我强烈建议阅读John Hann关于[curl.js](http://unscriptable.com/index.php/2011/03/30/curl-js-yet-another-amd-loader/)的文章以及James Burke的 [RequireJS](http://requirejs.org/docs/api.html)的API文档。

从最终产品角度来说，在开发模块时，使用优化工具（例如RequireJS优化器）用来串联脚本在部署时是非常值得推荐的。并且有趣的是，配合[Almond](https://github.com/jrburke/almond) AMD shim，RequireJS可以不与部署站点上的模块合并起来，你可以认为脚本加载器可以非常容易的从开发环境中剥离出去。

此外，James Burke还说RequireJS会有助于解决在页面加载后依然要动态加载模块的场景，记住这些点，让我们开始吧。

## AMD ##

> 一个在浏览器端编写模块化JavaScript的格式

AMD（异步模块定义）格式的总体目标是提供一个开发人员可以使用的模块化JavaScript解决方案。它诞生于Dojo项目中使用XHR+ eval的实现，其支持者希望能够通过避免新解决方案遭受过去不断遇到的问题的情况。

AMD本是一个提案，它定义了模块使得模块及其依赖都可以被[异步](http://dictionary.reference.com/browse/asynchronous)加载。它有一系列显著的优势，其中包括异步以及天生的高度灵活性（去除了模块代码和模块标识之间可能的高耦合性）。许多开发人员喜欢使用它并且认为这是一个向ES Harmony[模块系统](http://wiki.ecmascript.org/doku.php?id=harmony:modules)发展的可靠的跳板。

AMD最开始作为在CommonJS目录中的一个模块格式的规范草案，但是由于它无法达成完全共识，格式的进一步发展就转移到了[amdjs讨论组](https://github.com/amdjs)。

今天AMD被囊括在包括Dojo（1.7），MooTools(2.0),Firebug(1.8)甚至JQuery（1.7）等多个项目中。虽然CommonJS的AMD格式也有广泛的使用，但AMD最纯粹的描述还是仅指AMD本身或者异步模块支持，而不是CJS规范的参与者在wishlist目录上所罗列的。

> **说明：** 有一次提案被作为Module Transport/C规范提交，但是这个规范对于传输目前已知的CJS模块并不合适，不过，在定义模块时选择AMD的命名约定却更有意义。

### 模块入门 ###

在这里需要关注的两个关键概念是实现了模块定义的`define`方法以及用于处理依赖加载的`require`方法。根据提议，*define*使用以下格式来定义命名或者未命名模块：

```
define(
    module_id /*可选*/,
    [dependencies] /*可选*/,
    definition function /*初始化模块的函数 或者 对象*/
);
```

正如你在行内注释了解到的一样，`module_id`是一个可选的参数，这个参数通常在使用非AMD的串联工具时才需要（但也会有这个参数非常有用的个例存在）。当删掉这个参数时，我们称它为匿名模块。

当我们使用匿名模块时，模块识别的原则就是DRY（Don't Repeat Yourself)，通过细化模块来避免文件名和代码的重复。因为代码更具有可移植性，它能够很容易的移动到其它位置(或者文件系统）而不需要改变代码本身或者只改变代码的ID。在简单的包或不使用包的情况下`module_id`相当于文件路径。开发人员可以在多种环境下运行同一段代码而仅仅需要使用一个AMD优化器，这个优化器（如[r.js](https://github.com/jrburke/r.js/)）也可以在CommonJS环境下工作。

回到define定义上，dependencies参数表现为一个依赖数组，这个依赖数组是你定义的模块所需要的。第三个参数（'definition function'）则是一个函数，用来执行模块的实例化工作。一个标准的模块应该被定义成以下形式：

### 理解AMD: define() ###

```
// A module_id (myModule) is used here for demonstration purposes only
define('myModule',
    ['foo', 'bar'],
    // module definition function
    // dependencies (foo and bar) are mapped to function parameters
    function ( foo, bar ) {
        // return a value that defines the module export
        // (i.e the functionality we want to expose for consumption)
        // create your module here
        var myModule = {
            doStuff:function(){
                console.log('Yay! Stuff');
            }
        }
        return myModule;
});
// An alternative example could be..
define('myModule',
    ['math', 'graph'],
    function ( math, graph ) {
        // Note that this is a slightly different pattern
        // With AMD, it's possible to define modules in a few
        // different ways due as it's relatively flexible with
        // certain aspects of the syntax
        return {
            plot: function(x, y){
                return graph.drawPie(math.randomGrid(x,y));
            }
        }
    };
});
```

而另一方面，require则用于加载处于顶层的JavaScript文件代码或你希望的在模块内部动态地获取依赖。一个使用例子如下：

### 理解AMD:require() ###

```
// Consider 'foo' and 'bar' are two external modules
// In this example, the 'exports' from the two modules loaded are passed as
// function arguments to the callback (foo and bar)
// so that they can similarly be accessed

require(['foo', 'bar'], function ( foo, bar ) {
        // rest of your code here
        foo.doSomething();
});
```

### 动态加载的依赖 ###

```
define(function ( require ) {
    var isReady = false, foobar;

    // note the inline require within our module definition
    require(['foo', 'bar'], function (foo, bar) {
        isReady = true;
        foobar = foo() + bar();
    });

    // we can still return a module
    return {
        isReady: isReady,
        foobar: foobar
    };
});
```

### 理解AMD: 插件 ###

下面是定义一个AMD兼容插件的例子：

```
// With AMD, it's possible to load in assets of almost any kind
// including text-files and HTML. This enables us to have template
// dependencies which can be used to skin components either on
// page-load or dynamically.

define(['./templates', 'text!./template.md','css!./template.css'],
    function( templates, template ){
        console.log(templates);
        // do some fun template stuff here.
    }
});
```
/// XXX
> **注意**: 尽管上面的例子中我们使用css!的形式来加载css，但切记这种方法存在许多值得注意的地方，例如不能确切知道CSS何时加载完毕。这依赖于你构建的方式，比如CSS作为一个依赖被包含进优化后的文件的时候，在这种情况下，应该慎用作为依赖加载的CSS。

### 使用require.js加载AMD模块 ###

```
require(['app/myModule'],
    function( myModule ){
        // start the main module which in-turn
        // loads other modules
        var module = new myModule();
        module.doStuff();
});
```

### 使用curl.js加载AMD模块 ###

```
curl(['app/myModule.js'],
    function( myModule ){
        // start the main module which in-turn
        // loads other modules
        var module = new myModule();
        module.doStuff();
});
```

### 具备Deferred依赖的模块 ###

```
// This could be compatible with jQuery's Deferred implementation,
// futures.js (slightly different syntax) or any one of a number
// of other implementations
define(['lib/Deferred'], function( Deferred ){
    var defer = new Deferred();
    require(['lib/templates/?index.html','lib/data/?stats'],
        function( template, data ){
            defer.resolve({ template: template, data:data });
        }
    );
    return defer.promise();
});
```

### 为什么说AMD是编写模块化的JavaScript的好选择？ ###

- 为如何处理定义灵活的模块提供了一个明确的建议。
- 语法比目前我们依靠的全局命名空间以及`<script>`标签等解决方案清晰。有一个干净的方式用来声明独立的模块以及可能的依赖。
- 模块定义是经过封装的，帮助我们避免污染全局命名空间。
- 比起其他可选的解决方案（例如立马会介绍到的CommonJS）工作得更好。在跨域、本地、以及调试方面没有出现问题。使用时不需要依赖服务器端的工具。大多数AMD加载器在浏览器端加载模块时并不需要build过程。
- 提供了 'transport' 方法用于将多个模块包含在一个文件当中。其他的方法例如CommonJS却必须符合某种transport格式。
- 当需要时可以对脚本进行懒加载。

### 相关阅读 ###
- [The RequireJS Guide To AMD](http://requirejs.org/docs/whyamd.html)
- [What's the fastest way to load AMD modules?](http://unscriptable.com/index.php/2011/09/21/what-is-the-fastest-way-to-load-amd-modules/)
- [AMD vs. CJS, what's the better format?](http://unscriptable.com/index.php/2011/09/30/amd-versus-cjs-whats-the-best-format/)
- [AMD Is Better For The Web Than CommonJS Modules](http://blog.millermedeiros.com/2011/09/amd-is-better-for-the-web-than-commonjs-modules/)
- [The Future Is Modules Not Frameworks](http://unscriptable.com/code/Modules-Frameworks/)
- [AMD No Longer A CommonJS Specification](http://groups.google.com/group/commonjs/browse_thread/thread/96a0963bcb4ca78f/cf73db49ce267ce1?lnk=gst#)
- [On Inventing JavaScript Module Formats And Script Loaders](http://tagneto.blogspot.com/2011/04/on-inventing-js-module-formats-and.html)
- [The AMD Mailing List](http://groups.google.com/group/amd-implement)

### AMD模块和Dojo ###

在Dojo中定义兼容AMD的模块相当简单。类似之前的，定义任意模块，依赖放在一个数组中作为第一个参数并且提供一个回调（工厂函数），一旦依赖加载完毕这个回调将运行这个模块。例如：

```
define(["dijit/Tooltip"], function( Tooltip ){
    //Our dijit tooltip is now available for local use
    new Tooltip(...);
});
```

其模块的异步特性使得模块可以被Dojo异步加载器、RequireJS或者标准的[dojo.require()](http://docs.dojocampus.org/dojo/require)模块加载器等你习惯的模块加载器使用。

如果想更多地了解模块引用的，有许多有趣的陷阱，知道之后会很有用。虽然AMD主张的引用模块的方式，即在依赖列表里用一系列匹配参数声明它们，但这没有被Dojo 1.6的构建系统所支持-它仅能在兼容AMD的加载器上工作。例如：

```
define(["dojo/cookie", "dijit/Tooltip"], function( cookie, Tooltip ){
    var cookieValue = cookie("cookieName");
    new Tree(...);
});
```

嵌套命名空间作为一个模块有很多优势，不再需要每次直接引用完整的命名空间 - 在依赖中所有我们需要的只是`dojo/cookie`路径，而一旦给它赋予一个别名，我们就可以通过变量来引用。这样在您的应用程序中就不再需要反复键入`dojo.`了。

> **注意**：尽管Dojo 1.6没有官方地支持基于用户的AMD模块（或者异步加载），仍可以通过使用一些不同的脚本加载器来和Dojo配合工作。目前，所有的Dojo 核心以及Dijit模块都被转换成AMD语法，而改进到整个库支持AMD将在1.7到2.0版本实现。

最后一个需要注意的问题是如果你希望继续使用Dojo构建系统或者希望迁移老的模块到最新的AMD风格，如果罗列得详细些则会更容易地迁移。请注意，包括Dojo、Dijit和作为依赖的引用：

```
define(["dojo", "dijit", "dojo/cookie", "dijit/Tooltip"], function(dojo, dijit){
    var cookieValue = dojo.cookie("cookieName");
    new dijit.Tooltip(...);
});
```

### AMD模块的设计模式（Dojo）

如果你关注了我之前几篇文章关于设计模式的好处，就会知道这些设计模式能够快速地提升我们实现一般开发问题的整体解决方案的能力。关于AMD设计模式，[John Hann](http://twitter.com/unscriptable)最近给出了一个非常出色的presentation，包含了Singleton, Decorator, Mediator以及其它。如果有机会强烈建议大家去阅读下他的[slides](http://unscriptable.com/code/AMD-module-patterns/)。

这些模式的许多例子可以从下面找到：

#### 装饰模式 ####

```
// mylib/UpdatableObservable: a decorator for dojo/store/Observable
define(['dojo', 'dojo/store/Observable'], function ( dojo, Observable ) {
    return function UpdatableObservable ( store ) {

        var observable = dojo.isFunction(store.notify) ? store :
                new Observable(store);

        observable.updated = function( object ) {
            dojo.when(object, function ( itemOrArray) {
                dojo.forEach( [].concat(itemOrArray), this.notify, this );
            };
        };

        return observable; // makes `new` optional
    };
});


// decorator consumer
// a consumer for mylib/UpdatableObservable

define(['mylib/UpdatableObservable'], function ( makeUpdatable ) {
    var observable, updatable, someItem;
    // ... here be code to get or create `observable`

    // ... make the observable store updatable
    updatable = makeUpdatable(observable); // `new` is optional!

    // ... later, when a cometd message arrives with new data item
    updatable.updated(updatedItem);
});
```

#### 适配器模式 ####

```
// 'mylib/Array' adapts `each` function to mimic jQuery's:
define(['dojo/_base/lang', 'dojo/_base/array'], function (lang, array) {
    return lang.delegate(array, {
        each: function (arr, lambda) {
            array.forEach(arr, function (item, i) {
                lambda.call(item, i, item); // like jQuery's each
            })
        }
    });
});

// adapter consumer
// 'myapp/my-module':
define(['mylib/Array'], function ( array ) {
    array.each(['uno', 'dos', 'tres'], function (i, esp) {
        // here, `this` == item
    });
});
```

### AMD模块和JQuery ###

#### 基础 ####

与Dojo不同，jQuery只有一个文件，虽然有了基于库本身的插件机制，我们可以发现要定义并使用一个AMD模块其实是非常容易的，如下所示：

```
define(['js/jquery.js','js/jquery.color.js','js/underscore.js'],
    function($, colorPlugin, _){
        // Here we've passed in jQuery, the color plugin and Underscore
        // None of these will be accessible in the global scope, but we
        // can easily reference them below.

        // Pseudo-randomize an array of colors, selecting the first
        // item in the shuffled array
        var shuffleColor = _.first(_.shuffle(['#666','#333','#111']));

        // Animate the background-color of any elements with the class
        // 'item' on the page using the shuffled color
        $('.item').animate({'backgroundColor': shuffleColor });

        return {};
        // What we return can be used by other modules
    });
```

不过这个例子缺少了注册的概念。

#### 将JQuery注册为一个Async-compatible模块 ####

JQuery 1.7所支持的核心特性之一就是将JQuery注册成为一个asynchronous模块。有许多兼容的脚本加载器（包括RequireJS以及curl）都可以通过AMD格式加载模块，通过这种方式只需要很少的hacks就可以使工作正常。

由于JQuery的声望，AMD加载器需要顾及库的多个版本在同一个页面中被同时加载,即使你真的不想在同一个时刻加载多个不同的版本。加载器专门考虑了这个问题，或指导他们的用户如何处理关于第三方脚本和他们的库之间的问题。

1.7版本带给我们的新东西就是帮助我们避免第三方代码的一些问题，如在一个页面里意外加载了某个版本的jQuery，而这是页面所有者不希望的。你不希望其它实例妨碍到自己本身，新特性有利于这样的目标的实现。

这种工作方式即使用脚本加载器表明可以通过设定一个属性来支持jQuery多版本，即`define.amd.jQuery`设置为true。更详细的实现细节则更有趣，我们将JQuery注册为一个命名模块，虽然这有一定的风险，因为它可能跟其他可使用AMD的`define()`方法的文件串联在一起，而非使用一个符合AMD模块定义的合适的的串联脚本。

命名的AMD提供了一个针对大多数用例稳健和安全的的安全毯。

```
// Account for the existence of more than one global
// instances of jQuery in the document, cater for testing
// .noConflict()

var jQuery = this.jQuery || "jQuery",
$ = this.$ || "$",
originaljQuery = jQuery,
original$ = $,
amdDefined;

define(['jquery'] , function ($) {
    $('.items').css('background','green');
    return function () {};
});

// The very easy to implement flag stating support which
// would be used by the AMD loader
define.amd = {
    jQuery: true
};
```

#### 巧妙的JQuery插件 ####

最近讨论了关于如何使用Universal Module Definition([UMD](https://github.com/umdjs))模式来编写jQuery插件的一些[想法和例子](http://coding.smashingmagazine.com/2011/10/11/essential-jquery-plugin-patterns/)。UMD定义模块使之能够同时在客户端和服务器端工作，就像目前所有可见的流行脚本加载器一样。虽然这是一个新的领域，很多概念都在处在定稿之中，但在下面标题为AMD&&CommonJS的段落可以随意看看代码实例，并让我知道有哪些我们可以做的更好的地方。

### 那些脚本加载器和框架支持AMD？ ###

#### 浏览器端： ####

- RequireJS http://requirejs.org
- curl.js http://github.com/unscriptable/curl
- bdLoad http://bdframework.com/bdLoad
- Yabble http://github.com/jbrantly/yabble
- PINF http://github.com/pinf/loader-js
- (and more)

#### 服务器端： ####

- RequireJS http://requirejs.org
- PINF http://github.com/pinf/loader-js

### AMD 结论 ###

以上细碎的例子阐述了AMD模块是多么有用，但更希望向大家提供理解他们是如何工作的基础。

你或许会对目前现实中哪些大型应用程序使用AMD作为他们架构的一部分感兴趣。这些包括[IBM](http://www.ibm.com/)、[BBC iPlayer](http://www.bbc.co.uk/iplayer/)，这突出表明了在企业级别开发人员是多么严肃地对待这个格式。

至于要知道许多开发人员在他们的程序中选择AMD模块的原因，你应该对James Burke写的[文章](http://tagneto.blogspot.com/2011/04/on-inventing-js-module-formats-and.html)感兴趣。

## CommonJS ##

> 针对服务器端优化的模块格式

[CommonJS](http://www.commonjs.org/)是一个志愿工作组，目标是设计、原型化以及标准化Javascript API。迄今为止，他们已经尝试认可了[模块](http://www.commonjs.org/specs/modules/1.0/)以及[包](http://wiki.commonjs.org/wiki/Packages/1.0)标准。CommonJS模块的提案为服务器端声明模块时指定一个简单的API，而不像AMD尝试覆盖更广泛的关注点，例如io，文件系统，promise等。

### 入门 ###

从结构的角度来看，CJS模块是可复用的JavaScript片段，其中exports提供具体的对象，使之对任何独立的代码都是可用的 - 通常这样的模块没有函数包装（所以你不会看到例如define的使用）。

从更高层次来说，它们基本上仅包含两个主要部分：一个包含模块对象的自由变量命名为exports，提供给其他模块；一个require函数，模块可以通过require导入其它模块。

### 理解CJS: require()以及exports ###

```
// package/lib is a dependency we require
var lib = require('package/lib');

// some behaviour for our module
function foo(){
    lib.log('hello world!');
}

// export (expose) foo to other modules
exports.foo = foo;
```

### exports的基本使用 ###

```
// define more behaviour we would like to expose
function foobar(){
        this.foo = function(){
                console.log('Hello foo');
        }

        this.bar = function(){
                console.log('Hello bar');
        }
}

// expose foobar to other modules
exports.foobar = foobar;


// an application consuming 'foobar'

// access the module relative to the path
// where both usage and module files exist
// in the same directory

var foobar = require('./foobar').foobar,
    test   = new foobar();

test.bar(); // 'Hello bar'
```

### 与AMD等同的第一个CJS例子 ###

```
define(['package/lib'], function(lib){

    // some behaviour for our module
    function foo(){
        lib.log('hello world!');
    }

    // export (expose) foo for other modules
    return {
        foobar: foo
    };
});
```

### 使用多个依赖 ###

#### app.js ####

```
var modA = require('./foo');
var modB = require('./bar');

exports.app = function(){
    console.log('Im an application!');
}

exports.foo = function(){
    return modA.helloWorld();
}
```

#### bar.js ####

```
exports.name = 'bar';
```

#### foo.js ####

```
require('./bar');
exports.helloWorld = function(){
    return 'Hello World!!''
}
```

#### 浏览器端： ####

- curl.js http://github.com/unscriptable/curl
- SproutCore 1.1 http://sproutcore.com
- PINF http://github.com/pinf/loader-js
- (and more)

#### 服务器端： ####

- Node http://nodejs.org
- Narwhal https://github.com/tlrobinson/narwhal
- Persevere http://www.persvr.org/
- Wakanda http://www.wakandasoft.com/

### CJS适合浏览器吗？ ###

许多开发人员认为CommonJS更适合服务器端开发，其中一个原因是存在一些争议，即哪种格式在前Harmony时代应该作为一个事实上的标准并被使用而往前发展。反对CJS的观点包括在Javascript里CommonJS的许多API符合面向服务器的特性但是在浏览器端基本不能实现。例如，io、系统，js可以认为实际上无法实现它们的功能特性。

但也有种说法，了解CJS模块的结构无论如何是非常有用的，我们可以更好地鉴别他们是如何定义通用模块。即在客户端和服务器端都有的模块包括验证、转换以及模板引擎。许多开发人员都在努力某种格式，它可以对CJS进行优化，当某个模块可以在服务器端使用而在AMD里没有同样的模块时则可以使用AMD。

因为AMD模块能够使用插件从而使定义更加细粒度的东西例如构造函数和函数变得有意义。而CJS模块只能定义对象，如果你尝试从它们那获得构造函数可能会觉得非常乏味。

虽然这超出了本文的范畴，你可能已经意识到当讨论AMD和CJS时提到的`require`方法是有不同的类型的。

对类似的命名约定的关注注定是个混乱，当前工作组对全局的require功能的优点产生了分歧。John Hann在这里的建议是不应该称呼它为“require”，这可能导致告知用户区别全局以及内部require的的目标失败，若将全局加载器的方法命名为其它（例如：库的名字）会更有意义。因为这个原因，像curl.js就使用curl()来代替require。

#### 相关阅读 ###

- [Demystifying CommonJS Modules](http://dailyjs.com/2010/10/18/modules/)
- [JavaScript Growing Up](http://www.slideshare.net/davidpadbury/javascript-growing-up)
- [The RequireJS Notes On CommonJS](http://requirejs.org/docs/commonjs.html)
- [Taking Baby Steps With Node.js And CommonJS Creating Custom Modules](http://elegantcode.com/2011/02/04/taking-baby-steps-with-node-js-commonjs-and-creating-custom-modules/)
- [Asynchronous CommonJS Modules for the Browser](http://www.sitepen.com/blog/2010/07/16/asynchronous-commonjs-modules-for-the-browser-and-introducing-transporter/)
- [The CommonJS Mailing List](http://groups.google.com/group/commonjs)

## AMD 和 CommonJS ##
> 竞争，但都是有效的标准

虽然这篇文章的重点是使用AMD和CJS，事实上这两种格式都是有效的、有用的。

AMD采用了浏览器优先的开发思路、异步行为以及简化向后兼容，但它没有任何关于文件I/O的概念。它支持对象，函数，构造函数，字符串，JSON以及很多其他类型的模块，原生地运行于浏览器，令人难以置信地灵活。

CommonJS选择了另外一条服务器端优先的方式，使用同步行为，就像John Hann所指出的一样没有全局的负担,它试图适应未来的趋势（在服务器端）。我们其实想表达的意思就是因为CJS支持未封装的模块，感觉上它更接近ES.next/Harmony规范，放弃AMD强制使用的define封装。然而，CJS仅仅支持对象作为模块。

尽管其他的模块格式的思路可能会令人生畏，但你应该会对使用hybrid AMD、CJS以及Univeral AMD/CJS模块的实例感兴趣。

### 基本的AMD混合格式 (John Hann)  ###

```
define( function (require, exports, module){

    var shuffler = require('lib/shuffle');

    exports.randomize = function( input ){
        return shuffler.shuffle(input);
    }
});
```

### AMD/CommonJS 统一模块定义（变种2， [UMDjs](https://github.com/umdjs/umd)） ###

```
/**
 * exports object based version, if you need to make a
 * circular dependency or need compatibility with
 * commonjs-like environments that are not Node.
 */
(function (define) {
    //The 'id' is optional, but recommended if this is
    //a popular web library that is used mostly in
    //non-AMD/Node environments. However, if want
    //to make an anonymous module, remove the 'id'
    //below, and remove the id use in the define shim.
    define('id', function (require, exports) {
        //If have dependencies, get them here
        var a = require('a');

        //Attach properties to exports.
        exports.name = value;
    });
}(typeof define === 'function' && define.amd ? define : function (id, factory) {
    if (typeof exports !== 'undefined') {
        //commonjs
        factory(require, exports);
    } else {
        //Create a global function. Only works if
        //the code does not have dependencies, or
        //dependencies fit the call pattern below.
        factory(function(value) {
            return window[value];
        }, (window[id] = {}));
    }
}));
```

### 可扩展的UMD插件 ###

#### core.js ####

```
// Module/Plugin core
// Note: the wrapper code you see around the module is what enables
// us to support multiple module formats and specifications by
// mapping the arguments defined to what a specific format expects
// to be present. Our actual module functionality is defined lower
// down, where a named module and exports are demonstrated.

;(function ( name, definition ){
  var theModule = definition(),
      // this is considered "safe":
      hasDefine = typeof define === 'function' && define.amd,
      // hasDefine = typeof define === 'function',
      hasExports = typeof module !== 'undefined' && module.exports;

  if ( hasDefine ){ // AMD Module
    define(theModule);
  } else if ( hasExports ) { // Node.js Module
    module.exports = theModule;
  } else { // Assign to common namespaces or simply the global object (window)
    (this.jQuery || this.ender || this.$ || this)[name] = theModule;
  }
})( 'core', function () {
    var module = this;
    module.plugins = [];
    module.highlightColor = "yellow";
    module.errorColor = "red";

  // define the core module here and return the public API

  // this is the highlight method used by the core highlightAll()
  // method and all of the plugins highlighting elements different
  // colors
  module.highlight = function(el,strColor){
    // this module uses jQuery, however plain old JavaScript
    // or say, Dojo could be just as easily used.
    if(this.jQuery){
      jQuery(el).css('background', strColor);
    }
  }
  return {
      highlightAll:function(){
        module.highlight('div', module.highlightColor);
      }
  };

});
```

#### myExtension.js ####

```
;(function ( name, definition ) {
    var theModule = definition(),
        hasDefine = typeof define === 'function',
        hasExports = typeof module !== 'undefined' && module.exports;

    if ( hasDefine ) { // AMD Module
        define(theModule);
    } else if ( hasExports ) { // Node.js Module
        module.exports = theModule;
    } else { // Assign to common namespaces or simply the global object (window)


        // account for for flat-file/global module extensions
        var obj = null;
        var namespaces = name.split(".");
        var scope = (this.jQuery || this.ender || this.$ || this);
        for (var i = 0; i < namespaces.length; i++) {
            var packageName = namespaces[i];
            if (obj && i == namespaces.length - 1) {
                obj[packageName] = theModule;
            } else if (typeof scope[packageName] === "undefined") {
                scope[packageName] = {};
            }
            obj = scope[packageName];
        }

    }
})('core.plugin', function () {

    // define your module here and return the public API
    // this code could be easily adapted with the core to
    // allow for methods that overwrite/extend core functionality
    // to expand the highlight method to do more if you wished.
    return {
        setGreen: function ( el ) {
            highlight(el, 'green');
        },
        setRed: function ( el ) {
            highlight(el, errorColor);
        }
    };

});
```

#### app.js ####

```
$(function(){

    // the plugin 'core' is exposed under a core namespace in
    // this example which we first cache
    var core = $.core;

    // use then use some of the built-in core functionality to
    // highlight all divs in the page yellow
    core.highlightAll();

    // access the plugins (extensions) loaded into the 'plugin'
    // namespace of our core module:

    // Set the first div in the page to have a green background.
    core.plugin.setGreen("div:first");
    // Here we're making use of the core's 'highlight' method
    // under the hood from a plugin loaded in after it

    // Set the last div to the 'errorColor' property defined in
    // our core module/plugin. If you review the code further down
    // you'll see how easy it is to consume properties and methods
    // between the core and other plugins
    core.plugin.setRed('div:last');
});
```

## ES Harmony ##
> 未来的模块

[TC39](http://www.ecma-international.org/memento/TC39.htm)，标准本身受ECMAScript语法和语义定义所约束，其下一个迭代定义由一群非常聪明的程序员所设想。其中的许多开发人员（例如[Alex Russel](http://twitter.com/slightlylate)）一直在持续关注着过去几年大规模开发中Javascript使用的变革，也敏锐地意识到编写更加模块化的JS这一更好的语言特性的需求。

基于此，目前已经有许多令人兴奋的提案加入到这门语言中，包括灵活的、可在客户端以及服务器端工作的[模块](http://wiki.ecmascript.org/doku.php?id=harmony:modules)，[模块加载器](http://wiki.ecmascript.org/doku.php?id=harmony:module_loaders)以及[其他](http://wiki.ecmascript.org/doku.php?id=harmony:proposals)。在这一节，我将为你展示一些ES.next中针对模块语法的代码示例，你可以抢先体验到他们将会是什么。

> **注意：** 尽管Harmony仍然处于提案阶段，你仍已经可以尝试部分ES.next的特性。感谢Google's [Traceur](http://code.google.com/p/traceur-compiler/)编译器，它已经有了对编写模块化JavaScript的原生支持。为了能够快速安装和运行Traceur，请阅读[开始](http://code.google.com/p/traceur-compiler/wiki/GettingStarted)指南。这里也有一个JSConf的[presentation](http://traceur-compiler.googlecode.com/svn/branches/v0.10/presentation/index.html)值得一看，如果你对这个项目有兴趣并进一步了解的话。

### 包含Imports和Exports的模块 ###

如果你已经阅读了上面关于AMD和CJS的部分，你应该已经熟悉了模块依赖的概念以及模块输出（或者，我们允许其他模块使用的公开的API/变量）。在ES.next，这些概念被建议成一种略微简洁的方式，其中依赖被设定使用`import`关键字。而`export`与你期望的区别也不会很大，我想大多开发人员看了下面的代码之后会立刻“把握”到它的。

* **import** 声明作为一个局部变量绑定到模块的exports，可以重命名以避免命名碰撞/冲突。

* **export** 声明了一个模块的本地绑定，对外部是可见的。因此其它的模块可以读取这个export，但是不能修改它们。有趣的是，模块可以export子模块，虽然不能export已经在其它地方定义过的模块。你也可以重命名export从而使它们的外部名字与它们的本地名字不同。

```
module staff{
    // specify (public) exports that can be consumed by
    // other modules
    export var baker = {
        bake: function( item ){
            console.log('Woo! I just baked ' + item);
        }
    }
}

module skills{
    export var specialty = "baking";
    export var experience = "5 years";
}

module cakeFactory{

    // specify dependencies
    import baker from staff;

    // import everything with wildcards
    import * from skills;

    export var oven = {
        makeCupcake: function( toppings ){
            baker.bake('cupcake', toppings);
        },
        makeMuffin: function( mSize ){
            baker.bake('muffin', size);
        }
    }
}
```

### 从远程源加载的模块 ###

模块提案也迎合了基于远程的一类模块，简化了从外部地址加载模块的过程。这是一个获取并利用我们之前定义的模块的例子。

```
module cakeFactory from 'http://addyosmani.com/factory/cakes.js';
cakeFactory.oven.makeCupcake('sprinkles');
cakeFactory.oven.makeMuffin('large');
```

### 模块加载器API ###

模块加载器建议在一个高度可控的上下文中为加载模块描述一个动态的API。加载器上的设计支持包括`load（url，moduleInstance，error）`来加载模块,`createModule（object，globalModuleReferences）`以及[其它](http://wiki.ecmascript.org/doku.php?id=harmony:module_loaders)。这是在我们最初定义的模块中实现的动地态加载的另外一个例子。注意最后一个例子的不同之处，我们从一个远程源获取模块，而模块加载器能够更好地适应动态上下文。

```
Loader.load('http://addyosmani.com/factory/cakes.js',
    function(cakeFactory){
        cakeFactory.oven.makeCupcake('chocolate');
    });
```

### 针对服务器端的类CommonJS模块 ###

针对面向服务器的开发人员，ES.next建议的模块系统并不局限于在浏览器端查找模块。例如下面，你能够看到一个建议在服务器端使用的类CJS的模块。

```
// io/File.js
export function open(path) { ... };
export function close(hnd) { ... };
```

```
// compiler/LexicalHandler.js
module file from 'io/File';

import { open, close } from file;
export function scan(in) {
    try {
        var h = open(in) ...
    }
    finally { close(h) }
}
```

```
module lexer from 'compiler/LexicalHandler';
module stdlib from '@std';

//... scan(cmdline[0]) ...
```

### 含有构造器、Getters、Setters的类 ###

类的概念一直都是伴随着较真、争议的问题。目前为止我们在处理这个问题时既没有回到JavaScript的[原型](http://javascript.crockford.com/prototypal.html)特性也没有通过使用提供class定义能力、能够表现出与原型一样行为的框架或者抽象。

在Harmony，类与constructors和真正的私有化一起作为语言的一部分。在下面的例子中，包含了一些行内注释用来帮助理解类是如何被结构化的，但是你也可能会注意到这里缺少了'function'这个单词。这并不是一个typo错误：TC39已经意识到并努力减少`function`关键字的滥用，并希望这样做能够简化我们的代码。

```
class Cake{

    // We can define the body of a class' constructor
    // function by using the keyword 'constructor' followed
    // by an argument list of public and private declarations.
    constructor( name, toppings, price, cakeSize ){
        public name = name;
        public cakeSize = cakeSize;
        public toppings = toppings;
        private price = price;

    }

    // As a part of ES.next's efforts to decrease the unnecessary
    // use of 'function' for everything, you'll notice that it's
    // dropped for cases such as the following. Here an identifier
    // followed by an argument list and a body defines a new method

    addTopping( topping ){
        public(this).toppings.push(topping);
    }

    // Getters can be defined by declaring get before
    // an identifier/method name and a curly body.
    get allToppings(){
        return public(this).toppings;
    }

    get qualifiesForDiscount(){
        return private(this).price > 5;
    }

    // Similar to getters, setters can be defined by using
    // the 'set' keyword before an identifier
    set cakeSize( cSize ){
        if( cSize < 0 ){
            throw new Error('Cake must be a valid size -
            either small, medium or large');
        }
        public(this).cakeSize = cSize;
    }

}
```

### ES Harmony 结论 ###

正如你所看到的，ES.next即将带着许多令人兴奋的新特性到来。尽管目前Traceur能够在某种程度上体验新特性，但记住它可能不是为使用Harmony而规划到你的系统的最好的主意。因为这存在许多风险，例如规范的修改以及在跨浏览器级别潜在的失败（例如IE9需要一段时间才会消亡），因此你最好等到我们规范定稿以及覆盖到AMD（浏览器端）和CJS（服务器端）。

#### 相关阅读 ####

- [A First Look At The Upcoming JavaScript Modules](http://www.2ality.com/2011/03/first-look-at-upcoming-javascript.html)
- [David Herman On JavaScript/ES.Next (Video)](http://blog.mozilla.com/dherman/2011/02/23/my-js-meetup-talk/)
- [ES Harmony Module Proposals](http://wiki.ecmascript.org/doku.php?id=harmony:modules)
- [ES Harmony Module Semantics/Structure Rationale](http://wiki.ecmascript.org/doku.php?id=harmony:modules_rationale)
- [ES Harmony Class Proposals](http://wiki.ecmascript.org/doku.php?id=harmony:classes)

## 结论和进一步深入 ##
> 回顾

在这篇文章中我们回顾了许多使用现代模块格式用来编写模块化Javascript的可用选项。这些格式相比传统的模块模式拥有一些特性，其中包括：避免开发人员为每一个自身创建的模块新建一个全局变量，更好地支持静态以及动态依赖管理，提升了脚本加载器的能力，更好的服务器端模块兼容性，等等。

简而言之，我推荐大家尝试下今天推荐的格式，因为这些格式提供了很强的功效和灵活性，从而在我们构建基于可复用的功能块程序时能够帮助到我们。

差不多到这里为止了。如果你对今天谈到的话题有任何疑问，你可以[twitter](http://twitter.com/addyosmani)我，我很愿意帮助到你。

这篇文章的技术review使用了[Diigo](http://www.diigo.com/)(for [Google Chrome](http://www.diigo.com/tools/chrome_extension)).Diigo是一个免费的工具，允许你在web上的任何一个地方加入评论以及文本高亮。如果你有任何更正或者扩展需要提供，也请使用Diigo(或者GitHub [gist](http://gist.github.com/)),我会认真处理每一个你们发的消息。
