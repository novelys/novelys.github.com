---
layout: default
title:  "Building an app with Ember.js using HTML5 Storage APIs"
date:   2014-02-15 12:00:00
categories: blog
tags: salesbook emberjs rails
body_class: "blog"
comments: true
fb_image_path: "/img/blog/bdv-ember.png"
---

At Novelys, we have been developping a sales book for <strong>Eckes-Granini</strong> for a few years. At some point, they contacted us regarding some features they wanted to add to the application. With the client, we decided it was best to start from scratch, remove the cruft, and benefit from the best technologies that were now available. This post is going to dive into some of the technological choices we made and why.

### Why start over?

The application in its previous version was starting to get old. While it was almost always running on the latest versions of Ruby on Rails, the front end was handled by both Rails and `prototype.js`, and became quite huge over the years. Anyone remember Google Gears? It was used in the first versions, when it was the only way to make web application available offline. At the same time, better technologies became available to solve the original problem of making the salesbook available offline. And it was a classic case of a server side application, but `prototype.js` was doing all the heavy lifting in terms of interface and front-end logic.

It made complete sense to start over and split the application in two: the backend on one side exposing an API, and the client on the other side, consuming this API. The backend is still in Rails, and looks pretty much the same as before, though lots of code was removed since most of the logic was moved client-side. The API being mostly CRUD-oriented, it was up and running pretty quickly.

### Ember.js + BaaB (Browser As A Backend)

When doing full client-side applications, there's a **lot** of javascript frameworks that you can use. We have a few projects using `backbone.js`, but we felt it would not be the best fit for this case, and to be honest, we wanted to try <a href="http://emberjs.com">ember.js</a>, which was stable for a few weeks. After all, we were building an ambitious web application :)

The application has more than two layers, though. There is the API and the client, but the browser itself serves as a secondary backend. In order to make the application available offline, we have combined several technologies.

The HTML Application Cache, which was previously used for everything that was supposed to be available offline - assets AND documents - is now used only for the assets. We are using the `localStorage` API as a secondary backend: when offline, the model layer in `ember.js` consumes `localStorage` instead of the REST API.

### Storing (big) data in the browser

For the files themselves, we had many choices: `localStorage`, `WebSQL`, `IndexedDB`, `FileSystem` APIs... The first being fairly limited in terms of storage size, and the second being kind of deprecated, they were ruled out quickly. When it came to chose between the two remaining... Well, we *did* chose, but we are actually using both.

We felt that the `FileSystem` APIs were more suited to what we wanted to achieve than `IndexedDB`, but we had one constraint: the salesmen computers were running Firefox, and the FS APIs are available only with WebKit. We decided to use them anyway, since a <a href="https://github.com/ebidel/idb.filesystem.js/">polyfill</a> (using `IndexedDB` under the hood) was available. And on top of the polyfill, we used <a href="https://github.com/ebidel/filer.js">filer.js</a>, which allowed us to use those APIs in a way very similar to unix commands... And we made `filer.js` promise-compliant so that it looks nice to use with `ember.js`.

### To sum up

In the end, we are pretty satisfied with what we ended up building and what we explored: for starters, `ember.js` is an impressive piece of technology. There is room for improvement: `ember-data` is still in beta, and `queryParams` support in the router is experimental, among others. That being said, you can do some pretty serious stuff with, and it us moving in a good direction at a right pace. We will definitely use it again.

It was also interesting to toy with some of the latest HTML5 APIs. The different choices available can all make sense depending on what you are trying to achieve and the constraints you have, so in the end it is up to you. But it is always good to see the web being able to do more than it previously was :)

We would also like to thank <a href="https://github.com/ebidel">Eric Bidelman</a> for his work related to the FS APIs. He made both the polyfill mentionned earlier and `filer.js`, and wrote <a href="http://www.html5rocks.com/en/tutorials/file/filesystem/">a comprehensive</a> article on the topic.

And for those who are not satisfied with the FS APIs, Mozilla has released <a href="https://github.com/mozilla/localForage">localForage</a> a few days ago, which wraps `localStorage`/`IndexedDB`/`WebSQL` in a unified API.

<hr />

If you have questions regarding these matters, you should <a href="https://twitter.com/ksol">hit me up on twitter</a>
