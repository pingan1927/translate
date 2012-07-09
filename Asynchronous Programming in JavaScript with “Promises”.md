# 在JavaScript中使用“Promise”进行异步编程 #

>Published Monday, September 12, 2011 4:04 AM

为推动web 编程向前发展异步模式变得越来越普通和重要。使用JavaScript工作的挑战越发的大。为了使异步的模式变得简单，JavaScript库（像jQuery以及Dojo）已经都添加了一种叫做promise的抽象（有时也叫做deferred）。通过这些库，开发人员可以在每个浏览器中使用promise并得到很好的ECMAScript 5支持。在这篇文章中，我们一起探索下如何在你的web应用程序中通过将[XMLHttpRequest2](http://www.w3.org/TR/XMLHttpRequest2/) (XHR2)作为一个特定的例子来使用promise。

## 使用异步编程的好处以及挑战 ##

作为一个例子，请考虑下某个网页，以例如[XMLHttpRequest2](http://www.w3.org/TR/XMLHttpRequest2/) (XHR2) 或者 [Web](http://blogs.msdn.com/b/ie/archive/2011/07/01/web-workers-in-ie10-background-javascript-makes-web-apps-faster.aspx) [Workers](http://blogs.msdn.com/b/ie/archive/2011/07/12/debugging-web-workers-in-ie10.aspx)的异步操作运行。有许多好处，例如许多任务可以“并行”地发生。但对于当页面正在处理异步任务时开发人员保持页面对于用户的响应以及不阻塞人机交互是个复杂的事情。之所以复杂是因为程序的执行不再真正地遵循一条简单的线形路径。

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

嵌套回调让代码难以理解-哪些代码是应用具体的业务逻辑而哪些又是为处理异步请求而需要的模版代码？另外，由于嵌套回调，需要处理的错误变得支离破碎。我们需要确认许多地方来验证某个错误是否发生。

为减少协调异步行为的复杂性，开发人员找到了一种替代嵌套回调，协调一致的，对错误处理容易理解的方法。

## Promises ##

其中的一种模式就是promise，它代表了一种潜在地长时间运行而且不一定必须地完成操作的结果。与阻塞并长时间等待运行计算的完成不同，这种模式返回一个代表承诺（promised）结果的对象。

例如，需要创建一个请求到第三方系统，而它的网络延迟是不确定的。应用程序可以释放出来做其他事情直到返回值变得必要，而不是在等待时阻塞住整个程序。Pormise实现了一种方法，为状态改变通知注册回调函数，通常命名为then方法。

```
var results = searchTwitter(term).then(filterResults);
displayResults(results);
```

在任何时刻，promise只可能处于三种状态之一：unfulfilled（未完成）, resolved（已解决） or rejected（拒绝）。

为了说明这些概念如何工作，我们可以了解下[CommonJS Promise/A](http://wiki.commonjs.org/wiki/Promises/A)标准,这个标准已经在流行的库中有了许多的衍生工具。在promise对象中的then方法为resolved以及rejected状态添加了处理程序。then函数会返回另外一个promise对象以便形成promise管道，使开发人员能够把异步操作串联起来，这样第一个操作的结果就可以传入到第二个中了。

```
then(resolvedHandler, rejectedHandler);
```

函数resolvedHandler回调函数会在promise对象进入完成状态时被调用，并传递从计算返回的结果。而rejectedHandler函数会在进入拒绝状态时被调用。

可以使用promise的伪代码来重新实现上面的示例，主要包含创建一个Ajax请求用于搜索Twitter、用数据填充屏幕以及处理错误。为了更好的理解实现方法，我们尝试着从零开始构建一个promise模式的框架。首先我们以一个例子开始，即如果我们从头开始设计一个仅包含基础功能的promise库应该有什么，首先，我们需要一些对象格式来保存promise。
```
var Promise = function () {
    /* initialize promise */
};
```

接下来，我们需要实现then方法，允许我们根据promise的状态变化将操作串联在一起。这个方法包含两个函数参数，用于处理当promise被解决以及当promise被拒绝的情况。

```
Promise.prototype.then = function (onResolved, onRejected) {
    /* invoke handlers based upon state transition */
};
```

我们也需要一对方法来处理未完成和已解决或者已拒绝之间的状态转换。

```
Promise.prototype.resolve = function (value) {
    /* move from unfulfilled to resolved */
};

Promise.prototype.reject = function (error) {
    /* move from unfulfilled to rejected */
};
```

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

到目前为止，我们可以把promise模式应用于单个Ajax请求，接下来我们讨论一种场景——当我们想要发送超过一个Ajax请求并协调它们的结果。为了处理这种场景，我们会在我们的Promise对象中创造一个when方法用来储存会被调用的promise对象。一旦promise从unfulfilled转变成resolved或者rejected，then方法里对应的处理函数就会被调用。when方法在需要等待所有操作都完成才能继续的场景至关重要。

```
Promise.when = function () {
    /* handle promises arguments and queue each */
};
```

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

需要记住的重点是在这些例子的代码除了普通JavaScript代码并没有其它不同。Web开发人员必定会创造他们自己的类Promise库；但为了方便和一致性你可以利用在一般JavaScript库中暴露的promise模式。

## 探索jQuery和Dojo Toolkit中的Promises ##

有许多针对开发人员的JavaScript库已经以某种形式实现了Promise。现在我们就来过下暴露了promise或者相同概念的几个库。

### The Dojo Toolkit ###

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

Dojo Toolkit可能是第一个实现这种模式的库，但还有许多其他主流的库例如jQuery也提供了相似的模式。

### jQuery ###

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

与dojo不同的是，jQuery不会从then方法返回另外一个promise。取而代之的是，jQuery提供了一个pipe方法用于将操作串联在一起。另外，jQuery提供了其他的工具方法允许更加富的组件化，包括像jQuery的$语法风格从pipe方法中过滤。

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

## 结论 ##

对于开发人员有许多合适的选择来处理复杂的异步编程。使用众所周知的模式例如promise以及deferred对象，以及公开了它们的库，开发人员可以创建具有无缝连接异步请求的复杂交互。在这个例子中，我们首先开始讨论了通过XMLHttpRequest利用promise以及deferred对象，但是这种模式很容易被[Web Workers](http://www.w3.org/TR/workers/)、[setImmediate](https://dvcs.w3.org/hg/webperf/raw-file/tip/specs/setImmediate/Overview.html) API、[FileAPI](http://www.w3.org/TR/FileAPI/)或者其他的异步API给取代。你可以使用普通的JavaScript库因此也就无需编写模版代码。

[promises](http://msdn.microsoft.com/en-us/scriptjunkie/gg723713.aspx)模式是一个好的开始，但不会是一个最终的解决方案。实际上，[许多模式](https://github.com/joyent/node/wiki/modules#wiki-async-flow)都在兴起用于适应异步编程。我们觉得这是一个有趣的开发过程可以使我们作为一个web开发人员的生活更加容易并且也能够帮助你写出你自己的web应用程序。Happy coding!

—Matt Podwysocki, JavaScript geek and consultant

—Amanda Silver, Program Manager for JavaScript