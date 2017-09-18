JavaScript caching

There are situations when we need to quite often yank any DOM element (for example, using the querySelector). But unfortunately, this method is quite voracious and because of it the performance drops very badly. Or, for example, you often need to make heavy requests to the server within a single session. Or use functions that take something long.

Fortunately, we can take advantage of JavaScript's capabilities and write a cache function that can cache the results of other methods so that you do not need to call these methods again if necessary:

function cache(key, value) 
{
    if (typeof value == 'undefined') {
        return cache[key];
    }
    cache[key] = value;
}
Now for examples, consider how the above cache function will behave.

Example 1. Caching querySelector

// _io_q - обертка под querySelector, сохраняющая данные в кэш
_io_q = function(selector)
{
    if (!cache(selector)) { 
        cache(selector, document.querySelector(selector));
    }
    return cache(selector);
}
Now let's look at the speed of _io_q and querySelector:

console.time('regular querySelector');
for (var i = 0; i < 1000000; i++) {
    document.querySelector('h1');
}
console.timeEnd('regular querySelector'); // regular querySelector: 100.6123046875ms


console.time('cached _io_q');
for (var i = 0; i < 1000000; i++) {
    _io_q('h1');
}
console.timeEnd('cached _io_q'); // cached _io_q: 5.77392578125ms
Example 2. Caching requests from the server

Let's write a function that sends a very heavy request to the server:

function longRequestToServer(params)
{
    var key = params.endpoint + '_' + params.value;
    if (!cache(key)) {
        var result = 0;
        for (var i = 0; i < 999999999; i++) {
            result++;
        }
        cache(key, result);
    }
    return cache(key);
}
Let's see how the function will behave:

console.time('first run');
longRequestToServer({
    endpoint : '/loadExample', 
    value : 10}
);
console.timeEnd('first run'); // first run: 1012.068115234375ms

console.time('second run');
longRequestToServer({
    endpoint : '/loadExample', 
    value : 10}
);
console.timeEnd('second run'); // second run: 1.31884765625ms

console.time('other request');
longRequestToServer({
    endpoint : '/loadSomeOtherData', 
    value : 15
});
console.timeEnd('other request'); // other request: 1033.783203125ms
TL; DR

Using the capabilities of JavaScript'a can effectively cache data, gaining in performance. But be careful, because the performance gains can be lost on resources and by inadvertency you can get a memory leak.