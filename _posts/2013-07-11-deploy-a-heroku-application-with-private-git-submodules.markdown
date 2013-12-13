---
layout: post
title: "Deploy a Heroku application with private git submodules"
date: 2013-07-11 00:19
comments: true
categories: 
---

Have you ever had to include a **private gem** into your rails application? I've read about some alternatives like [Gemfury](http://www.gemfury.com/), but a found pretty easy solution, instead of using your gem, include it as a git submodule in your Heroku application.

**Be careful!! This is not the most secure way. We'll use our private GitHub OAuth token.**

First, we have to create our GitHub token using a HTTP request:

``` bash
$ curl -u <github_username> -d '{"scopes": ["repo"], "note": "generate token"}' https://api.github.com/authorizations
Enter host password for user '<github_username>':
```

That will output a body like this:

``` javascript
{
  "id": 9879871,
  "url": "https://api.github.com/authorizations/9879871",
  "app": {
    "name": "generate token (API)",
    "url": "http://developer.github.com/v3/oauth/#oauth-authorizations-api",
    "client_id": "jdksauc98s932849euw"
  },
  "token": "78fdf6s7d6g7fd6g7df6g7df6g7df6g7df",
  "note": "generate token",
  "note_url": null,
  "created_at": "2013-07-10T12:12:11Z",
  "updated_at": "2013-07-10T12:12:11Z",
  "scopes": [
    "repo"
  ]
}
```

We can use this ```78fdf6s7d6g7fd6g7df6g7df6g7df6g7df``` to add the git submodule:

``` bash
$ git submodule add https://78fdf6s7d6g7fd6g7df6g7df6g7df6g7df:x-oauth-basic@github.com/<github_username>/my-library.git lib/my-library
```

Next time you push your application to Heroku, it'll pull the git submodule using the token to access your private repository.

This can be also done using your *username* and *password* instead of the OAuth token. 