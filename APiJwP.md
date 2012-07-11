# Asynchronous Programming in JavaScript with “Promises” #
＃ 在JavaScript中使用“Promise”进行异步编程 ＃

>Published Monday, September 12, 2011 4:04 AM

Asynchronous patterns are becoming more common and more important to moving web programming forward. They can be challenging to work with in JavaScript. To make asynchronous (or async) patterns easier, JavaScript libraries (like jQuery and Dojo) have added an abstraction called promises (or sometimes deferreds). With these libraries, developers can use promises in any browser with good ECMAScript 5 support. In this post, we’ll explore how to use promises in your web applications using [XMLHttpRequest2](http://www.w3.org/TR/XMLHttpRequest2/) (XHR2) as a specific example.

为推动web 编程向前发展异步模式变得越来越普通和重要。使用JavaScript工作的挑战越发的大。为了使异步的模式变得简单，JavaScript库（像jQuery以及Dojo）已经都添加了一种叫做promise的抽象（有时也叫做deferred）。通过这些库，开发人员可以在每个浏览器中使用promise并得到很好的ECMAScript 5支持。在这篇文章中，我们一起探索下如何在你的web应用程序中通过将[XMLHttpRequest2](http://www.w3.org/TR/XMLHttpRequest2/) (XHR2)作为一个特定的例子来使用promise。

## Benefits and Challenges with Asynchronous Programming ##
## 使用异步编程的好处以及挑战 ##

As an example, consider a web page that starts an asynchronous operation like [XMLHttpRequest2](http://www.w3.org/TR/XMLHttpRequest2/) (XHR2) or [Web](http://blogs.msdn.com/b/ie/archive/2011/07/01/web-workers-in-ie10-background-javascript-makes-web-apps-faster.aspx) [Workers](http://blogs.msdn.com/b/ie/archive/2011/07/12/debugging-web-workers-in-ie10.aspx). There’s a benefit as some work happens “in parallel.” There’s complexity for the developer to keep the page responsive to people and not block human interaction while coordinating what the web page is doing with the asynchronous work. There’s complexity because program execution no longer really follows a simple linear path.

作为一个例子，请考虑下某个网页，以例如[XMLHttpRequest2](http://www.w3.org/TR/XMLHttpRequest2/) (XHR2) 或者 [Web](http://blogs.msdn.com/b/ie/archive/2011/07/01/web-workers-in-ie10-background-javascript-makes-web-apps-faster.aspx) [Workers](http://blogs.msdn.com/b/ie/archive/2011/07/12/debugging-web-workers-in-ie10.aspx)的异步操作运行。有许多好处，例如许多任务可以“并行”地发生。但对于当页面正在处理异步任务时开发人员保持页面对于用户的响应以及不阻塞人机交互是个复杂的事情。之所以复杂是因为程序的执行不再真正地遵循一条简单的线形路径。

When you make an asynchronous call, you need to handle both successful completion of the work as well as any potential errors that may arise during execution. Upon the successful completion of one asynchronous call, you may want to pass the result into make another Ajax request. This can introduce complexity through *nested callbacks*.

当你作出一个异步的响应，你必须处理任务的成功返回并同时处理在执行时可能产生的任何潜在的错误。根据一个异步请求的成功完成，你可能将返回的结果作为另外一个Ajax的请求中。这可能通过*嵌套回调*产生复杂的事情。

```
function searchTwitter(term, onload, onerror) {
    var xhr, results, url;

    url = 'http://search.twitter.com/search.json?rpp=100&q=' + term;

    xhr = new XMLHttpRequest();
    xhr.open('GET', url, true);

    xhr.onload = function (e) {
        if (this.status === 200) {
            results = JSON.parse(this.responseText);
        onload(results);
        }
    };

    xhr.onerror = function (e) {
        onerror(e);
    };

    xhr.send();
}

function handleError(error) {
    /* handle the error */
}

function concatResults() {
    /* order tweets by date */
}

function loadTweets() {
    var container = document.getElementById('container');

    searchTwitter('#IE10', function (data1) {
        searchTwitter('#IE9', function (data2) {
            /* Reshuffle due to date */
            var totalResults = concatResults(data1.results, data2.results);

            totalResults.forEach(function (tweet) {
                var el = document.createElement('li');
                el.innerText = tweet.text;
                container.appendChild(el);
            });
        }, handleError);
    }, handleError);
}
```

The nested callbacks make the code hard to understand – what code is business logic specific to the app and what is the boilerplate code required to deal with the asynchronous call? In addition, due to the nested callbacks, the error handling becomes fragmented. We must check several places to see if an error occurred.

嵌套回调让代码难以理解-哪些代码是应用具体的业务逻辑而哪些又是为处理异步请求而需要的模版代码？另外，由于嵌套回调，需要处理的错误变得支离破碎。我们需要确认许多地方来验证某个错误是否发生。

To reduce the complexity of coordinating asynchronous behavior, developers have looked for a way to perform consistent, easy to understand error handling with an alternative to nested callbacks.

为减少协调异步行为的复杂性，开发人员找到了一种替代嵌套回调，协调一致的，对错误处理容易理解的方法。

## Promises ##

One pattern is a promise, which represents the result of a potentially long running and not necessarily complete operation. Instead of blocking and waiting for the long-running computation to complete, the pattern returns an object which represents the promised result.

其中的一种模式就是promise，它代表了一种潜在地长时间运行而且不一定必须地完成操作的结果。与阻塞并长时间等待运行计算的完成不同，这种模式返回一个代表承诺（promised）结果的对象。

An example of this might be making a request to a third-party system where network latency is uncertain. Instead of blocking the entire application while waiting, the application is free to do other things until the value is needed. A promise implements a method for registering callbacks for state change notifications, commonly named the then method:

例如，需要创建一个请求到第三方系统，而它的网络延迟是不确定的。应用程序可以释放出来做其他事情直到返回值变得必要，而不是在等待时阻塞住整个程序。Pormise实现了一种方法，为状态改变通知注册回调函数，通常命名为then方法。

```
var results = searchTwitter(term).then(filterResults);
displayResults(results);
```

At any moment in time, promises can be in one of three states: unfulfilled, resolved or rejected.

在任何时刻，promise只可能处于三种状态之一：unfulfilled（未完成）, resolved（已解决） or rejected（拒绝）。

To give an idea how the concept works, let’s start out with the [CommonJS Promise/A](http://wiki.commonjs.org/wiki/Promises/A) proposal which has several derivatives in popular libraries. The then method on the promise object adds handlers for the resolved and rejected states. This function returns another promise object to allow for promise-pipelining, enabling the developer to chain together async operations where the result of the first operation will get passed to the second.

为了说明这些概念如何工作，我们可以了解下[CommonJS Promise/A](http://wiki.commonjs.org/wiki/Promises/A)标准,这个标准已经在流行的库中有了许多的衍生工具。在promise对象中的then方法为resolved以及rejected状态添加了处理程序。then函数会返回另外一个promise对象以便形成promise管道，使开发人员能够把异步操作串联起来，这样第一个操作的结果就可以传入到第二个中了。

```
then(resolvedHandler, rejectedHandler);
```

The resolvedHandler callback function is invoked when the promise enters the fulfilled state, passing in the result from the computation. The rejectedHandler is invoked when the promise goes into the failed state.

函数resolvedHandler回调函数会在promise对象进入完成状态时被调用，并传递从计算返回的结果。而rejectedHandler函数会在进入拒绝状态时被调用。

We’ll revisit the example above using a pseudo code example of a promise to make the Ajax request to search Twitter, populate the screen with data, and handle errors. Let’s start with an example of what a promise library might look like if we were designing one from scratch with just the very basics. First we’ll need some form of object to keep the promise.

可以使用promise的伪代码来重新实现上面的示例，主要包含创建一个Ajax请求用于搜索Twitter、用数据填充屏幕以及处理错误。为了更好的理解实现方法，我们尝试着从零开始构建一个promise模式的框架。首先我们以一个例子开始，即如果我们从头开始设计一个仅包含基础功能的promise库应该有什么，首先，我们需要一些对象格式来保存promise。
```
var Promise = function () {
    /* initialize promise */
};
```

Next, we’ll need to implement the then method which allows us to chain together operations based upon state change of our promise. This method takes two functions for handling when the promise is resolved and for when the promise is rejected.

接下来，我们需要实现then方法，允许我们根据promise的状态变化将操作串联在一起。这个方法包含两个函数参数，用于处理当promise被解决以及当promise被拒绝的情况。

```
Promise.prototype.then = function (onResolved, onRejected) {
    /* invoke handlers based upon state transition */
};
```

We’ll also need a couple of methods to perform a state transition between unfulfilled and resolved or rejected states.

我们也需要一对方法来处理未完成和已解决或者已拒绝之间的状态转换。

```
Promise.prototype.resolve = function (value) {
    /* move from unfulfilled to resolved */
};

Promise.prototype.reject = function (error) {
    /* move from unfulfilled to rejected */
};
```
Now that we have some boilerplate for what a Promise object could be, let’s walk through the example from above of querying Twitter for #IE10 tagged tweets. First, we’ll create a method for making an Ajax GET request using the XMLHttpRequest2 to a given URL and wrap it in a promise. Next, we’ll create a method specifically for Twitter calling our Ajax wrapper method with a given search term. Finally, we’ll invoke our search function and display the results in an unordered list.

对于一个promise对于应该是什么样现在我们已经搭建的差不多了。我们可以继续上面的示例，获取包含IE10标签的tweets。首先，我们创建一个方法通过使用XMLHttpRequest2来发送一个Ajax Get请求到一个给定的URL并且将它封装成一个promise。接下来，我们将特别为Twitter创建一个方法用来调用含有给定搜索条件的Ajax封装方法。最后，我们会调用我们的搜索函数并在无序列表中展示结果。

```
function searchTwitter(term) {

    var url, xhr, results, promise;
    url = 'http://search.twitter.com/search.json?rpp=100&q=' + term;
    promise = new Promise();
    xhr = new XMLHttpRequest();
    xhr.open('GET', url, true);

    xhr.onload = function (e) {
        if (this.status === 200) {
            results = JSON.parse(this.responseText);
            promise.resolve(results);
        }
    };

    xhr.onerror = function (e) {
        promise.reject(e);
    };

    xhr.send();

    return promise;

}

function loadTweets() {
    var container = document.getElementById('container');

    searchTwitter('#IE10').then(function (data) {
        data.results.forEach(function (tweet) {
            var el = document.createElement('li');
            el.innerText = tweet.text;
            container.appendChild(el);
        });
    }, handleError);
}
```

Now that we’re able to make a single Ajax request as a Promise, let’s discuss a scenario when we want to make more than one Ajax request and coordinate the results. To handle this scenario, we’ll create a when method on our Promise object to queue the promises for to be invoked. Once the promises move from unfulfilled to either resolved or rejected, the appropriate handler is called in the then method. The when method is essentially a fork-join operation that awaits completion of all of the operations before continuing.

到目前为止，我们可以把promise模式应用于单个Ajax请求，接下来我们讨论一种场景——当我们想要发送超过一个Ajax请求并协调它们的结果。为了处理这种场景，我们会在我们的Promise对象中创造一个when方法用来储存会被调用的promise对象。一旦promise从unfulfilled转变成resolved或者rejected，then方法里对应的处理函数就会被调用。when方法在需要等待所有操作都完成才能继续的场景至关重要。

```
Promise.when = function () {
    /* handle promises arguments and queue each */
};
```

Now we can queue up multiple promises simultaneously, for example by searching for both #IE10 and #IE9 on Twitter.

现在我们可以同时存储多个promise，以在Twitter上搜索IE10和IE9两个标签的内容为例。

```
var container, promise1, promise2;

container = document.getElementById('container');

promise1 = searchTwitter('#IE10');
promise2 = searchTwitter('#IE9');

Promise.when(promise1, promise2).then(function (data1, data2) {
    /* Reshuffle due to date */
    var totalResults = concatResults(data1.results, data2.results);

    totalResults.forEach(function (tweet) {
        var el = document.createElement('li');
        el.innerText = tweet.text;
        container.appendChild(el);
    });
}, handleError);
```

The important thing to remember is that the code in these samples is nothing but normal JavaScript. Web developers certainly create their own Promise-like libraries; but for convenience and consistency you can leverage the promises patterns exposed in common JavaScript libraries.

需要记住的重点是在这些例子的代码除了普通JavaScript代码并没有其它不同。Web开发人员必定会创造他们自己的类Promise库；但为了方便和一致性你可以利用在一般JavaScript库中暴露的promise模式。

## Exploring Promises in jQuery and the Dojo Toolkit ##
## 探索jQuery和Dojo Toolkit中的Promises ##

There are many JavaScript libraries that are available to the developer which implement some form of a Promise. Let’s now explore a few libraries which expose promises or similar concepts.

有许多针对开发人员的JavaScript库已经以某种形式实现了Promise。现在我们就来过下暴露了promise或者相同概念的几个库。

### The Dojo Toolkit ###

The first widespread use of this pattern was with the [dojo](http://dojotoolkit.org/reference-guide/dojo/Deferred.html) toolkit deferred object in version 0.9. Just like the CommonJS Promises/A proposal above, this object exposes a then method which allows the developer to handle both the fulfillment and error states and chain the promises together. The dojo.Deferred object exposes two additional methods; resolve which fulfills the promise, and reject, which sends the promise into the rejected state. Below is an example of using the dojo.Deferred object to make an Ajax request to an URL and parse the results.

第一个广泛使用promise模式的是0.9版本[dojo](http://dojotoolkit.org/reference-guide/dojo/Deferred.html) toolkit中的deferred对象。与上面的CommonJS Promises/A标准一样，这个对象暴露了一个then方法，允许开发人员处理包括完成和错误的状态并将对应的promise串联在一起。dojo.Deferred对象暴露了两个额外的方法；resolve符合promise，reject则给promise发送rejected状态。下面是一个使用dojo.Deferred对象创建一个Ajax请求到一个URL并解析返回结果的例子。

```
function searchTwitter(term) {
    var url, xhr, results, def;
    url = 'http://search.twitter.com/search.json?rpp=100&q=' + term;
    def = new dojo.Deferred();
    xhr = new XMLHttpRequest();
    xhr.open('GET', url, true);

    xhr.onload = function (e) {
        if (this.status === 200) {
            results = JSON.parse(this.responseText);
            def.resolve(results);
        }
    };

    xhr.onerror = function (e) {
        def.reject(e);
    };

    xhr.send();

    return def;
}

dojo.ready(function () {
    var container = dojo.byId('container');

    searchTwitter('#IE10').then(function (data) {
        data.results.forEach(function (tweet) {
            dojo.create('li', {
                innerHTML: tweet.text
            }, container);
        });
    });
});
```

Fortunately, some of the Ajax methods such as dojo.xhrGet already return a dojo.Deferred object, so you don’t need to worry about wrapping those methods yourself.

非常庆幸地是，许多Ajax方法例如dojo.xhrGet已经返回一个dojo.Deferred对象，因此你不必担心需要自己封装这些方法。

```
var deferred = dojo.xhrGet({
    url: "search.json",
    handleAs: "json"
});

deferred.then(function (data) {
    /* handle results */
}, function (error) {
    /* handle error */
});
```

The next concept that Dojo introduces is the dojo.DeferredList, which allows the developer to handle multiple dojo.Deferred objects at once and then return the results to the callback handler passed to the then function. This mirrors the when method we had on our own version of the promise object.

Dojo带来的第二个概念是dojo.DeferredList,它允许开发人员同时处理多个dojo.Deferred对象并且通过传入then函数返回结果给回调处理程序。这其实就是上面所提到的when方法的另一种表现形式。

```
dojo.require("dojo.DeferredList");

dojo.ready(function () {
    var container, def1, def2, defs;
    container = dojo.byId('container');

    def1 = searchTwitter('#IE10');
    def2 = searchTwitter('#IE9');
    defs = new dojo.DeferredList([def1, def2]);

    defs.then(function (data) {
        // Handle exceptions
        if (!results[0][0] || !results[1][0]) {
            dojo.create("li", {
                innerHTML: 'an error occurred'
            }, container);
            return;
        }

        var totalResults = concatResults(data[0][1].results, data[1][1].results);

        totalResults.forEach(function (tweet) {
            dojo.create("li", {
                innerHTML: tweet.text
            }, container);
        });
    });
});
```

The Dojo Toolkit may have been the first one to implement this pattern, but there are other major libraries such as jQuery which have also exposed a similar pattern.

Dojo Toolkit可能是第一个实现这种模式的库，但还有许多其他主流的库例如jQuery也提供了相似的模式。

### jQuery ###

jQuery introduced a new concept in version 1.5 called Deferred which is also a derivative implementation of the CommonJS Promises/A proposal. The Deferred object exposes a then method which allows the developer to handle both the fulfillment and error states. Like dojo, this object also exposes resolve and reject. The developer can create a Deferred object in jQuery by calling the $.Deferred function.

jQuery在1.5版本介绍了一个新概念叫做*Deferred*，也是CommonJS Promises/A标准的一种衍生实现。Deferred对象公开了then方法，允许开发人员处理包括完成以及失败的状态。类似于dojo，这个对象也公开了resolve以及reject。开发人员能够在jQuery中创建叫做$.Deferref函数的Deferred对象。

```
function xhrGet(url) {
    var xhr, results, def;
    def = $.Deferred();
    xhr = new XMLHttpRequest();
    xhr.open('GET', url, true);

    xhr.onload = function (e) {
        if (this.status === 200) {
            results = JSON.parse(this.responseText);
            def.resolve(results);
        }
    };

    xhr.onerror = function (e) {
        def.reject(e);
    };

    xhr.send();

    return def;
}

function searchTwitter(term) {
    var url, xhr, results, def;
    url = 'http://search.twitter.com/search.json?rpp=100&q=' + term;
    def = $.Deferred();
    xhr = new XMLHttpRequest();
    xhr.open('GET', url, true);

    xhr.onload = function (e) {
        if (this.status === 200) {
            results = JSON.parse(this.responseText);
            def.resolve(results);
        }
    };

    xhr.onerror = function (e) {
        def.reject(e);
    };

    xhr.send();

    return def;
}

$(document).ready(function () {
    var container = $('#container');

    searchTwitter('#IE10').then(function (data) {
        data.results.forEach(function (tweet) {
            container.append('<li>' + tweet.text + '</li>');
        });
    });
});
```

Different from dojo, jQuery does not return another promise from the then method. Instead, jQuery provides the pipe method to chain operations together. Additionally, jQuery offers other utility methods which allow for richer composition including filtering through the pipe method as well as the jQuery style $ syntax.

与dojo不同的是，jQuery不会从then方法返回另外一个promise。取而代之的是，jQuery提供了一个pipe方法用于将操作串联在一起。另外，jQuery提供了其他的工具方法允许更加富的组件化，包括像jQuery的$语法风格从pipe方法中过滤。

The jQuery 1.5 release alters the Ajax methods to now return the jqXHR object which directly implements the promise interface.

jQuery 1.5发行版修改了Ajax方法，现在变成返回jqXHR对象，直接实现了promise接口。

```
$.ajax({
    url: 'http://search.twitter.com/search.json',
    dataType: 'jsonp',
    data: { q: '#IE10', rpp: 100 }
}).then(function (data) {
    /* handle data */
}, function (error) {
    /* handle error */
});
```

For consistency, the jQuery.ajax method also provides the success, error, and complete methods.
为了保持一致性，jQuery.ajax方法也提供了success、error以及complete方法。

```
$.ajax({
    url: 'http://search.twitter.com/search.json',
    dataType: 'jsonp',
    data: { q: '#IE10', rpp: 100 }
}).success(function (data) {
    /* handle data */
}).error(function (error) {
    /* handle error */
});
```

## Conclusion ##
## 结论 ##

There are many options available to the developer on how to deal with the complexities of asynchronous programming. With well-known patterns such as promises and deferred objects, and libraries that expose them, the developer is able to create rich interactions which seamlessly bridge asynchronous requests. In this example, we discussed leveraging promises and deferred objects atop XMLHttpRequests  but the patterns could easily be layered on top of [Web Workers](http://www.w3.org/TR/workers/), the [setImmediate](https://dvcs.w3.org/hg/webperf/raw-file/tip/specs/setImmediate/Overview.html) API, the [FileAPI](http://www.w3.org/TR/FileAPI/), or any other asynchronous API. You can use common JavaScript libraries so you don’t have to write boilerplate code.

对于开发人员有许多合适的选择来处理复杂的异步编程。使用众所周知的模式例如promise以及deferred对象，以及公开了它们的库，开发人员可以创建具有无缝连接异步请求的复杂交互。在这个例子中，我们首先开始讨论了通过XMLHttpRequest利用promise以及deferred对象，但是这种模式很容易被[Web Workers](http://www.w3.org/TR/workers/)、[setImmediate](https://dvcs.w3.org/hg/webperf/raw-file/tip/specs/setImmediate/Overview.html) API、[FileAPI](http://www.w3.org/TR/FileAPI/)或者其他的异步API给取代。你可以使用普通的JavaScript库因此也就无需编写模版代码。

The [promises](http://msdn.microsoft.com/en-us/scriptjunkie/gg723713.aspx) pattern is a good start, but it’s not the end of the solution. In fact, [many patterns](https://github.com/joyent/node/wiki/modules#wiki-async-flow) are emerging to address asynchronous programming. We think this is an interesting development which makes our lives easier as web developers and think it can help you write your web applications as well. Happy coding!

[promises](http://msdn.microsoft.com/en-us/scriptjunkie/gg723713.aspx)模式是一个好的开始，但不会是一个最终的解决方案。实际上，[许多模式](https://github.com/joyent/node/wiki/modules#wiki-async-flow)都在兴起用于适应异步编程。我们觉得这是一个有趣的开发过程可以使我们作为一个web开发人员的生活更加容易并且也能够帮助你写出你自己的web应用程序。Happy coding!

—Matt Podwysocki, JavaScript geek and consultant

—Amanda Silver, Program Manager for JavaScript