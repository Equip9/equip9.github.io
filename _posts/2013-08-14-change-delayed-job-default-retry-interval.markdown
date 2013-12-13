---
layout: post
title: "Change Delayed::Job default retry interval"
date: 2013-08-14 22:53
comments: true
categories: 
---

By default [Delayed::Job](https://github.com/collectiveidea/delayed_job) sets the interval between retries using this formula increasing in time:

``` ruby
next_time = last_time + (attempts ** 4) + 5
```

As [Greg Lazarev](https://github.com/gylaz) points out in this [anwser](http://stackoverflow.com/questions/16226922/delayed-job-configure-run-at-and-max-attempts-for-a-specific-job) you can modify the default interval defining a ```reschedule_at``` method in your class. For example:

``` ruby
@video = Video.new
@video.delay.track_encoding
```

Where this is the Video model:

``` ruby
class Video
  def track_encoding
    # your background job code here
  end
  def reschedule_at(attemtps, time)
    Time.now + 3600 * 2 # 2 hours from now
  end
end
```

This is the Delayed::Job source code: [base.rb](https://github.com/collectiveidea/delayed_job/blob/95deddc99af84e652dd865a292e959db4b0a1179/lib/delayed/backend/base.rb#L131-135)