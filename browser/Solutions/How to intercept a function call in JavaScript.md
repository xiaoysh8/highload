How to intercept a function call in JavaScript

How to simply make a convenient debugging and not crawl into the code or how bearded hackers intercept ajax requests , violating your security.

Listener of a method call to an object

Let's say we have an app object , it has a doStaf () method and we need to find out what comes to it.

For this we need to execute the following code:

(function(doStaf) {

   app.doStaf = function() {

       // здесь выводим все аргументы, с которыми был вызван app.doStaf()
       console.log('app.doStaf args', arguments);
                
       return doStaf.apply(this, arguments);
   };
    
})(app.doStaf);
This code uses an anonymous function to override app.doStaf () function in which we perform the manipulations we need with arguments, and then call the old app.doStaf () , which was saved in the closure.

Next, for convenience, wrap all this code in the function:

function listenCall (method, callback, obj) {
    if (typeof method !== "string" || typeof callback !== "function") return;
        
    obj = obj || window;
            
    (function(objMethod) {

        obj[method] = function() {
            try {
                callback.apply(obj, arguments);
            } catch (e) {}
                
            return objMethod.apply(obj, arguments);
        };
    
    })(obj[method]);
}
Note the order of the arguments: first send the string-name of the method or function that you want to listen to (intercept), the second is the callback that you need to execute to call a function or method, and only the third is passing the object whose method you want to intercept. This is done, because the third parameter is not mandatory and if it is not, the method will be taken from the window object.

This function can intercept the call of only globally declared objects or functions.

Also, the callback call is wrapped in a try . This is necessary in order not to break the execution of the function in the event that there are errors in your code.

Intercept ajax-request

At the end, using the function described above, we intercept the call of any ajax-request by hanging the listener of the send () method on the xmlHttpRequest object :

listenCall('send', function () {

    // запрос получил ответ
    if (this.readyState == 4) {

        console.log('Запрос успешно перехвачен!');
        
    }

}, xmlHttpRequest);
Now, having the necessary skills, you can intercept both calls to console.log () , and, for example ... requests within the payment system.