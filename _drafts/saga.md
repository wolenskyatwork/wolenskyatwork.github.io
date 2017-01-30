---
layout: post
title: "A Comprehensive Overview of Redux-Saga"
categories: redux saga testing
---
**A Little Bit of History**

The Redux-Saga library is based upon two things. One of them is part of the name - Redux-Saga is built to work
by expanding upon the functionality provided by React-Redux. If you are not familiar with React-Redux, I suggest
first getting yourself up to speed here: http://redux.js.org/docs/basics/UsageWithReact.html

Since you need a knowledge of Redux to work with Redux-Saga, its importance to the library is not forgotten. The
second inspiration, lengthily titled the Command and Query Responsibility Segregration Pattern (CQRS), provides
valuable insight into understanding how the Redux-Saga library interacts with your application.

The Command and Query Responsibility Segregration Pattern's formal definition is: 
'A piece of code that coordinates and routes messages between bounded contexts and aggregates'

For Redux-Saga, that 'piece of code' is the middleware itself. The `bounded context and aggregates` loosely translate
to an API you are hitting, and the 'messages' will be the saga functions you actually write for the middleware to
interpret.

If you want a more in-depth discussion of CQRS, you can go here:
https://msdn.microsoft.com/en-us/library/jj591569.aspx

As it applies to Redux-Saga, the CQRS pattern can be simplified to two main points:
-A Transaction at each step has a defined compensating transaction
-All messages are mediated by the process manager, which define the message flow in a single place


The basic setup looks like this:

{% highlight javascript %}
const sagaMiddleWare = createSagaMiddleware();
const store = createStore(
	reducers,
	applyMiddleware(sagaMiddleware)
)
{% endhighlight %}

Ever heard of the principle that 'form follow function' in biology? A little bit of that idea applies here - 
the saga middleware is initialized

So in terms of the traditional REDUX flow: 

You are still using ACTIONS.
You are still using REDUCERS.

SAGA introduces WORKERS/ WATCHERS

SAGA WATCHER is listening to all ACTIONs that are being dispatched.
That ACTION is mapped to a WORKER saga which is responsible for actually talking to the SAGA
middleware and taking any action in response to the return from yields. This the part where
all messages are mediated by the SAGAS, and the message flow is here. Actions can be dispatched
again which are this time picked up by the REDUCERS.

**REDUCERS AND SAGAS ARE BOTH LISTENING TO DISPATCHED ACTIONS**
First reducers are notified, and then sagas
The programmer is responsible for appropriately assigning actions to be dealt with either 
by the REDUCERS or the SAGAS

This certainly adds a layer of complexity - it becomes even more important to decide,
What do you do in the SAGA? - Anything involved in dealing w/ async code and the area between API and reducer
What do you do in the REDUCER? - Anything specifically updating the state
What do you do in the COMPONENT?

To write SAGAS, you need to understand GENERATORS and also what you intend to test

function* name(params) { statement }
returns a GENERATOR object

which is an iterator with a next() method on it. Will run until hits a yield
expression (yield or yield*) will return { done: boolean, value: object }

the specifics of writing a generator function for SAGAs come into play when you need to call
other functions within your generator fucntion.

Here is a basic function:

{% highlight javascript %}
function doThings(first) {
	delay(first, 1000);
	const didSomething = doSomethingElse;
	return didSomething * 2;
}
{% endhighlight %} 

To turn this into a GENERATOR object:
{% highlight javascript %}
function* doThings(first) {
	yield delay(first, 1000);
	const didSomething = yield doSomethingElse;
	yield put(didSomething * 2);
}
{% endhighlight %}

And a testable SAGA generator:
{% highlight javascript %}
function* doThings(first) {
	yield call(delay, 1000);
	yield put(first * 2);
}
{% endhighlight %}

Now we will jump around these three examples a little bit. The big difference between one and two is the yield
- if you are familiar with rubys yield, the idea is similar. <Insert a little more on yield here>
The problem is the side-effect of delay: it happens, but testing that side effect is incredibily difficult.


What happens between two and third is crucial to understanding what REDUX-SAGA claims to be doing better than
the other guys. 
`yield call(delay, 1000)` -> the function that will be called is passed off to the SAGA middleware, which is
actually responsible for executing our function. While the function
runs (with whatever side effects) our generator function is blocked. Saga returns an EFFECT object confirming
that the function has finished and includes any return value. Why is this great? Because we know exactly what
the shape of the object is that we are expecting to come back. So, when we want to test, it is straight forward to
compare doThings.next().value with call(delay, 1000); 

(What does the effect of call(delay, 1000) actually look like?)
No mocking is even needed at this point to test the control flow.


functions are working in order to actually set up tests that are useful. The learning curve between the first 
test shown and testing any other more complicated saga is big.

In a way, REDUX-SAGA takes the complexity of asyncronous programming out of the actual SAGA itself (allowing you
to essentially write your SAGAS in a syncronous style) and moves that complexity to the testing 
framework and understanding generators. And testing each individual yield statement can be tedious.

 I have to point out the danger of this is of course there is no enforcement that any of those tests
are even there - that is on the development team itself. 

What SAGA does even better is allowing for greater control over testing in the control flow.
-takeEvery vs. takeLatest
-running tasks in parallel
-race conditions

In the spirit of functional programming, SAGAS are composable. This is great for composing a SAGA at a macro level

You can also cancel tasks, which takes us back to the CQRS pattern REDUX-SAGA is based on - defining what to 
do on cancel is as easy as REDUCER.ERROR

**CONCLUSION**

