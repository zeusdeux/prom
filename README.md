pimp
====
[![Build Status](https://travis-ci.org/zeusdeux/pimp.svg?branch=master)](https://travis-ci.org/zeusdeux/pimp)

A simple (P)romise/A+ spec compliant (imp)lementation that I wrote to wrap my head around it.   
It passes all the tests that are part of the [Promises/A+ Compliance Test Suite](https://github.com/promises-aplus/promises-tests).

##Installation

####For `node`:   
```javascript
npm install pimp --save
```

Then go ahead and `require` it in your `node` projects as:   
```javascript
var pimp = require('pimp')
```

####For browsers:

Install using `bower`: 
```javascript
bower install pimp
```

#####or

You can grab `pimp.min.js` and its source map `pimp.min.map` (optional) from the browser folder   
and import it using the usual `<script></script>` tags anywhere in your code.   
It will add a `Pimp` constructor to the `window` object as `window.Pimp` for you to use.

##API

###Constructor

- [Pimp](#pimpfunction)

###Public

- [then](#thenonfulfilled-onrejected)
- [inspect](#inspect)
- [catch](#catchrejectionhandler)
- [finally](#finallycallback)

###Static

- [resolve](#pimpresolvevalue)
- [reject](#pimprejectreason)
- [cast](#pimpcastvalue)
- [all](#pimpalliterable)
- [allFail](#pimpallfailiterable)
- [race](#pimpraceiterable)
- [deferred](#pimpdeferred)
- [denodeify](#pimpdenodeify)

###Changelog

- [0.2.3](#023)
- [0.2.2](#022)
- [0.2.1](#021)
- [0.2.0](#020)


##Constructor

###Pimp(function)
The constructor returns a `Promise`.   
It accepts a `function` as a parameter and provides the `function` with two parameters, `resolve` and `reject`.     
They can be used to resolve/fulfill or reject the `Promise` that was returned by the constructor.

Example:
```javascript
var prom1 = new Pimp(function(fulfill, reject) {
    setTimeout(function() {
        fulfill(10);
    }, 2000);
});

prom1.then(function(v){
    console.log(v); //logs 10 after approximately 2000ms
});
```

##Public methods

###then(onFulfilled, onRejected)
It returns a `Promise`.   
If/when the returned `Promise` fulfills, it runs the `onFulfilled` handler.   
If/when the returned `Promise` gets rejected, it runs the `onRejected` handler.  
You can call `then` on a Promise as many times you want.   
Also, You can chain as many `then` calls as you want.      
All registered handlers will be run in sequence when the Promise resolves.   

It basically does all that a Promises/A+ compliant `then` method should do.    
Read more [here](http://promises-aplus.github.io/promises-spec/#the__method).

Example:
```javascript
var prom1 = new Pimp(function(fulfill, r) {
    setTimeout(function() {
        fulfill(10);
    }, 2000);
});

prom1.then(function(v) {
    console.log(v); //logs 10
});

prom1.then(function(v) {
    console.log("omg not %o again",v); //logs omg not 10 again
});
```

###inspect()
It tells you the current state of the promise.
It returns an object of the form:
```javascript
{
    state: <state of promise>,
    value/reason: <value/reason of promise>
}
```

###catch(rejectionHandler)
It adds a rejection handler to the promise it is called on.
It returns a new promise that resolves to the value of the parent promise or calls   
`rejectionHandler` with the reason the parent promise rejected with.
It is essentially a short hand for `.then( function(v){ return v; }, rejectionHandler )`.

Example:
```javascript
var p = Pimp.reject(10);
p.catch(function(v){
    console.log(v); //logs 10
});
```

###finally(callback)
It is analogous to `finally` in `try-catch-finally` clause and returns a `promise`.   
It lets you run a `callback` irrespective of the final state of the promise it (i.e., `finally`) is chained to.   
It lets you do so without changing the value/reason/state that the promise it is chained to resolved/rejected with.   
Therefore, you can chain to a `finally` call just like you would if it weren't there i.e., it is transparent.   
If the `callback` given to finally returns a promise, then the resolution of the promise returned by `finally` is delayed till the promise returned by the `callback` resolves.   
It works like [Q's finally](https://github.com/kriskowal/q/wiki/API-Reference#promisefinallycallback) implementation.

Example:
```javascript
var p = Pimp.resolve(10).finally(function(v){
    console.log(v); //logs 10
    return 20;
}).then(function(v){
    console.log(v); //logs 10 (value returned by finally callback is ignored)
});
```

#####Note:
- If the `callback` given to `finally` throws then the `promise` returned by `finally` rejects with that error as reason.

##Static methods

###Pimp.resolve(value)
It returns a `Promise` that is *resolved* to the value passed to it. The value can be any value as specified [here](http://promises-aplus.github.io/promises-spec/#terminology).    
If it's a `thenable` then the returned `Promise` resolves to it as per the [Promises/A+ Resolution Procedure](http://promises-aplus.github.io/promises-spec/#the_promise_resolution_procedure).

Example:
```javascript
Pimp.resolve(10).then(function(v){
    console.log(v); //logs 10
});
```

###Pimp.reject(reason)
It returns a `Promise` whose state is *rejected* and value is the reason provided.

Example:
```javascript
Pimp.reject("DENIED!").then(function(v) {
    console.log("yo %o", v); //this handler doesn't run as the Promise is rejected
}, function(v) {
    console.log(v); //logs DENIED!
});
```

###Pimp.cast(value)
Casts the value to a `Promise` of type `Pimp` and returns the `Promise`.

Example:
```javascript
Pimp.cast(";-;").then(function(v){
    console.log(v); //logs ;-;
});
```

###Pimp.all(iterable)
It returns a `Promise` which resolves when all elements in the iterable resolve.   
Its value is an `Array` of the values returned by each promise in the iterable on their resolution.   
If any value in the iterable rejects, then the Promise returned by `Pimp.all` immediately rejects with its reason.   
It rejects with the value returned by what had rejected in the iterable list passed to `Pimp.all`.   
`Pimp.all` discards all other items in the iterable list irrespective of their state when a reject occurs.   

Example:
```javascript
var p1 = new Pimp(function(resolve, reject) {
    setTimeout(resolve, 5500, "one");
});
var p2 = new Pimp(function(resolve, reject) {
    setTimeout(resolve, 6200, "two");
});

Pimp.all([45, true, p1, p2]).then(function(values) {
    //logs Values resulting from Pimp.all: [45, true, "one", "two"]
    console.log("Values resulting from Pimp.all: %o", values);
});
```

#####Note:
- If any value in the iterable is not `thenable` then it is first cast to a `Promise` using `Pimp.cast` internally

###Pimp.allFail(iterable)
It returns a `Promise` which resolves when all elements in the iterable reject.   
Its value is an `Array` of the reasons returned by each promise in the iterable on their rejection.   
If any value in the iterable resolves, then the Promise returned by `Pimp.all` immediately rejects with its value.   
It rejects with the value returned by what had resolved in the iterable list passed to `Pimp.allFail`.   
`Pimp.allFail` discards all other items in the iterable list irrespective of their state when a resolution occurs.   

Example:
```javascript
var p1 = new Pimp(function(resolve, reject) {
    setTimeout(reject, 200, "one");
});
var p2 = new Pimp(function(resolve, reject) {
    setTimeout(reject, 500, "two");
});

Pimp.allFail([p1, p2]).then(function(values) {
    //logs Values resulting from Pimp.all: ["one", "two"] 
    console.log("Values resulting from Pimp.allFail: %o", values);
});
```

#####Note:
- If any value in the iterable is not `thenable` then it is first cast to a `Promise` using `Pimp.cast` internally

###Pimp.race(iterable)
It returns a `Promise` that adopts the value/reason of the first item in the iterable that resolves or rejects.   

Example:
```javascript
var p1 = new Pimp(function(resolve, reject) {
    setTimeout(resolve, 500, "one");
});
var p2 = new Pimp(function(resolve, reject) {
    setTimeout(reject, 400, "two");
});
var p3 = new Pimp(function(resolve, reject) {
    setTimeout(reject, 1000, "three");
});
var p4 = new Pimp(function(resolve, reject) {
    setTimeout(resolve, 200, "four");
});

var racer = Pimp.race([p1, p2, p3, p4]);
racer.then(function(value) {
    //logs Value resulting from Pimp.race: "four"
    console.log("Value resulting from Pimp.race: %o", value);
});
```

#####Note:
- If any value in the iterable is not `thenable` then it is first cast to a `Promise` using `Pimp.cast` internally

###Pimp.deferred()
It returns an object (a "deferred") of the form:

```javascript
{
    promise: promise,
    resolve: promise.resolve,
    reject: promise.reject,
    inspect: promise.inspect
}
```

Example:
```javascript
var p = Pimp.deferred();
p.resolve(10);
console.log(p.inspect().value); //logs 10
```

###Pimp.denodeify()
It takes a Node.js style, callback accepting function and returns a promise returning function.

Example:
```javascript
var nodeReadFile = require('fs').readFile;
var promisifiedReadFile = Pimp.denodeify(nodeReadFile);
var filePromise = promisifiedReadFile("./path/to/a/file", { encoding: "utf8" });

//assume file contains the data "10"

filePromise.then(function(data){
    console.log(data); //logs 10
    return data;
});
```
##Changelog

####0.2.3
- Added `Pimp.prototype.finally`
- Added more tests
- Bugfixes

####0.2.2
- Bugfixes

####0.2.1
- major performance upgrade for Pimp in the browser
- Pimp for the browser now comes bundled with [setImmediate shim](https://github.com/YuzuJS/setImmediate)
- It now is faster in the browser than [Q](https://github.com/kriskowal/q) according to the tests [here](http://jsperf.com/pimp-vs-bluebird-vs-q-vs-rsvp/4)
- The build process now tests the generated browser code too

####0.2.0
- Added `Pimp.prototype.catch`
- Added `Pimp.denodeify`
- Added `Pimp.deferred`
- Changed how `Pimp.cast` works (it now follows the spec)
- Added support for browsers
- Added automated system for generating Pimp for the browser
- Added a test suite

##Note

- To be able to run `make gen-cov` to generate coverage data, you must install [node-jscoverage](https://github.com/visionmedia/node-jscoverage)
- It must also be available in your PATH env variable so that it can be executed from anywhere
- Since [node-jscoverage](https://github.com/visionmedia/node-jscoverage) isn't a node project, I have not listed it as a dependency but it sure is if   
you want to generate coverage info.
