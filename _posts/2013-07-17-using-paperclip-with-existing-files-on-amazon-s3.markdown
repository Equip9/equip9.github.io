---
layout: post
title: "Using Paperclip with existing files on Amazon S3"
date: 2013-07-17 17:27
comments: true
categories: 
---

Paperclip is a very handy ruby gem to deal with media uploaded to Amazon S3. I works fine with content uploaded from your Rails application, but sometimes you'll want to use it as well for files already stored in Amazon S3. In that case, you'll need to manually set the paperclip attributes. I'm using this apporach: [Using Paperclip with files already on Amazon S3](http://stackoverflow.com/questions/3961107/using-paperclip-with-files-already-on-amazon-s3). Since your files are already in S3 you can get the attributes using the [aws-sdk](http://rubygems.org/gems/aws-sdk) gem: [Class: AWS::S3::S3Object](http://docs.aws.amazon.com/AWSRubySDK/latest/AWS/S3/S3Object.html#content_length-instance_method).

``` ruby

def set_file_as_attachment
  s3 = AWS::S3.new
  obj = s3.buckets['your-bucket'].objects['path/you/file.mp4']
  self.attachment_file_name    = 'file.mp4'
  self.attachment_content_type = MIME::Types.type_for(self.media_file_name).first.to_s
  self.attachment_file_size    = obj.content_length
  self.attachment_updated_at   = Time.now
  self.save!
end

```

