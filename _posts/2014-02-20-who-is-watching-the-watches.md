---
layout: post
description: Tips for optimizing your AngularJS application's performance.
title:  "Watch your $watches"
tags: [angularjs, performance]
comments: true
image:
  feature: watches_feature.jpg
---
I worked on a fairly large-scale mobile web application driven by AngularJS for the past 6 months, so when I was asked if there is anything that I could recommend a budding AngularJS developer looking to ensure predictable performance and responsivenes, my near-instinctual response was to "watch your $watches".

> Data binding is in my opinion one of the most important features that AngularJS brings to the table.

It is the engine that drives interactivity of the application, allowing changes in one part of the application to trigger reactions in another. Like any powerful engine, its lends itself to abuse(oftentimes unintended) which comes at a cost of reduced performance, especially on low powered mobile devices.

## Runaway watches

{% highlight html %}
{% raw %}
<div class="article-row" ng-repeat="article in articles" ng-click="openarticle($index)">
  <div class="article-author">
    <img ng-src="{{article.author.avatarUrl}}">
    <span class="author-name">{{article.author.name}}</span>
  </div>
  <div class="article-title">{{article.title}}</div>
  <div class="article-date">{{article.date}}</div>
</div>
{% endraw %}
{% endhighlight %}

> Angular internally creates a watch for every ng-XX directive and for every double-curly-brackets interpolation.

In the example above, since there are 4 implicit watches inside this `ng-repeat` block, if there were 50 articles we would end up creating 201 watches(1 extra comes from `ng-repeat` itself which has to watch the "articles" in order to react to changes in that array).


Now this is an overly simplified template which represents only a fraction of views used in an average AngularJS application. Yet its very easy to inadvertently cause it to create hundreds of watches even though the data that this template represents is unlikely to change during the lifecycle of the app.


So why do we need to keep watching them?

## So I got many watches, so what?!

Some might argue that dirty checking is insanely fast and only take milliseconds to complete, so its ok to have many of them. These people are both right and wrong.

They are right because on a modern PC in a modern browser dirty checking is in fact fast and digest cycles may take mere milleseconds.

They are also wrong, because thats not always the case on a low power mobile device.


Performance implications of a digest cycle become especially apparent when we are trying to integrate smooth UI animations and transitions with user input such as touch or drag which(depending on how its implemented) may trigger digests during said animations and transitions and introduce choppiness into that interaction. The more watches there are and the longer it takes to evaluate them, the higher the chance of the digest to interfere with an animation or a transition and cause a visible UI framerate drop.

## The solutions and the workaround

Now that we have shown that a lot of watches and errant digest cycles can be detrimental to perceived performance, how does one avoid or reduce their impact? I got few suggestions for you:

#### Use `bindonce`

Use a set of awesome [bindonce](https://github.com/Pasvaz/bindonce) directives to reduce the number of watches. Bindonce waits untill the expression is evaluated once and then unbinds the watch, making that value essentially immutable.

{% highlight html %}
{% raw %}
<div class="article-row" bindonce `ng-repeat`="article in articles" ng-click="openarticle($index)">
  <div class="article-author">
    <img bo-src="article.author.avatarUrl">
    <span class="author-name" bo-text="article.author.name"></span>
  </div>
  <div class="article-title" bo-text="article.title"></div>
  <div class="article-date" bo-text="article.date"></div>
</div>
{% endraw %}
{% endhighlight %}

Once it has rendered, even with 50 articles,  this example would use only 1 watch -- the one used by `ng-repeat`

#### Use `ng-if`

Whenever possible, use `ng-if` instead of `ng-show` or `ng-hide`. The difference between `ng-if` and `ng-hide` is that an element with `ng-hide` is simply hidden with `'display:none'`, but it still remains on the DOM with all the corresponding scope watches still being evaluated. `ng-if` on other hand removes the element from DOM which also destroys the associated watches.

#### Dont carpet bomb the watches

If your use case allows for it, throttle or debounce the scope changes. There is absolutely no need to trigger the digest cycle every 50ms if every 250ms would be good enough. I like to use the absolutely awesome [lodash](http://lodash.com/) library, but if you want you can implement throttling in house easily.

#### If all else fails, use <del>brute force</del> old school

I have come to terms that there are situations when you do not want to gamble invoking the digest cycle at all.

Recently I came across a situation like that when I was simulating a native swipe by doing a gpu-accelerated transfrom on a 'drag' event and when a user would release his or her finger, I would initiate a gpu-acellerated transition to simulate inertia. Whenever I tried updating a scope variable on 'dragend', therefore triggering a digest cycle, there was a distinct pause between the 'dragend' event and the beginning of that inertia transition.

> This was an example when milliseconds count.

My solution was to go where no AngularJS developer likes to go, but sometimes they really must: I ventured outside the AngularJS lifecycle and implemented a good old *Observer/Observable* pattern. More on this in my next post.


