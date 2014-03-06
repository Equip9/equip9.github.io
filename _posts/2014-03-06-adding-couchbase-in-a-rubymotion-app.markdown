---
layout: post
title: "Adding Couchbase Lite to a RubyMotion application"
date: 2014-03-06 17:00
comments: true
categories:
---

After reading this blog post from Adam Wiggins [Mobile syncing with Couchbase](https://medium.com/wandering-cto/6f076d8c7e08) I thouhgt I could give it a try.
First thing you'll notice reading Couchbase [Getting Started](http://docs.couchbase.com/couchbase-lite/cbl-ios/#adding-couchbase-lite-to-your-app)
guide is that you have to include their framework and its dependencies.

* [Vendoring 3rd-Party Libraries](http://www.rubymotion.com/developer-center/guides/project-management/#_vendoring_3rd_party_libraries)
* [How to Use Objective-C Libraries With RubyMotion](http://markvillacampa.com/2012/06/09/how-to-use-objective-c-libraries-with-rubymotion/)

You can do it in your RubyMotion ```Rakefile```

``` ruby
app.frameworks += ['CFNetwork', 'Security', 'SystemConfiguration']
app.libs       += ['/usr/lib/libsqlite3.dylib', '/usr/lib/libz.dylib']
app.vendor_project('vendor/couchbase-lite-community-ios_1.0-beta2/CouchbaseLite.framework',
  :static,
  :products => ['CouchbaseLite'],
  :headers_dir => 'Headers'
)
```

But, after running  ```rake``` I got this error message:

``` ruby
Objective-C stub for message `createDatabaseNamed:error:' type `@@:@^@' not precompiled.
Make sure you properly link with the framework or library that defines this message.
```

I learnt from [this post](https://groups.google.com/forum/#!msg/rubymotion/tPCxSMCA2f4/9dAjRSF0-g4J)
that some vital information is not being generated. You'll notice an ```#ifdef CBL_DEPRECATED``` condition in Couchbase Lite source code which prevents RubyMotion to create some

My approach was to make sure that this constant was defined before ```CBLManager.h``` was reached so RubyMotion could
generate this snippet of code:

```xml
<method selector='createDatabaseNamed:error:'>
<arg declared_type='NSString*' index='0' name='name' type='@'/>
<arg declared_type='NSError**' index='1' name='outError' type='^@'/>
<retval declared_type='CBLDatabase*' type='@'/>
</method>
```

I created a file named ```AAA-rubymotion-fix.h``` with this content:

``` c
//
// Define CBL_DEPRECATED before anything else

#define CBL_DEPRECATED
```
Now you can run ```rake clean``` and ```rake``` again and stub error message won't appear.
