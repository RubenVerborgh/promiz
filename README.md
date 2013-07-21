promiz
======
<a href="http://promises-aplus.github.com/promises-spec">
    <img src="http://promises-aplus.github.com/promises-spec/assets/logo-small.png"
         align="right" alt="Promises/A+ logo" />
</a>
A proper compact promise (promises/A+ spec compliant) library.

## What are promises?
```javascript
function testPromise(val) {
    // An example asyncronous promise function
    var deferred = promiz.defer()
    setTimeout(function(){
        deferred.resolve(val)
    }, 0)
    return deferred
}
testPromise(22).then(function(twentyTwo){
    // This gets called when the async call finishes

    return 33
}).then(function(thiryThree){
    // Values get passed down the chain. Simple right?
    // Now lets return a promise instead of a value

    return testPromise(99)
}).then(function(ninetyNine){
    // The inner promise was resolved asyncronouslu, and its value passed in
    // If you've ever used the async library, it's akin to a waterfall
    // Finally, one last mind bending trick

    return [testPromise(11), testPromise(33), testPromise(55)]
}).all().then(function(list){
    // list === [11, 33, 55]
    // Yeah, try doing that with async (obviously you could, but it would look hideous)
    // and just because we can
    return list
}).spread(function(eleven, thirtyThree, fiftyFive){
    // There you go, now you have a general idea of how promises work
    // To asnwer the original question, a promise is just a deferred object with a `then` function

// .done() makes sure that if any errors occured durring execution, they get thrown
}).done()
// alternatively, catch errors with the `catch` function
// .fail(function(err){ })
```

## Building your own promises with promiz
Promiz has many helper functions to help you convert regular functions into promises
```javascript
// Resolve/Reject defer()
function testPromise(val) {
    // create a new instance of a deffered object (a `promise`)
    var deferred = promiz.defer()
    setTimeout(function(){
        if (val === 42) {
            deferred.resolve('correct')
        } else {
            // This throws an error, which can be caught by .catch() or .done()
            deferred.reject(new Error('incorrect input'))
        }
    }, 0)
    return deferred
}
testPromise(42).then()

// fcall (function call)
function testFn(val){
    if (val === 42){
        return 'correct'
    }
    throw new Error('incorrect input')
}
promiz.fcall(testFn, 42).then()

// nfcall (node function call)
function nodeFn(val, callback) {
    if (val === 42) {
        return callback(null, 'correct')
    }
    return callback(new Error('incorrect input'))
}

promiz.nfcall(nodeFn, 42).then()
```

## Promise methods
#### .then()
```javascript
// .then() takes an optional second error handler, which will catch all errors from previous calls including the current success call
promise.then(function success(){}, function error(){})
```
#### .spread()
```javascript
// .spread() calls .all() and then `applys` over the target function
promise.then(function(){ return [promise(), promise()] }).spread(function(one, two){ })
```
#### .all()
```javascript
// resolves all promises in the result array
promise.then(function(){ return [promise(), promise()] }).all().then(function(list){ /* all promises have been resolved */})
```
#### .fail()
```javascript
// catches any errors that have been thrown thus far
promise.then(function(){ throw new Error('hello') }).fail(function(err){ })
```
#### .done()
```javascript
// If any errors have not been caught by this point, throw them.
// This ends the promise chain
promise.done()
```
#### .nodeify()
```javascript
// Sometimes you may need to support both promises and callbacks (eg. a developer on your team doesn't know promises)
// This function allowes you to create dual functions, that can act like both
function dualFunction(/* optional callback */ callback){
    return promise.nodeify(callback)
}
// you can use it as a promise function
dualFunction().then()
// or a node-style callback function
dualFunction(function(err, val){ })
```

### Notes
 - If your entire promise chain is comprised of syncronous functions, the promise will run syncronously
## Licence: MIT