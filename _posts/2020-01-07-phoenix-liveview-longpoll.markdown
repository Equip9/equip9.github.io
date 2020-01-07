---
layout: post
title: "Phoenix LiveView with longpoll"
date: 2020-01-07 17:00
comments: true
categories:
---

One of the [Phoenix Framework](https://www.phoenixframework.org/) killer features is [LiveView](https://github.com/phoenixframework/phoenix_live_view), which lets you build real time websites using websockets and [Elixir](https://elixir-lang.org/).

Phoenix LiveView has been built on top of [Phoenix Channels](https://hexdocs.pm/phoenix/channels.html), which use websockets technology to let the client and the server communicate, but might not be very obvious that one can use [longpolling](https://en.wikipedia.org/wiki/Push_technology#Long_polling) as an alternative for websockets when setting up LiveView. As pointed out [here](https://elixirforum.com/t/live-view-fallback-with-no-websocket/26068) LiveView can use the `LongPoll` transport like one would use for normal websockets.

On `assets/js/app.js`, you can pass options as a third argument to `LiveSocket()`.

```js
import {Socket, LongPoll} from "phoenix"
import LiveSocket from "phoenix_live_view"
let liveSocket = new LiveSocket("/admin/ticker/live", Socket, {transport: LongPoll})
liveSocket.connect()
```

And remember to enable longpoll in `lib/demo_web/endpoint.ex`.

```elixir
socket "//live", Phoenix.LiveView.Socket,
  websocket: [connect_info: [session: @session_options]],
  longpoll: true
```
