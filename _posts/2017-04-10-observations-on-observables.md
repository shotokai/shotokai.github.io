---
layout: post
title: "Observations on Observables and ngrx"
date: 2017-04-10
---

I recently started working with ngrx in an Angular project.  Angular (the javascript framework formerly known as Angular 2), brings with it rxjs, and rxjs is all about Observables.  For anyone using Angular's Http module, Observables are a construct you've run into already.  You may have pushed them aside cognitively, using `myObservable.toPromise()` to put things right and get your promises back.

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
> you could simplify the above by processing the json file an overlaying 'super' permissions onto the other permissions.  I've omitted that in the example because it is not jermaine to the goal of understanding Observables.

{% highlight javascript %}
let foo = 'bar';
{% endhighlight %}
