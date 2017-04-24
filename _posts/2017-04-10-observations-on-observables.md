---
layout: post
title: "Observations on Observables and ngrx"
date: 2017-04-10
---

Angular (the javascript framework formerly known as Angular 2), brings with it rxjs, and rxjs is all about Observables.  For anyone using Angular's Http module, Observables are something you've run into already.  You may have pushed them aside cognitively, using `myObservable.toPromise()` to put things right and get your Promise back.

If your using ngrx to manage state, you are already knee deep in Observable waters.  Once you start thinking about the **state** of your application in the redux way, you begin to see lots of opportunities to make your application more predictable and easy to reason about by leveraging Observables.

One place where I wanted to implement the observer pattern was in dealing with authorization (we are talking about who can do what here, not who are you? - which is usually referred to as authentication).  I wanted to be able to use a common 'permissions' scheme in the application api (which is written in node) and in the client.  So my thought was to store a mapping of user roles to permissions on the server, use it for my node api, and provide it to the client at a REST endpoint.  All straightforward so far...

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
> You can simplify the above by processing the json file an overlaying 'super' permissions onto the other permissions.  This has been omitted because it is not jermaine to the goal of understanding Observables.

Here is the basic functionality we are looking for:
- In our Angular app we get this JSON object using ngrx (action/reducer/effect) and place it in the store.
- We have an Angular Service that we ask *authorization* questions of.  e.g. 'can the current user edit this task?'.
- If the permissions change in the store, the answer to our question updates accordingly and that change effects our UI.

To do this following our redux pattern, our 'question' (does the user have permission to...) should return an Observable to which we will subscribe in the Component(s) that need to know about authorization.  We call this Observable `hasPermission()`, and it to the `currentUser` and `permissions`, both of which are Observables as well.  The result is that if the user of the application changes (for instance a role is removed, or the user logs out), or if our permissions scheme changes, the `hasPermission` Observable will emit a new value, and the state of our UI will adapt accordingly.

Time to roll up the sleves and dig in.  I'm going to approach this by creating an Angular Service to handle authorization.  Let's start with our imports:

{% highlight typescript %}
import {Injectable} from '@angular/core';
import {Observable, Subscription} from 'rxjs/Rx';
import {Store} from '@ngrx/store';
import * as _ from 'lodash';

import {Permissions, User} from '../models';
import * as fromRoot from '../reducers';
import * as permissions from '../actions/permissions';
{% endhighlight %}

And next the basics of our service/class:

{% highlight typescript %}
//...

@Injectable()
export class AuthorizationService {

  oPermissions: Observable<Permissions>;
  permissions: Permissions;

  oCurrentUser: Observable<User>;
  currentUser: User;

  isLoading: Observable<boolean>;
  error: Observable<any>;


  constructor(private store: Store<fromRoot.State>) {
    this._init();
  }

  // ...public class methods...

  private _init() {
    this.store.dispatch(new permissions.LoadPermissions(null));

    this.oPermissions = this.store.select(fromRoot.getPermissions);
    this.oCurrentUser = this.store.select(fromRoot.getCurrentUser);

    this.store.select(fromRoot.getPermissionsError).subscribe(error => {
      this.error = Observable.of(error);
    });

    this.store.select(fromRoot.getPermissionsIsLoading).subscribe(isLoading => {
      this.isLoading = Observable.of(isLoading);
    });
  }
 // ...other private class methods...
}

{% endhighlight %}

I start by injecting the store in the class constructor, and then creating local variables which are Observables of the currentUser and permissions from our store (`this.oCurrentUser` and `this.oPermissions`) in the private `_init()` method.

So when Angular injects the service it will 'new' the exported class `AuthorizationService`, and the methods on this class will be available in the component(s) into which it is injected.  Beyond the setup code we've looked at so far, my class has two methods:

{% highlight typescript %}

  // @param {string} permissionGroup e.g. 'tasks'
  // @param {string} permissionType e.g. 'create'
  hasPermission(permissionGroup: string, permissionType: string): Observable<boolean> {
    // console.log('subscribed to hasPermission');
    return Observable.create(observer => {
      try {
        const permissionsSub: Subscription = this.oPermissions.subscribe(res => {
          this.permissions = res;
          observer.next(this._hasPermissions(permissionGroup, permissionType));
        });
        const currentUserSub: Subscription = this.oCurrentUser.subscribe(res => {
          this.currentUser = res;
          observer.next(this._hasPermissions(permissionGroup, permissionType));
        });
        return () => {
          permissionsSub.unsubscribe();
          currentUserSub.unsubscribe();
        };
      } catch (err) {
        observer.error(err);
      }
    });
  }
  
  private _hasPermissions(permissionGroup: string, permissionType: string): boolean {
    let matched = [];

    if (this.permissions[permissionGroup]) {
      const permittedRoles = this.permissions[permissionGroup][permissionType];
      const userRoles = this.currentUser.Roles.map(role => role.name);
      matched = _.intersection(permittedRoles, userRoles);
    }
    return !!matched.length;
  }
{% endhighlight %}

What makes this all work 


