# 用AMD，CommonJS 和 ES Harmony编写模块化的JavaScript #

## 模块化 ##

> 解耦你的程序的重要性

当我们说一（某）个程序是模块化的，通常我们指它是由一组存放在模块中的高度解耦的独立功能片段所组成。正如你所了解的，[松耦合](http://arguments.callee.info/2009/05/18/javascript-design-patterns--mediator/)通过消除可能的依赖性从而促进应用程序可维护性变得更简单。当松耦合被高效地实现时，可以很容易地观察到系统的一部分变化是如何影响到另一部分的。

然而与传统的编程语言不用的是，当前版本的JavaScript（[ECMA-262]）并没有为提供一个代码清晰的，有条理的导入此类代码模块的方式。规范里令人担忧的一点是之前并不需要非常伟大的想法，直到最近几年对有条理的JavaScript程序的需求变得愈加明显。

Instead, developers at present are left to fall back on variations of the [module](http://www.adequatelygood.com/2010/3/JavaScript-Module-Pattern-In-Depth) or [object literal](http://blog.rebeccamurphey.com/2009/10/15/using-objects-to-organize-your-code) patterns. With many of these, module scripts are strung together in the DOM with namespaces being described by a single global object where it's still possible to incur naming collisions in your architecture. There's also no clean way to handle dependency management without some manual effort or third party tools.

取而代之的是，在当前开发者仍然只能回到使用模块或者对象字面量模式等替代方法上。*在这当中，模块脚本被一个单一全局对象命名空间在Dom中被串联起来*。在缺少一些人工的处理或者第三方工具时，这也不是一个合理的处理依赖管理的方式。

虽然这些问题的原生解决方案将会在ES Harmony中到来，好消息就是编写模块化的JavaScript从来都不是容易的但你可以从今天开始做这件事情。

在这篇文章中，我们会观察三种编写模块化JavaScript的格式：**AMD**,**CommonJS**以及下一个JavaScript版本的提案，**Harmony**。

## 前奏 ##

> 关于脚本加载器的说明

很难在不谈论脚本加载器的情况下讨论AMD和CommonJS模块。目前，脚本加载是达到一个目标的方法，这个目标就是模块化JavaScript使之能够在今天的程序中使用。为了这点，很遗憾，使用兼容的脚本加载器是必要的。为了从本文中获得最大收益，我建议对流行的脚本加载工具是如何工作的先做一个基本了解从而通过上下文对模块格式的解释更能理解。

在AMD和CJS格式中，有很多很好的加载器用来处理模块加载，而我的个人选择是 [RequireJS](http://requirejs.org/) 和[curl.js](https://github.com/unscriptable/curl)。关于这些工具的完整教程不在本文讨论的范围之内，但我强烈建议阅读John Hann关于curl.js的文章以及James Burke的RequireJS的API文档以了解更多。

从生产角度来看，在与这些模块一起工作时，使用优化工具（例如RequireJS优化器）用来串联脚本在部署时非常值得推荐。有趣地是，配合Almond AMD shim，RequireJS不需要在部署站点上合并起来，你可以认为脚本加载器可以非常容易的切换到开发环境。

也就是说，James Burke有可能会，将这些笔记记在脑子里，让我们开始吧。

## AMD ##

> 一个在浏览器端编写模块化JavaScript的格式

AMD（异步模块定义）格式的总体目标是提供一个当今程序员可以使用的模块化JavaScript解决方案。它诞生于Dojo使用XHR+ eval的现实经历，这个格式的支持者希望未来的解决方案避免那些在过去遭遇到的弱点

AMD模块格式本身是一个提议，用于定义模块使模块以及依赖都可以被异步加载。它有一系列显著的优势，包括两者都是异步的以及删除一般在代码和模块识别之间可能的高耦合性天生的高度灵活性。许多开发者喜欢使用它并且认为这是一个被提及的面向ES Harmony模块系统的可靠的跳板。

AMD最开始作为一个在CommonJS目录中模块格式的规范草案，但是由于它无法达成完全共识，格式的进一步发展就转移到了amdjs组。

Today it's embraced by projects including Dojo (1.7), MooTools (2.0), Firebug (1.8) and even jQuery (1.7). Although the term CommonJS AMD format has been seen in the wild on occasion, it's best to refer to it as just AMD or Async Module support as not all participants on the CJS list wished to pursue it.

今天它被囊括在包括Dojo（1.7），MooTools(2.0),Firebug(1.8)甚至JQuery（1.7）等多个项目中。*CommonJS的术语AMD的格式虽然已在野外场合上看到，这是最好的指空肠名单上的所有参与者不希望追求它只是AMD或异步模块支持。*

> **Note:** There was a time when the proposal was referred to as Modules Transport/C, however as the spec wasn't geared for transporting existing CJS modules, but rather, for defining modules it made more sense to opt for the AMD naming convention.

> **说明：** .。。。

### Getting Started With Modules ###
### 模块入门 ###

在这里需要关注的两个关键概念是用于促进模块定义的define方法以及用于处理依赖加载的require方法。根据提议，define使用以下签名明码来定义命名的或者未命名的模块：

`
define(
    module_id /*optional*/, 
    [dependencies] /*optional*/,
    definition function /*function for instantiating the module or object*/
);
`

正如你在行内注释了解到的一样，module_id是一个可选的参数，这个参数通常在非AMD的串联工具被使用时才需要（也会有这个参数非常有用的个例存在）。当删掉这个参数时，我们称它为匿名模块。

当我们使用匿名模块时，模块识别的原则就是DRY（Don't Repeat Yourself)，通过使模块碎片化来避免文件名和代码的复制。因为代码更具有可移植性，它能够很容易的移动到其它位置(或者文件系统）而不需要改变代码本身或者只改变代码的ID。在简单的包或不使用包的情况下module_id相当于文件路径。开发者可以在多种环境下运行同一段代码而仅仅需要使用一个AMD优化器，这个优化器在CommonJS例如r.js环境下工作。

回到define定义上，dependencies参数表现为一个依赖数组，这个依赖数组是你定义的模块所需要的。第三个参数（'definition function'）则是一个函数，用来执行模块的实例化工作。一个标准的模块应该被定义成以下形式：

### 理解AMD: define() ###

`
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
`

require on the other hand is typically used to load code in a top-level JavaScript file or within a module should you wish to dynamically fetch dependencies. An example of its usage is:

另一方面，require通常用于加载处于顶层的JavaScript文件代码或你希望的在模块内部动态地获取依赖。一个使用的例子如下：

### Understanding AMD: require() ###
### 理解AMD:require() ###

`// Consider 'foo' and 'bar' are two external modules
// In this example, the 'exports' from the two modules loaded are passed as
// function arguments to the callback (foo and bar)
// so that they can similarly be accessed
 
require(['foo', 'bar'], function ( foo, bar ) {
        // rest of your code here
        foo.doSomething();
});`

### Dynamically-loaded Dependencies ###
### 动态加载的依赖 ###

`define(function ( require ) {
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
});`

### Understanding AMD: plugins ###
### 理解AMD: 插件 ###

The following is an example of defining an AMD-compatible plugin:

`// With AMD, it's possible to load in assets of almost any kind
// including text-files and HTML. This enables us to have template
// dependencies which can be used to skin components either on
// page-load or dynamically.
 
define(['./templates', 'text!./template.md','css!./template.css'],
    function( templates, template ){
        console.log(templates);
        // do some fun template stuff here.
    }
});`

> **Note**: Although css! is included for loading CSS dependencies in the above example, it's important to remember that this approach has some caveats such as it not being fully possible to establish when the CSS is fully loaded. Depending on how you approach your build, it may also result in CSS being included as a dependency in the optimized file, so use CSS as a loaded dependency in such cases with caution.

> **Note**: 尽管上面的例子中为了加载css依赖而将css!包括进来，但切记这种方法存在许多注意事项，例如当CSS完全加载时并能确保完全渲染出来。这依赖于你构建的过程，**待写**

### Loading AMD Modules Using require.js ###

### 使用require.js加载AMD模块 ###

`require(['app/myModule'], 
    function( myModule ){
        // start the main module which in-turn
        // loads other modules
        var module = new myModule();
        module.doStuff();
});`

### Loading AMD Modules Using curl.js ###

### 使用curl.js加载AMD模块 ###

`curl(['app/myModule.js'], 
    function( myModule ){
        // start the main module which in-turn
        // loads other modules
        var module = new myModule();
        module.doStuff();
});`

### Modules With Deferred Dependencies ###

### 具备Deferred依赖的模块 ###

`// This could be compatible with jQuery's Deferred implementation,
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
});`

### Why Is AMD A Better Choice For Writing Modular JavaScript? ###

### 为什么说AMD是编写模块化的JavaScript的好选择？ ###

- 为如何处理定义灵活的模块提供了一个明确的建议。
- 语法比目前我们依靠的全局命名空间以及**&amp;script&amp;**标签等解决方案清晰。有一个干净的方式用来声明独立的模块以及可能的依赖。
- 模块定义是经过封装的，帮助我们避免污染全局命名空间。
- 比起其他可选的解决方案（例如很快会介绍的CommonJS）工作得更好。在跨域、本地、以及debug方面没有问题。使用时不需要依赖服务器端的工具。大多数AMD的加载器在浏览器端加载模块时并不需要build过程。
- 提供了 'transport'方法用于将多个模块包含在一个文件当中。其他的方法例如CommonJS却必须同意一个transport格式。
- 当需要是可以懒加载脚本。

### Related Reading ###
### 相关阅读 ###
The RequireJS Guide To AMD
What's the fastest way to load AMD modules?
AMD vs. CJS, what's the better format?
AMD Is Better For The Web Than CommonJS Modules
The Future Is Modules Not Frameworks
AMD No Longer A CommonJS Specification
On Inventing JavaScript Module Formats And Script Loaders
The AMD Mailing List

### AMD Modules With Dojo ###
### AMD模块和Dojo ###

在Dojo中定义兼容AMD的模块相当简单。As per above，定义任意模块，依赖放在一个数组中作为第一个参数并且提供一个回调（工厂），一旦依赖加载完毕这个回调将运行这个模块。例如：

`define(["dijit/Tooltip"], function( Tooltip ){
    //Our dijit tooltip is now available for local use
    new Tooltip(...);
});`

请注意模块的匿名特性使得模块可以被Dojo异步加载器、RequireJS或者标准的dojo.require()模块加载器等你习惯的模块加载器使用。

For those wondering about module referencing, there are some interesting gotchas that are useful to know here. Although the AMD-advocated way of referencing modules declares them in the dependency list with a set of matching arguments, this isn't supported by the Dojo 1.6 build system - it really only works for AMD-compliant loaders. e.g:

`define(["dojo/cookie", "dijit/Tooltip"], function( cookie, Tooltip ){
    var cookieValue = cookie("cookieName"); 
    new Tree(...); 
});`

This has many advances over nested namespacing as modules no longer need to directly reference complete namespaces every time - all we require is the 'dojo/cookie' path in dependencies, which once aliased to an argument, can be referenced by that variable. This removes the need to repeatedly type out 'dojo.' in your applications.

> **Note**: Although Dojo 1.6 doesn't officially support user-based AMD modules (nor asynchronous loading), it's possible to get this working with Dojo using a number of different script loaders. At present, all Dojo core and Dijit modules have been transformed to the AMD syntax and improved overall AMD support will likely land between 1.7 and 2.0.

The final gotcha to be aware of is that if you wish to continue using the Dojo build system or wish to migrate older modules to this newer AMD-style, the following more verbose version enables easier migration. Notice that dojo and dijit and referenced as dependencies too:

`define(["dojo", "dijit", "dojo/cookie", "dijit/Tooltip"], function(dojo, dijit){
    var cookieValue = dojo.cookie("cookieName");
    new dijit.Tooltip(...);
});`

### AMD Module Design Patterns (Dojo) ###

If you've followed any of my previous posts on the benefits of design patterns, you'll know that they can be highly effective in improving how we approach structuring solutions to common development problems. John Hann recently gave an excellent presentation about AMD module design patterns covering the Singleton, Decorator, Mediator and others. I highly recommend checking out his slides if you get a chance.

Some samples of these patterns can be found below:

#### Decorator pattern: ####

`// mylib/UpdatableObservable: a decorator for dojo/store/Observable
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
});`

#### Adapter pattern ####

`// 'mylib/Array' adapts `each` function to mimic jQuery's:
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
});`

### AMD Modules With jQuery ###

#### The Basics ####

Unlike Dojo, jQuery really only comes with one file, however given the plugin-based nature of the library, we can demonstrate how straight-forward it is to define an AMD module that uses it below.

`define(['js/jquery.js','js/jquery.color.js','js/underscore.js'],
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
    });`

There is however something missing from this example and it's the concept of registration.

#### Registering jQuery As An Async-compatible Module ####

One of the key features that landed in jQuery 1.7 was support for registering jQuery as an asynchronous module. There are a number of compatible script loaders (including RequireJS and curl) which are capable of loading modules using an asynchronous module format and this means fewer hacks are required to get things working.

As a result of jQuery's popularity, AMD loaders need to take into account multiple versions of the library being loaded into the same page as you ideally don't want several different versions loading at the same time. Loaders have the option of either specifically taking this issue into account or instructing their users that there are known issues with third party scripts and their libraries.

What the 1.7 addition brings to the table is that it helps avoid issues with other third party code on a page accidentally loading up a version of jQuery on the page that the owner wasn't expecting. You don't want other instances clobbering your own and so this can be of benefit.

The way this works is that the script loader being employed indicates that it supports multiple jQuery versions by specifying that a property, define.amd.jQuery is equal to true. For those interested in more specific implementation details, we register jQuery as a named module as there is a risk that it can be concatenated with other files which may use AMD's define() method, but not use a proper concatenation script that understands anonymous AMD module definitions.

The named AMD provides a safety blanket of being both robust and safe for most use-cases.

`// Account for the existence of more than one global 
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
};`

#### Smarter jQuery Plugins ####

I've recently discussed some ideas and examples of how jQuery plugins could be written using Universal Module Definition (UMD) patterns here. UMDs define modules that can work on both the client and server, as well as with all popular script loaders available at the moment. Whilst this is still a new area with a lot of concepts still being finalized, feel free to look at the code samples in the section title AMD && CommonJS below and let me know if you feel there's anything we could do better.

### What Script Loaders & Frameworks Support AMD? ###

#### In-browser: ####

- RequireJS http://requirejs.org
- curl.js http://github.com/unscriptable/curl
- bdLoad http://bdframework.com/bdLoad
- Yabble http://github.com/jbrantly/yabble
- PINF http://github.com/pinf/loader-js
- (and more)

#### Server-side: ####

- RequireJS http://requirejs.org
- PINF http://github.com/pinf/loader-js

### AMD Conclusions ###
### AMD 结论 ###

The above are very trivial examples of just how useful AMD modules can truly be, but they hopefully provide a foundation for understanding how they work.

You may be interested to know that many visible large applications and companies currently use AMD modules as a part of their architecture. These include IBM and the BBC iPlayer, which highlight just how seriously this format is being considered by developers at an enterprise-level.

For more reasons why many developers are opting to use AMD modules in their applications, you may be interested in this post by James Burke.

## CommonJS ##
## CommonJS ##

> A Module Format Optimized For The Server

> 针对服务器端优化的模块格式

CommonJS are a volunteer working group which aim to design, prototype and standardize JavaScript APIs. To date they've attempted to ratify standards for both modules and packages. The CommonJS module proposal specifies a simple API for declaring modules server-side and unlike AMD attempts to cover a broader set of concerns such as io, filesystem, promises and more.

CommonJS是一个志愿者工作组，目标是设计，原型化以及标准化Javascript API。迄今为止，他们已经尝试认可了模块以及包标准。CommonJS模块的提案为服务器端声明模块时指定一个简单的API而不像AMD尝试覆盖了更广泛的关注点，例如io，文件系统，promise等。

### Getting Started ###
### 入门 ###

从结构的视图来看，CJS模块是可重复使用的JavaScript片段，其中exports提供具体的对象，使之对任何独立的代码都是可用的 - 通常这样的模块没有函数包装（所以你不会看到例如define的使用）。

从更高层次来说，它们基本上仅包含两个主要部分：一个包含模块对象的自由变量命名为exports，提供给其他模块；一个require函数，模块可以通过require导入其它模块。

### Understanding CJS: require() and exports ###
### 理解CJS: require()以及exports ###

`// package/lib is a dependency we require
var lib = require('package/lib');
 
// some behaviour for our module
function foo(){
    lib.log('hello world!');
}
 
// export (expose) foo to other modules
exports.foo = foo;`

### Basic consumption of exports ###
### exports的基本使用 ###
 
`// define more behaviour we would like to expose
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
 
test.bar(); // 'Hello bar'`
 
### AMD-equivalent Of The First CJS Example ###
### 与AMD等价的第一个CJS例子 ###

`define(['package/lib'], function(lib){
 
    // some behaviour for our module
    function foo(){
        lib.log('hello world!');
    } 
 
    // export (expose) foo for other modules
    return {
        foobar: foo
    };
});`

### Consuming Multiple Dependencies ###
### 使用多个依赖 ###

#### app.js ####

`var modA = require('./foo');
var modB = require('./bar');
 
exports.app = function(){
    console.log('Im an application!');
}
 
exports.foo = function(){
    return modA.helloWorld();
}`

#### bar.js ####

`exports.name = 'bar';`

#### foo.js ####

`require('./bar');
exports.helloWorld = function(){
    return 'Hello World!!''
}`

### What Loaders & Frameworks Support CJS? ###
### 那些加载器和框架支持CJS？ ###

#### In-browser: ####

- curl.js http://github.com/unscriptable/curl
- SproutCore 1.1 http://sproutcore.com
- PINF http://github.com/pinf/loader-js
- (and more)

#### Server-side: ####

- Nodehttp://nodejs.org
- Narwhal https://github.com/tlrobinson/narwhal
- Perseverehttp://www.persvr.org/
- Wakandahttp://www.wakandasoft.com/

### Is CJS Suitable For The Browser? ###
### CJS适合浏览器吗？ ###

There are developers that feel CommonJS is better suited to server-side development which is one reason there's currently a level of **disagreement** over which format should and will be used as the de facto standard in the pre-Harmony age moving forward. Some of the arguments against CJS include a note that many CommonJS APIs address server-oriented features which one would simply not be able to implement at a browser-level in JavaScript - for example, io, system and js could be considered unimplementable by the nature of their functionality.

许多开发者认为CommonJS更适合服务器端开发，其中一个原因是*** 在前Harmony时代的一个事实上的标准。反对CJS的观点包括CommonJS的API符合服务器端的特性但是在浏览器端基本不能实现。例如，io，系统以及js可以认为实际上无法实现它们的功能。

That said, it's useful to know how to structure CJS modules regardless so that we can better appreciate how they fit in when defining modules which may be used everywhere. Modules which have applications on both the client and server include validation, conversion and templating engines. The way some developers are approaching choosing which format to use is opting for CJS when a module can be used in a server-side environment and using AMD if this is not the case.

有种说法，了解CJS模块的结构是非常有用的，当定义可以更好地鉴别他们

As AMD modules are capable of using plugins and can define more granular things like constructors and functions this makes sense. CJS modules are only able to define objects which can be tedious to work with if you're trying to obtain constructors out of them.

Although it's beyond the scope of this article, you may have also noticed that there were different types of 'require' methods mentioned when discussing AMD and CJS.

The concern with a similar naming convention is of course confusion and the community are currently split on the merits of a global require function. John Hann's suggestion here is that rather than calling it 'require', which would probably fail to achieve the goal of informing users about the different between a global and inner require, it may make more sense to rename the global loader method something else (e.g. the name of the library). It's for this reason that a loader like curl.js uses curl() as opposed to require.

#### Related Reading ###

- Demystifying CommonJS Modules
- JavaScript Growing Up
- The RequireJS Notes On CommonJS
- Taking Baby Steps With Node.js And CommonJS Creating Custom Modules
- Asynchronous CommonJS Modules for the Browser
- The CommonJS Mailing List

## AMD && CommonJS ##
## AMD 和 CommonJS
> Competing, But Equally Valid Standards
> 竞争，但都是有根据的标准

虽然这篇文章的重点是使用AMD和CJS，事实上这两种格式都是凑效的、有用的。

AMD adopts a browser-first approach to development, opting for asynchronous behaviour and simplified backwards compatability but it doesn't have any concept of File I/O. It supports objects, functions, constructors, strings, JSON and many other types of modules, running natively in the browser. It's incredibly flexible.

CommonJS on the other hand takes a server-first approach, assuming synchronous behaviour, no global baggage as John Hann would refer to it as and it attempts to cater for the future (on the server). What we mean by this is that because CJS supports unwrapped modules, it can feel a little more close to the ES.next/Harmony specifications, freeing you of the define() wrapper that AMD enforces. CJS modules however only support objects as modules.

Although the idea of yet another module format may be daunting, you may be interested in some samples of work on hybrid AMD/CJS and Univeral AMD/CJS modules.

### Basic AMD Hybrid Format (John Hann) ###
### 基本的AMD混合格式 ###

`define( function (require, exports, module){
    
    var shuffler = require('lib/shuffle');
 
    exports.randomize = function( input ){
        return shuffler.shuffle(input);
    }
});`

### AMD/CommonJS Universal Module Definition (Variation 2, UMDjs) ###
### AMD/CommonJS 统一模块定义（变种2， UMDjs） ###

`/**
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
}));`

### Extensible UMD Plugins With (Variation by myself and Thomas Davis). ###
### 可扩展的UMD插件 ###

#### core.js ####

`// Module/Plugin core
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
 
});`

#### myExtension.js ####

`;(function ( name, definition ) {
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
 
});`

#### app.js ####

`$(function(){
 
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
});`

## ES Harmony ##
> Modules Of The Future

> 未来的模块

TC39, the standards body charged with defining the syntax and semantics of ECMAScript and its future iterations is composed of a number of very intelligent developers. Some of these developers (such as Alex Russell) have been keeping a close eye on the evolution of JavaScript usage for large-scale development over the past few years and are acutely aware of the need for better language features for writing more modular JS.

TC39，ECMAScript和其未来的迭代定义的语法和语义的收费标准机构组成的一个非常聪明的开发商的数量。这其中的许多开发者（例如Alex Russel）一直持续关注着Javascript使用在过去这些年大规模开发中的变革，也敏锐地意识到为了编写更加模块化的JS的更好的语言特性的需求。

For this reason, there are currently proposals for a number of exciting additions to the language including flexible modules that can work on both the client and server, a module loader and more. In this section, I'll be showing you some code samples of the syntax for modules in ES.next so you can get a taste of what's to come.

基于此，目前已经有许多令人兴奋的提案加入到这门语言中，包括灵活的可在客户端以及服务器端工作的模块，模块加载器以及其他。在这一节，我将展示一些在ES.next模块语法的代码示例，你可以先体会到他们会是什么。

> **Note:** Although Harmony is still in the proposal phases, you can already try out (partial) features of ES.next that address native support for writing modular JavaScript thanks to Google's Traceur compiler. To get up and running with Traceur in under a minute, read this getting started guide. There's also a JSConf presentation about it that's worth looking at if you're interested in learning more about the project.

### Modules With Imports And Exports ###

If you've read through the sections on AMD and CJS modules you may be familiar with the concept of module dependencies (imports) and module exports (or, the public API/variables we allow other modules to consume). In ES.next, these concepts have been proposed in a slightly more succinct manner with dependencies being specified using an import keyword. export isn't greatly different to what we might expect and I think many developers will look at the code below and instantly 'get' it.

* **import** declarations bind a module's exports as local variables and may be renamed to avoid name collisions/conflicts.

* **export** declarations declare that a local-binding of a module is externally visible such that other modules may read the exports but can't modify them. Interestingly, modules may export child modules however can't export modules that have been defined elsewhere. You may also rename exports so their external name differs from their local names.
 
`module staff{
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
}`

### Modules Loaded From Remote Sources ###
### 从远程源加载的模块 ###

The module proposals also cater for modules which are remotely based (e.g. a third-party API wrapper) making it simplistic to load modules in from external locations. Here's an example of us pulling in the module we defined above and utilizing it:

`module cakeFactory from 'http://addyosmani.com/factory/cakes.js';
cakeFactory.oven.makeCupcake('sprinkles');
cakeFactory.oven.makeMuffin('large');`

### Module Loader API ###

The module loader proposed describes a dynamic API for loading modules in highly controlled contexts. Signatures supported on the loader include load( url, moduleInstance, error) for loading modules, createModule( object, globalModuleReferences) and others. Here's another example of us dynamically loading in the module we initially defined. Note that unlike the last example where we pulled in a module from a remote source, the module loader API is better suited to dynamic contexts.

`Loader.load('http://addyosmani.com/factory/cakes.js',
    function(cakeFactory){
        cakeFactory.oven.makeCupcake('chocolate');
    });`

### CommonJS-like Modules For The Server ###

For developers who are server-oriented, the module system proposed for ES.next isn't just constrained to looking at modules in the browser. Below for examples, you can see a CJS-like module proposed for use on the server:

`// io/File.js
export function open(path) { ... };
export function close(hnd) { ... };`

`// compiler/LexicalHandler.js
module file from 'io/File';
 
import { open, close } from file;
export function scan(in) {
    try {
        var h = open(in) ...
    }
    finally { close(h) }
}`

`module lexer from 'compiler/LexicalHandler';
module stdlib from '@std';
 
//... scan(cmdline[0]) ...`

### Classes With Constructors, Getters & Setters ###

The notion of a class has always been a contentious issue with purists and we've so far got along with either falling back on JavaScript's prototypal nature or through using frameworks or abstractions that offer the ability to use class definitions in a form that desugars to the same prototypal behavior.

In Harmony, classes come as part of the language along with constructors and (finally) some sense of true privacy. In the following examples, I've included some inline comments to help you understand how classes are structured, but you may also notice the lack of the word 'function' in here. This isn't a typo error: TC39 have been making a conscious effort to decrease our abuse of the function keyword for everything and the hope is that this will help simplify how we write code.

`class Cake{
 
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
 
 
}`

### ES Harmony Conclusions ###

As you can see, ES.next is coming with some exciting new additions. Although Traceur can be used to an extent to try our such features in the present, remember that it may not be the best idea to plan out your system to use Harmony (just yet). There are risks here such as specifications changing and a potential failure at the cross-browser level (IE9 for example will take a while to die) so your best bets until we have both spec finalization and coverage are AMD (for in-browser modules) and CJS (for those on the server).

#### Related Reading ####

- A First Look At The Upcoming JavaScript Modules
- David Herman On JavaScript/ES.Next (Video)
- ES Harmony Module Proposals
- ES Harmony Module Semantics/Structure Rationale
- ES Harmony Class Proposals

##Conclusions And Further Reading ##
> A Review

In this article we've reviewed several of the options available for writing modular JavaScript using modern module formats. These formats have a number of advantages over using the (classical) module pattern alone including: avoiding a need for developers to create global variables for each module they create, better support for static and dynamic dependency management, improved compatibility with script loaders, better (optional) compatibility for modules on the server and more.

在这篇文章中我们回顾了许多目前可用的使用当前模块规范的编写模块化的Javascript的选择。这些规范拥有一些特性包括传统的模块模式，其中包括：避免开发者创建为一个全局变量

In short, I recommend trying out what's been suggested today as these formats offer a lot of power and flexibility that can help when building applications based on many reusable blocks of functionality.

And that's it for now. If you have further questions about any of the topics covered today, feel free to hit me up on twitter and I'll do my best to help!

The technical review for this article was doing using Diigo (for Google Chrome). Diigo is a free tool that allows you to add both comments and highlights to any live document on the web and if there are corrections or extensions you would like to suggest, please use either Diigo (or a GitHub gist) and I'll do my test to address any points you send over.
