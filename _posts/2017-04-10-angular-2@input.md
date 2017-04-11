---
layout: post
title: "Angular 2 @Input get and set"
date: 2017-04-10
---
Oh the joys of digging deep.  I'm working with a classic master/detail scenario in Angular 2, where the detail is dialog (modal) Component.  The common approach to this is to get the collection (in this case tasks) in the master, and then to pass the selected task to the *dumb* or presentational Component using a property on the detail Component `class` which is decorated with `@Input`. [Checkout the Angular Guide for a great walkthrough of how and why to use @Input](Input guide).

Now, since I have drunk the ngrx/redux Cool-Aid I can't rely on my old friend *two-way data binding* in the way I used to.  I still need to bind my form inputs data back to a model, but that model can't be the **store**, or I will find myself mutating **state**.  At which point the redux Police will show up, and well you can imagine the rest.

What to do.

To break it down a bit.  The parent component subscribes to a *slice* of the **store**.  In this case that *slice* is **Tasks**.  My detail componet has an @Input of task, which the parent component binds to:

{% highlight xml %}
<my-detail-component [task]="selectedTask"></my-detail-component>
{% endhighlight %}

In my detail component I receive the input:

{% highlight typescript %}
export class MyDetailComponent implements OnInit {
  _task: TaskBuffer | undefined;

  @Input() set task(value) {
    this._task = this._createBuffer(value);
  }
  
  constructor(
  private store: Store<fromRoot.State>
  ...
  ) { }
  
  // create and return a buffer so that ngModel does not bind to the store
  private _createBuffer (task): TaskBuffer {
    return Object.assign({}, task);
  }
}
{% endhighlight %}

[Input guide]: https://angular.io/docs/ts/latest/guide/template-syntax.html#!#inputs-outputs

