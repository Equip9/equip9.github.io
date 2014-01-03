---
layout: post
title: "Simple Document Versioning with CouchDB and Rails"
date: 2014-01-03 16:59
comments: true
categories: 
---


I'm new to [CouchDB](http://couchdb.apache.org/), but there are 2 built-in features that have caught my attention:

1. Versioning system using _document revisions_
2. Web admin inteface: [Futon](http://wiki.apache.org/couchdb/Getting_started_with_Futon)

Versioning system using document revision
=========================================

Each document update is a new revision to that document, so you won't get data corruption easily in case your connection dropped. This is an examples of creating a new document using [CouchRest](http://rubygems.org/gems/couchrest) ruby gem:


``` ruby
DB = CouchRest.database!("http://localhost:5984/database-name")
DB.save_doc({team: "FC. Barcelona", player: "Messi"})
# => {"ok"=>true, "id"=>"9c6182ff604654baa0d23aa7b73eb09d", "rev"=>"1-0842f1a97b2a32ea71faa8a152156f98"} 
```

Updating a document (creates a new revision)

``` ruby
DB = CouchRest.database!("http://localhost:5984/database-name")
doc = DB.get("9c6182ff604654baa0d23aa7b73eb09d")
DB.update_doc(doc.id) {|doc| doc["player"] = "Leo Messi"}
# => #<CouchRest::Document _id: "9c6182ff604654baa0d23aa7b73eb09d", _rev: "2-f47d99f2b9544508e83ab3af877f39f1", team: "FC. Barcelona", player: "Leo Messi">
```

Notice same document ```_id``` but different ```_rev```!


Futon: CouchDB simple web interface
===================================

This is a nice feature when building a new app using CouchDB and you want a quick check or access to your documentms. Instead of start using cURL you can use built-in Futon web interface opening this URL in your browser: 

```
http://localhost:5984/_utils/
```
