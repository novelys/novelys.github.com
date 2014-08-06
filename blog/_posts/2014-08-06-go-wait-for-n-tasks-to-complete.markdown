---
layout: default
title:  "Go: Wait for n tasks to complete with wait groups"
date:   2014-08-06 12:00:00
categories: blog
tags: golang parallelism
body_class: "blog"
comments: true
author: ksol
---
Though we've been Ruby lovers for the past decade here at Novelys, we're always keeping an eye on other languages and ecosystems. We've been playing around with google's `go` here and then, and recently I was trying to achieve something that I was quite used to do in `javascript`: making `n` parallel requests to various apis, then wait for all of them to complete and move on. In a way, it is the opposite topic of a [previous blog post](http://www.novelys.com/blog/2014/07/07/ember-parallel-loading-of-several-resources.html), in which I did *not* wanted to wait for all of the requests to complete before displaying feedback.

In this case, I simply needed to hit several endpoints and exit. I went with `go` because I knew I would not have my hand on the server where the program would be ran and be able to do a proper ruby install, so a single binary with no dependency was a strong argument in favor of `go`. In the first version of my script-like, things were done sequentially and it was quite slow, between 2 and 5 seconds per request.

Since it's `go` code, it's quite easy to make it parallel by replacing `functioncall(arg1, arg2)` with `go functioncall(arg1, arg2)`, but then the main routine does **not** wait for the go routines to complete and return. What I wanted was something similar to `jQuery.when` or `RSVP.all` for goroutines. I first looked into channels but quickly found that there is a utility in `go`'s standard library that does exactly what I wanted to, `sync.WaitGroup`.

The usage is very straigthforward: for each goroutine launched, add one item to the wait group. Then, when your goroutine has finished, make it tell the wait group that it has completed. The following gist should be clear enough, even if you're not used to `go` code:

{% gist ksol/e0424d2721674913c243 %}

I find `go` to be a very good companion to Ruby, having strengths where the latter is known to be weak. What do you think?
