---
layout: post
title: "Angular 2 @Input get and set"
date: 2017-04-10
---
Oh the joys of digging deep.  I find myself working with an Angular Component that is displaying detail in a dialog (modal).  Since I have drunk the ngrx/redux Cool-Aid, I'm abandoning my old friend *two-way data binding* in favor of something... different.  Now I still need to bind by view data back to a model, but that model can't be the **store**, or I will find myself mutating **state**.  At which point the redux Police will show up, and well you can imagine the rest.

What to do.

To break it down a bit.  The parent component subscribes to a *slice* of the **store**.  In this case that *slice* is **Tasks**.  My detail componet has an @Input of task, which the parent component binds to:

```html
<my-detail-component [task]="selectedTask"></my-detail-component>
```

In my detail component I receive the input:

{% highlight typescript %}
export class MyDetailComponent implements OnInit {
  _task: TaskBuffer | undefined;

  @Input() set task(value) {
    this._task = this._createBuffer(value);
  }
  
  constructor() { }
  // create and return a buffer so that ngModel does not bind to the store
  private _createBuffer (task): TaskBuffer {
    return Object.assign({}, task);
  }
}
{% endhighlight %}



