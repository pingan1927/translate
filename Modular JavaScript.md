# Writing Modular JavaScript With AMD, CommonJS & ES Harmony #
## Modularity  ##
> The Importance Of Decoupling Your Application

When we say an application is modular, we generally mean it's composed of a set of highly decoupled, distinct pieces of functionality stored in modules. As you probably know, [loose coupling](http://arguments.callee.info/2009/05/18/javascript-design-patterns--mediator/) facilitates easier maintainability of apps by removing dependencies where possible. When this is implemented efficiently, its quite easy to see how changes to one part of a system may affect another.

我们说一（某）个应用是模块化的，我们通常是指它是由一组存储于模块中高度解耦的、具有不同功能的片段组成。正如你所了解的，[松耦合](http://arguments.callee.info/2009/05/18/javascript-design-patterns--mediator/)通过消除可能的依赖从而更容易有利于应用程序的可维护性。当模块化很有效地实现，可以很容易地观察到系统的这一部分是如何影响到另一部分。

Unlike some more traditional programming languages however, the current iteration of JavaScript ([ECMA-262](http://www.ecma-international.org/publications/standards/Ecma-262.htm)) doesn't provide developers with the means to import such modules of code in a clean, organized manner. It's one of the concerns with specifications that haven't required great thought until more recent years where the need for more organized JavaScript applications became apparent.

然而，与传统的编程语言不用的是，当前的javascript版本（[ECMA-262]）并没有提供给开发者一个清晰地，格式良好的导入模块的手段。

Instead, developers at present are left to fall back on variations of the [module](http://www.adequatelygood.com/2010/3/JavaScript-Module-Pattern-In-Depth) or [object literal](http://blog.rebeccamurphey.com/2009/10/15/using-objects-to-organize-your-code) patterns. With many of these, module scripts are strung together in the DOM with namespaces being described by a single global object where it's still possible to incur naming collisions in your architecture. There's also no clean way to handle dependency management without some manual effort or third party tools.

取而代之的是，目前的开发者仍然只能回到使用模块或者对象字面量的变化上。在这当中，模块脚本被一个单一全局对象命名空间在Dom中被串联起来

Whilst native solutions to these problems will be arriving in [ES Harmony](http://wiki.ecmascript.org/doku.php?id=harmony:modules), the good news is that writing modular JavaScript has never been easier and you can start doing it today.

In this article, we're going to look at three formats for writing modular JavaScript: **AMD**, **CommonJS** and proposals for the next version of JavaScript, **Harmony**.

## Prelude ##
> A Note On Script Loaders

It's difficult to discuss AMD and CommonJS modules without talking about the elephant in the room - [script loaders](http://msdn.microsoft.com/en-us/scriptjunkie/hh227261). At present, script loading is a means to a goal, that goal being modular JavaScript that can be used in applications today - for this, use of a compatible script loader is unfortunately necessary. In order to get the most out of this article, I recommend gaining a basic understanding of how popular script loading tools work so the explanations of module formats make sense in context.

There are a number of great loaders for handling module loading in the AMD and CJS formats, but my personal preferences are [RequireJS](http://requirejs.org/) and [curl.js](https://github.com/unscriptable/curl). Complete tutorials on these tools are outside the scope of this article, but I can recommend reading John Hann's post about [curl.js](http://unscriptable.com/index.php/2011/03/30/curl-js-yet-another-amd-loader/) and James Burke's [RequireJS](http://requirejs.org/docs/api.html) API documentation for more.

From a production perspective, the use of optimization tools (like the RequireJS optimizer) to concatenate scripts is recommended for deployment when working with such modules. Interestingly, with the [Almond](https://github.com/jrburke/almond) AMD shim, RequireJS doesn't need to be rolled in the deployed site and what you might consider a script loader can be easily shifted outside of development.

That said, James Burke would probably say that being able to dynamically load scripts after page load still has its use cases and RequireJS can assist with this too. With these notes in mind, let's get started.

## AMD ##
> A Format For Writing Modular JavaScript In The Browser
> 浏览器端编写模块化JavaScript的规范

The overall goal for the AMD (Asynchronous Module Definition) format is to provide a solution for modular JavaScript that developers can use today. It was born out of Dojo's real world experience using XHR+eval and proponents of this format wanted to avoid any future solutions suffering from the weaknesses of those in the past.

AMD（异步模块定义）规范的总体目标是为当今开发者提供一个可以使用的模块化JavaScript解决方案。*它诞生于Dojo的现实世界中使用的XHR+ eval和该格式的支持者希望患有那些在过去的弱点，以避免任何未来的解决方案的经验。*

The AMD module format itself is a proposal for defining modules where both the module and dependencies can be [asynchronously](http://dictionary.reference.com/browse/asynchronous) loaded. It has a number of distinct advantages including being both asynchronous and highly flexible by nature which removes the tight coupling one might commonly find between code and module identity. Many developers enjoy using it and one could consider it a reliable stepping stone towards the [module system](http://wiki.ecmascript.org/doku.php?id=harmony:modules) proposed for ES Harmony.

AMD模块规范本身是一个提案，用于定义模块，使模块以及以来都可以被异步加载。

AMD began as a draft specification for a module format on the CommonJS list but as it wasn't able to reach full concensus, further development of the format moved to the [amdjs](https://github.com/amdjs) group.

AMD最开始作为一个CommonJS目录中的模块格式的规范草案，但是由于它无法完全达成共识，格式的进一步发展就是提出一个amdjs组。

Today it's embraced by projects including Dojo (1.7), MooTools (2.0), Firebug (1.8) and even jQuery (1.7). Although the term CommonJS AMD format has been seen in the wild on occasion, it's best to refer to it as just AMD or Async Module support as not all participants on the CJS list wished to pursue it.

今天它被囊括在包括Dojo（1.7），MooTools(2.0),Firebug(1.8)甚至JQuery（1.7）等多个项目中。*CommonJS的术语AMD的格式虽然已在野外场合上看到，这是最好的指空肠名单上的所有参与者不希望追求它只是AMD或异步模块支持。*

> **Note:** There was a time when the proposal was referred to as Modules Transport/C, however as the spec wasn't geared for transporting existing CJS modules, but rather, for defining modules it made more sense to opt for the AMD naming convention.

### Getting Started With Modules ###
### 模块入门 ###

The two key concepts you need to be aware of here are the idea of a `define` method for facilitating module definition and a `require` method for handling dependency loading. *define* is used to define named or unnamed modules based on the proposal using the following signature:

在这里你需要关注的两个关键概念是用于促进模块定义的define方法以及用于处理依赖加载的require方法。提案这提案，define使用以下格式来定义命名的或者未命名的模块：

`define(
    module_id /*optional*/, 

    [dependencies] /*optional*/, 

    definition function /*function for instantiating the module or object*/
);`