---
layout: post
title: "Observations on Observables and ngrx"
date: 2017-04-10
---

Angular (the javascript framework formerly known as Angular 2), brings with it rxjs, and rxjs is all about Observables.  For anyone using Angular's Http module, Observables are a construct you've run into already.  You may have pushed them aside cognitively, using `myObservable.toPromise()` to put things right and get your Promise back.

If your using ngrx to manage state, you are already knee deep in Observable waters.  Once you start thinking about the 'state' of your application in the redux way, you begin to see lots of opportunities to make your application more predictable and easy to reason about by leveraging Observables.

One place where I wanted to implement the observer pattern in my application was in dealing with authorization.  We're talking about who can do what here, not who are you? (which is usually referred to as authentication).  I wanted to be able to use a common 'permissions' scheme in the application api (which is written in node) and in the client.  So my thought was to store a mapping of user roles to permissions on the server, use it for my node api, and provide it to the client at a REST endpoint.  All straightforward so far...

So we have a `JSON` file that maps user roles to permissions:
{% highlight json %}
{
    "tasks": {
        "super": ["admin", "power user"],
        "create": ["admin", "user", "power user"],
        "delete": ["admin", "power user"],
        "update": ["admin", "power user"],
        "complete": ["admin", "user", "power user"],
        "incomplete": ["admin", "user", "power user"]
    }
}
{% endhighlight %}
> You can simplify the above by processing the json file an overlaying 'super' permissions onto the other permissions.  I've omitted that in the example because it is not jermaine to the goal of understanding Observables.

Here is the basic functionality we are looking for:
- In our Angular app we get this JSON object using a ngrx (action/reducer/effect) and place it in the store.
- We have an Angular Service that we ask *authorization* questions of.  e.g. 'can the current user edit this task?'.
- If the permissions change in the store, the answer to our question updates accordingly and that change effects our UI.

To do this following our redux pattern, our 'question' should return an Observable, to which we will subscribe in the Component(s) that need to know about authorization.  Time to dig in.

My first misconception was that `Observable.of(whatever)` would do the trick. Wrong. What this method does is to create an observable from the arguements it is given.  Once there are no more values in the arguments, the Observable completes, so your observer doesn't get future values.

{% highlight javascript %}
let foo = 'bar';
{% endhighlight %}
