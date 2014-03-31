---
layout: post
title: "$digest-free event aggregation in AngularJS"
modified: 2014-03-30 23:14:48 -0700
tags: [angularjs performance]
image:
  feature: event_aggregator_feature.jpg
comments: true
---

While Event Aggregator pattern may have a somewhat intricate name, its a relatively simple concept that is fairly widely used in most of the common programming languages and frameworks. As a pattern, its very similar to [Publish/Subscribe](http://en.wikipedia.org/wiki/Publish/subscribe). An Event Aggregator is essentially a centralized message broker that allows interested parties to subscribe and unsubscribe to custom events so that they will be notified when these custom events are triggered.

> If you have ever used an `$("button").on("click", someCallback);` with jQuery, you have used an Event Aggregator to subscribe to an event.

### So why do I need this in AngularJS

AngularJS boasts quite a few notification mechanisms, including scope broadcasts, scope emits, data binding through good old scope inheritance, etc. They all however depend on a digest cycle which can at times be prohibitively expensive.

One such example was described in my previous post [Watch your $watches](http://vlad.io/watch-your-watches/): a digest cycle caused a noticeable pause between the time a user would take his/her finger off the screen after a drag, and inertia-esque transition that should immediately follow. If we needed to change state of some other object on 'dragend', how would we do it without triggering the digest cycle?

### [angular-event-aggregator](https://github.com/vladgurovich/angular-event-aggregator) to the rescue

While jQuery or Angular's jqLite both provide subscribe/unsubscribe or `on`/`off` functions, they are very DOM event specific. I was mainly inspired by [Backbone.js](http://backbonejs.org/) more generic and, to be honest, more readable implementation. [angular-event-aggregator](https://github.com/vladgurovich/angular-event-aggregator) component exposes an injectable `eventAggregator` service that provides 3 basic functions:

* **on(event, callback)**: Register a callback to be executed when event is triggered
* **off(event, callback)**: Unregister a callback from being executed when event is triggered
* **trigger(event, eventObject)**: Triggering an event causes all the callbacks registered for this event to be called with a provided eventObject as an argument.

#### For example:

Register for an event. Note how callback unregisters itself upon execution.

{% highlight js %}
app.controller("MyCtrl", ['eventAggregator', function(eventAggregator){
  var myCallback = function(item) {
    console.log(item);
    eventAggregator.off('some.event.happens', myCallBack);
  };
  eventAggregator.on('some.event.happens', myCallBack);
}
{% endhighlight %}

Trigger the event.

{% highlight js %}
app.controller("SomeOtherCtrl", ['eventAggregator', function(eventAggregator){
  eventAggregator.trigger('some.event.happens', 'myItem');
}]);
{% endhighlight %}

> This service can now be used to change state of different objects without triggering the $digest cycle. With such great power comes great responsibility: please use it responsibly :)

#### TODO

This is a very basic implementation of an Event Aggregator pattern. It lacks features that some other aggregators may have(like `.one()` function to only trigger a callback once for example). I plan on improving it in the coming weeks/months/years and Im definitely open to Pull Requests.
