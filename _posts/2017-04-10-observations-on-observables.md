---
layout: post
title: "Observations on Observables"
date: 2017-04-10
---

Observables are harder.

{% highlight ruby %}
def show
  @widget = Widget(params[:id])
  respond_to do |format|
    format.html # show.html.erb
    format.json { render json: @widget }
  end
end
{% endhighlight %}
