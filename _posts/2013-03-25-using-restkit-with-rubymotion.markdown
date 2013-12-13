---
layout: post
title: "Using RestKit with RubyMotion"
date: 2013-03-25 20:32
comments: true
categories: 
---

The ThunderboltLabs guys have a great post about using [RestKit with RubyMotion](http://thunderboltlabs.com/posts/restkit-object-mapping-with-rubymotion.html). I was trying to follow their instructions, but I noticed that some methods are deprecated in [RestKit](http://restkit.org/) 0.20.0. Now, RestKit uses [AFNetworking](http://afnetworking.com/) and some code changes.

Translating Objective-C code to Ruby is not difficult, but I had some trouble when I had to write a _block_. 

Here is an Objective-C block,

``` objc
[operation setCompletionBlockWithSuccess:^(RKObjectRequestOperation *operation, RKMappingResult *result) {
  NSLog(@"The public timeline Tweets: %@", [result array]);
} failure:nil];
```
 
Translated to a Ruby block in RubyMotion,

``` ruby
operation.setCompletionBlockWithSuccess(
  lambda do |operation, result| 
    puts "The public timeline Tweets: " + result.array.inspect
  end,
  failure: nil)
```



