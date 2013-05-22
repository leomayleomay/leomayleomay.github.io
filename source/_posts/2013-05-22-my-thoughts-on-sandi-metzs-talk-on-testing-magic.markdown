---
layout: post
title: "My thoughts on Sandi Metz's talk on testing magic"
date: 2013-05-22 22:29
comments: true
categories: ruby rails test sandimetz OO
---
Sandi Metz ([@sandimetz](https://twitter.com/sandimetz)) never lets the audiences down, this time, she brought another great talk on testing. I highly recommend you at least walk through the [slides](https://speakerdeck.com/skmetz/magic-tricks-of-testing-railsconf), here is the [video](http://www.confreaks.com/videos/2452-railsconf2013-the-magic-tricks-of-testing) if you are blessed.

I learned a lot from the talk that I don't have to have 100% coverage for the test suite. Here's some pickups you can take away:

* _DO NOT_ test private method, since they are the most unstable part in the whole object
* _DO NOT_ test out-going query message, it has no side effect to the message receiver
* _DO_ test the out-going command message, ensure you have told the message receiver something it should execute on its part. If you heard about ["Tell, don't ask"](http://pragprog.com/articles/tell-dont-ask), you know that one object should __tell__ another object to do something instead of __asking__ the state of another object and make the decision based on the returned result. Since the out-going command message has side effect to the message receiver, you need to make sure the message is sent out
* _DO_ test the incoming query message, just ensure the return result is expected
* _DO_ test the incoming command message, just ensure the changes is made as expected

That's all, happy testing. BTW, keep two things in mind for good OO and testing:

* Dependency injection
* Tell, don't ask