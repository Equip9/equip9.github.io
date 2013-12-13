---
layout: post
title: "Define action parameter in UIBarButtonItem"
date: 2013-03-28 20:39
comments: true
categories: 
---

Although RubyMotion is great, sometimes is hard to know how to use some Cocoa methods or parameters. This is how you translate an action parameter **(SEL)action**:


``` objc
UIBarButtonItem *refreshButton = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemRefresh 
                                                          target:self 
                                                          action:@selector(refreshMeasures)];
```
<br />

``` ruby
refreshButton = UIBarButtonItem.alloc.initWithBarButtonSystemItem(
      UIBarButtonSystemItemRefresh, target:self, action:'refreshMeasures')    
```

where _refreshMeasures_ is a method defined in the class

``` ruby
def refreshMeasures
end
```