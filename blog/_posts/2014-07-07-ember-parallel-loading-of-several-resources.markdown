---
layout: default
title:  "Ember.js: parallel loading of several resources"
date:   2014-07-07 12:00:00
categories: blog
tags: emberjs javascript
body_class: "blog"
comments: true
author: ksol
---
We had another opportunity to use Ember.js in a client project recently. The application is a kind of video aggregator: the front page loads many rows of videos, each coming from a different source. Since the various requests can take some time, we want to display a spinner for each row. Nothing too fancy or unusual here.

*I'm assuming you're familiar with the concepts and conventions of `ember.js` here, so if you're not you might want to find a tutorial or follow [the official guides](http://emberjs.com/guides).*

With `ember.js`, the data is fetched from `Route` objects. If you're making one API call (or if you are using `ember-data`), you can return the promise object directly, and `ember` is smart enough to lookup a subroute called "loading" in which you can handle this intermediate state before displaying your route's template. This is usually what you need when you want to display a spinner or a loading text. You can also return a javascript object, or a value, or anything, in which case the "loading" subroute is not used.

But what if you have **many** API calls and not just one? You can use a "meta-promise", ie. a promise that get resolved once every "sub-promise" is resolved. You can achieve this using `RSVP.all` or `RSVP.hash`, and it works great. But it is an all-or-nothing situation: you cannot see the individual progress for each promise, and if one fails, the meta-promise also fails. So we would have a global spinner, not one per row, and if one of the requests fails, we cannot easily display one error for this row, and the other rows correctly.

Of course we could cheat, and do all the requests from the controller at initialization and track their progress, but it feels dirty, and it should since it breaks the responsibilities intended for the different layers in `ember.js`: controllers are not meant to fetch data, this is meant to be done by routes.

Finally, after some research we found something interesting in the API : the [Ember Promise Mixin](http://emberjs.com/api/classes/Ember.PromiseProxyMixin.html). As the name implies, it is a mixin that renders objects promise-aware. Instead of supplying the `content` property (or letting the template do it for you), you need to set the `promise` property. When it gets resolved (or rejected), `content` is set. And you can access properties such as `isPending`, `isRejected`, etc. to follow the lifecycle of your promise and update the template when the promise's state changes.

So, in our route, we fetch all our promises, and returns them as a regular js object (not wrapped in `RVSP.hash`). Then, we're overriding the setup of the `IndexController` and fills the properties we'll be needing: one for each promise, and one array for the promises "labels". In our template, we're iterating over the labels, instanting one `RowController` per label. These controllers are the ones that are promise-aware: we just need to override the initialisation to fetch the associated promise from the `IndexController`, and we're good to go.

The following gist (it's `CoffeeScript`, but should be simple enough to be readable by non-coffeescripters) sums it up:

{% gist ksol/1b47cb87fcc3300cbacb %}

I find this approach more elegant and closer to what ember.js is supposed to feel like. However, it was neither trivial nor easy to find, and it took a bit of guess-work to make things work properly. If you ever had this issue with `ember.js`, I'm curious to see if you solved it differently and how!
