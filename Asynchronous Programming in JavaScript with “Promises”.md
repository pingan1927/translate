# Asynchronous Programming in JavaScript with “Promises” #
＃ 在JavaScript中使用“Promise”进行异步编程 ＃
>Published Monday, September 12, 2011 4:04 AM

Asynchronous patterns are becoming more common and more important to moving web programming forward. They can be challenging to work with in JavaScript. To make asynchronous (or async) patterns easier, JavaScript libraries (like jQuery and Dojo) have added an abstraction called promises (or sometimes deferreds). With these libraries, developers can use promises in any browser with good ECMAScript 5 support. In this post, we’ll explore how to use promises in your web applications using XMLHttpRequest2 (XHR2) as a specific example.

在推动web 编程向前发展方面异步模式变得越来越普通和重要。ta

## Benefits and Challenges with Asynchronous Programming ##

As an example, consider a web page that starts an asynchronous operation like XMLHttpRequest2 (XHR2) or Web Workers. There’s a benefit as some work happens “in parallel.” There’s complexity for the developer to keep the page responsive to people and not block human interaction while coordinating what the web page is doing with the asynchronous work. There’s complexity because program execution no longer really follows a simple linear path.

When you make an asynchronous call, you need to handle both successful completion of the work as well as any potential errors that may arise during execution. Upon the successful completion of one asynchronous call, you may want to pass the result into make another Ajax request. This can introduce complexity through nested callbacks.

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

To reduce the complexity of coordinating asynchronous behavior, developers have looked for a way to perform consistent, easy to understand error handling with an alternative to nested callbacks.

## Promises ##

One pattern is a promise, which represents the result of a potentially long running and not necessarily complete operation. Instead of blocking and waiting for the long-running computation to complete, the pattern returns an object which represents the promised result.

An example of this might be making a request to a third-party system where network latency is uncertain. Instead of blocking the entire application while waiting, the application is free to do other things until the value is needed. A promise implements a method for registering callbacks for state change notifications, commonly named the then method:

```
    var results = searchTwitter(term).then(filterResults);
    displayResults(results);
```

At any moment in time, promises can be in one of three states: unfulfilled, resolved or rejected.

To give an idea how the concept works, let’s start out with the CommonJS Promise/A proposal which has several derivatives in popular libraries. The then method on the promise object adds handlers for the resolved and rejected states. This function returns another promise object to allow for promise-pipelining, enabling the developer to chain together async operations where the result of the first operation will get passed to the second.

```
    then(resolvedHandler, rejectedHandler);
```

The resolvedHandler callback function is invoked when the promise enters the fulfilled state, passing in the result from the computation. The rejectedHandler is invoked when the promise goes into the failed state.

We’ll revisit the example above using a pseudo code example of a promise to make the Ajax request to search Twitter, populate the screen with data, and handle errors. Let’s start with an example of what a promise library might look like if we were designing one from scratch with just the very basics. First we’ll need some form of object to keep the promise.

```
    var Promise = function () {
        /* initialize promise */
    };
```

Next, we’ll need to implement the then method which allows us to chain together operations based upon state change of our promise. This method takes two functions for handling when the promise is resolved and for when the promise is rejected.

```
    Promise.prototype.then = function (onResolved, onRejected) {
        /* invoke handlers based upon state transition */
    };
```

We’ll also need a couple of methods to perform a state transition between unfulfilled and resolved or rejected states.

```
    Promise.prototype.resolve = function (value) {
        /* move from unfulfilled to resolved */
    };

    Promise.prototype.reject = function (error) {
        /* move from unfulfilled to rejected */
    };
```
Now that we have some boilerplate for what a Promise object could be, let’s walk through the example from above of querying Twitter for #IE10 tagged tweets. First, we’ll create a method for making an Ajax GET request using the XMLHttpRequest2 to a given URL and wrap it in a promise. Next, we’ll create a method specifically for Twitter calling our Ajax wrapper method with a given search term. Finally, we’ll invoke our search function and display the results in an unordered list.

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

```
    Promise.when = function () {
        /* handle promises arguments and queue each */
    };
```

Now we can queue up multiple promises simultaneously, for example by searching for both #IE10 and #IE9 on Twitter.

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

## Exploring Promises in jQuery and the Dojo Toolkit ##

There are many JavaScript libraries that are available to the developer which implement some form of a Promise. Let’s now explore a few libraries which expose promises or similar concepts.

### The Dojo Toolkit ###

The first widespread use of this pattern was with the dojo toolkit deferred object in version 0.9. Just like the CommonJS Promises/A proposal above, this object exposes a then method which allows the developer to handle both the fulfillment and error states and chain the promises together. The dojo.Deferred object exposes two additional methods; resolve which fulfills the promise, and reject, which sends the promise into the rejected state. Below is an example of using the dojo.Deferred object to make an Ajax request to an URL and parse the results.

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

### jQuery ###

jQuery introduced a new concept in version 1.5 called Deferred which is also a derivative implementation of the CommonJS Promises/A proposal. The Deferred object exposes a then method which allows the developer to handle both the fulfillment and error states. Like dojo, this object also exposes resolve and reject. The developer can create a Deferred object in jQuery by calling the $.Deferred function.

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

The jQuery 1.5 release alters the Ajax methods to now return the jqXHR object which directly implements the promise interface.

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

There are many options available to the developer on how to deal with the complexities of asynchronous programming. With well-known patterns such as promises and deferred objects, and libraries that expose them, the developer is able to create rich interactions which seamlessly bridge asynchronous requests. In this example, we discussed leveraging promises and deferred objects atop XMLHttpRequests but the patterns could easily be layered on top of Web Workers, the setImmediate API, the FileAPI, or any other asynchronous API. You can use common JavaScript libraries so you don’t have to write boilerplate code.

The promises pattern is a good start, but it’s not the end of the solution. In fact, many patterns are emerging to address asynchronous programming. We think this is an interesting development which makes our lives easier as web developers and think it can help you write your web applications as well. Happy coding!

—Matt Podwysocki, JavaScript geek and consultant
—Amanda Silver, Program Manager for JavaScript