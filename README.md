# future.lucee
A Futures Implementation for Lucee

Lucee has multiple concurrency features between async functions, tasks and thread {}, but working with the underlying thread implementation is cumbersome to work with. A Future provides syntactic suger over the use of thread {} by executing the task in a thread and giving back a handle to check on completion to deal with the result.

##How to Use
Future.lucee is a single class, future.cfc. The basic use is to create a new future and pass a closure to it that will be executed.

```coldfusion
<cfscript>
future = new future(function(){
	sleep(2000);
	return "some value"
});

//Blocks the thread and waits for the result
echo(future.get());
</cfscript>
```

##Why to Use
A future is useful in the scenario where you have a background task that you want to execute, but will need the result of the task at some point during the request. For example, say you have to make an HTTP request to a third party resouce to get some data, and do some local querying for other data, and then when these are both complete, output them to the browser.

You don't need to use a future when you do not care about the result of the task. In this case, a simple thread {} will suffice.

##Features
###Result Callbacks
Future can optionally take callbacks which will be called at the appropriate time. All of the callbacks execute asynchronously along with the task. Thus is the task is canceled, none of these functions can be guaranteed to have executed. The constructor signature is:
```
public function init(required function task, 
		     function success, 
		     function error, 
		     function finally){
```
####task()
The primary closure to execute 

####success(any result)
A closure which will recieve the result of the task if there was any

####error(any error)
A closure which will receive the result of the error when executing the task

####finally(any result) or finally(any result)
A closure which will always execute and receives either an error, or the result of the task. The value is passed as a named parameter. To have the closure check if it was a result or an error, in the body of the closure use: `structKeyExists(arguments,"error")` or `structKeyExists(arguments,"result")`

###Chainable
Futures can be chained using the then() function which will pause execution of the next future until the first future completes. Chainable futures would be used when you want to return control to the calling page that created the chain, but each future must execute sequentially.

```coldfusion
<cfscript>
future = new future(function(){
	sleep(1000);
	return 10;
}).then(new future(function(priorFuture){
	sleep(1000);
	priorValue = priorFuture.get();
	return "20" + priorValue;
}));

echo("The result was: #future.get()# and took #future.elapsed()# ms to finish");
</cfscript>
```

The echo produces `The result was: 30 and took 2019 ms to finish` which we can see that it took the time for both futures to complete.

##Thread Saftey
The closure passed to the future contains references to the scopes of where it was created. Be careful about race conditions when accessing the variables scope or global scope. You should `var` any local variables inside the closure to ensure no race conditions with the calling page. You should lock {} any access to global scopes

##Limitations
Future makes use of the Lucee thread tag which cannot support nested threads. Therefore any code blocks passed to future will error with `"could not create a thread within a child thread"` if there was a nested future. If you need to do further parallel processing within the executing code, make use of one of Lucee's parallel functions: map(), each(), every(), reduce(), some(), filter()

The example nestedMap.cfm shows how to do this.
