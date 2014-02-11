---
layout: post
title: "Ember-rails gem commands"
date: 2014-02-11 19:59
comments: true
categories: 
---

This is a personal note for remembering basic ```ember-rails``` commands:

# Installation

First, you'll need to add a couple of gems into your ```Gemfile```

``` ruby
gem 'ember-rails'
gem 'ember-source'
```

Then, create a scaffold for your Ember app and install source files (ember & ember-data)

```
$ bin/rails generate ember:bootstrap
% bin/rails generate ember:install
```
 
This will create a bunch of new folders and directories.

## Basic commands


Create a new <i>tales</i> route:

```
$ bin/rails g ember:route tales
```

Create <i>tales</i> template:

```
$ bin/rails g ember:template tales
```

Create new <i>tale</i> form template:

```
$ bin/rails g ember:template tables/new
```