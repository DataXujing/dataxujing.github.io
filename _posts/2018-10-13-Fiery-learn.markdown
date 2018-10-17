---
layout: post
title: "Fiery 教程"
img: bowen56.png
date: 2018-10-13 12:00:00 
description: You'll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [R,Fiery]
---

## Fiery Example

> Fiery是一个灵活轻量级的框架，用于在R中购进Web服务器。

Fiery和Shiny的区别是什么？

+ Shiny uses magic to make everything work from R, Fiery lets you do all the hard work.
+ Shiny wants the main app-logic to be server-side, Fiery don't care what you do.
+ Shiny uses a reactive model to define the app-logic, Fiery don't care what you do (see a pattern emerge).
+ Shiny wants you to use htmltools to build the html, Fiery really don't care what you use.


Fiery是围绕一个清晰的服务器生命周期设计的，在生命周期中的特定点触发事件，这些事件将调用附加到这些事件的处理程序。除了生命周期事件之外，还可以触发自定义事件并将处理程序附加到这些事件中。Fiery的设计考虑到了模块性，因此可以针对不同的任务开发插件，并根据具体项目进行混合和匹配。

虽然介绍可能表明Fiery很难使用，但事实并非如此。处理http请求的许多艰巨工作都封装在处理http请求和响应的`reqres`中。此外，经常使用的插件是`routr`，它提供了HTTP请求的强大路由，从而进一步简化了服务器逻辑。

```r
library(fiery)

# 创建新的APP

app <- Fire$new()

# 每次启动时都设置数据

app$on('start',function(server,...){
  server$set_data('visits',0)
  server$set_data('cycles',0)
})



# 计算循环次数（内部循环）

app$on('cycle-start',function(server,...){
  server$set_data('cycles',server$get_data('cycles')+1)
})

# 计算请求次数

app$on('before-request',function(server,...){
  server$set_data('visits',server$get_data('visits')+1)
})


# 处理请求

app$on('request',function(server,request,...){
  response <- request$respond()
  response$status <- 200L
  response$body <- paste0('<h1> This is indeed a test. You are number',
  	server$get_data('visits'),'</h1>')
  response$type <- 'html'
})

# 在console中show请求次数

app$on('after-request',function(server,...){
  message(server$get_data('visits'))
  flush.console()
})


# 结束掉server在50轮后

app$on('cycle-end',function(server,...){
  if(server$get_data('cycles') > 50){
    message('Ending...')
    flush.console()
    server$extinguish()
  }
})

# Be polite

app$on('end',function(server){
  message('Goodbye')
  flush.console()
})


app$ignite(showcase=TRUE)

```

通常，大部分逻辑将发生在请求和消息处理程序中，如果不需要其他生命周期事件，则可以自由地忽略它们。

## R topics documented

1.delay

因为R默认是单线程的，如果server中有计算时间比较长的内容，可以使用delay。

```r
app$delay({
# Heavy calculation
}, then = function(res, server) {
# Do something with 'res' (the result of the expression) and 'server' the
# server object itself
})
```

2.event

fiery是基于事件的模型策略，整个的server周期中，会不断地有事件触发，事件的处理函数可以通过`on()`方法实现。除了预定义的生命周期事件之外，还可以触发自定义事件，使用` trigger()`方法。不允许手动触发生命周期事件。

+ Life cycle Events
  - `start`： Will trigger once when the app is started but before it is running. The handlers will receive the app itself as the server argument as well as any argument passed on from the ignite()method. Any return value is discarded.
  - `resume`： Will trigger once after the start event if the app has been started using the reignite() method. The handlers will receive the app itself as the server argument as well as any argument passed on from the reignite() method. Any return value is discarded.
  - `end`： Will trigger once after the app is stopped. The handlers will receive the app itself as the server argument. Any return value is discarded.
  - `cycle-start`: Will trigger in the beginning of each loop, before the request queue is flushed. The handlers will receive the app itself as the server argument. Any return value is discarded.
  - `cycle-end`: Will trigger in the end of each loop, after the request queue is flushed and all delayed, timed, and asynchronous calls have been executed. The handlers will receive the app itself as the server argument. Any return value is discarded.
  - `header`: Will trigger every time the header of a request is received. The return value of the last called handler is used to determine if further processing of the request will be done. If the return value is TRUE the request will continue on to normal processing. If the return value is FALSE the response will be send back and the connection will be closed without retrieving the payload. The handlers will receive the app itself as the server argument, the client id as the id argument and the request object as the request argument
  - `before-request`: Will trigger prior to handling of a request (that is, every time a request is received unless it is short-circuited by the header handlers). The return values of the handlers will be passed on to the request handlers and can thus be used to inject data into the request handlers (e.g. session specific data). The handlers will receive the app itself as the server argument, the client id as the id argument and the request object as the request argument
  - `request`: Will trigger after the before-request event. This is where the main request handling is done. The return value of the last handler is send back to the client as response. If no handler is registered a 404 error is returned automatically. If the return value is not a valid response, a 500 server error is returned instead. The handlers will receive the app itself as the server argument, the client id as the id argument, the request object as the request argument, and the list of values created by the before-event handlers as the arg_list argument.
  - `after-request`: Will trigger after the request event. This can be used to inspect the response (but not modify it) before it is send to the client. The handlers will receive the app itself as the server argument, the client id as the id argument, the request object as the request argument, and the response as the response argument. Any return value is discarded.
  - `before-message`: This event is triggered when a websocket message is received. As with the before-request event the return values of the handlers are passed on to the message handlers. Specifically if a 'binary' and 'message' value is returned they will override the original values in the message and after-message handler arguments. This can e.g. be used to decode the message once before passing it through the message handlers. The before-message handlers will receive the app itself as the server argument, the client id as the id argument, a flag indicating whether the message is binary as the binary argument, the message itself as the message argument, and the request object used to establish the connection with the client as the request argument.
  - `message`: This event is triggered after the before-message event and is used for the primary websocket message handling. As with the request event, the handlers for the message event receives the return values from the before-message handlers which can be used to e.g. inject session specific data. The message handlers will receive the app itself as the server argument, the client id as the id argument, a flag indicating whether the message is binary as the binary argument, the message itself as the message argument, the request object used to establish the connection with the client as the request argument, and the values returned by the before message handlers as the arg_list argument. Contrary to the request event the return values of the handlers are ignored as websocket communication is bidirectional
  - `after-message`: This event is triggered after the message event. It is provided more as an equivalent to the after-request event than out of necessity as there is no final response to inspect and handler can thus just as well be attached to the message event. For clear division of server logic, message specific handlers should be attached to the message event, whereas general handlers should, if possible, be attached to the after-message event. The after-message handlers will receive the app itself as the server argument, the client id as the id argument, a flag indicating whether the message is binary as the binary argument, the message itself as the message argument, and the request object used to establish the connection with the client as the request argument.
  - `send`: This event is triggered after a websocket message is send to a client. The handlers will receive the app itself as the server argument, the client id as the id argument and the send message as the message argument. Any return value is discarded.
  - `websocket-closed`: This event will be triggered every time a websocket connection is closed. The handlers will receive the app itself as the server argument, the client id as the id argument and request used to establish the closed connection as the request argument. Any return value is discarded.

+ Custom Events(自定义事件）

除了上述系统定义好的事件，你也可以自定义事件

```r
# Add a handler to the 'new-event' event
id <- app$on('new-event', function() {
message('Event fired')
})
# Trigger the event
app$trigger('new-event')
# Remove the handler
app$off(id)

```

3.Fire

定义一个新的app对象

```r
app <- Fire$new(host = ’127.0.0.1’, port = 8080L)

```

+ Methods
  - `ignite(block = TRUE, showcase = FALSE, ...)`: Begins the server, either blocking the console
if block = TRUE or not. If showcase = TRUE a browser window is opened directing at
the server address. ... will be redirected to the start handler(s)
  - `start(block = TRUE, showcase = FALSE, ...)`: A less dramatic synonym of for ignite()
reignite(block = TRUE, showcase = FALSE, ...) As ignite but additionally triggers the
resume event after the start event
  - `resume(block = TRUE, showcase = FALSE, ...)`: Another less dramatic synonym, this time
for reignite()
  - `extinguish()`: Stops a running server
  - `stop()`: Boring synonym for extinguish()
  - `is_running()`: Check if the server is currently running
  - `on(event, handler, pos = NULL)` <sup>重要</sup>:  Add a handler function to to an event at the given position(pos) in the handler stack. Returns a string uniquely identifying the handler. See the event documentation for more information.
  - `off(handlerId)`: Remove the handler tied to the given id
  - `trigger(event, ...)`: Triggers an event passing the additional arguments to the potential handlers
  - `send(message, id)`:  Sends a websocket message to the client with the given id, or to all connected clients if id is missing
  - `log(event, message, request, ...)`: Send a message to the logger. The event defines the
type of message you are passing on, while request is the related Request object if applicable.
  - `close_ws_con(id)`: Closes the websocket connection started from the client with the given id,
firing the websocket-closed event
  - `attach(plugin, ..., force = FALSE)`: Attaches a plugin to the server. See the plugin documentation for more information. Plugins can only get attached once unless force = TRUE
  - `has_plugin(name)`: Check whether a plugin with the given name has been attached
  - `header(name, value)`: Add a global header to the server that will be set on all responses. Remove by setting value = NULL
  - `set_data(name, value)`<sup>重要</sup>: Adds data to the servers internal data store
  - `get_data(name)`<sup>重要</sup>: Extracts data from the internal data store
  - `remove_data(name)`: Removes the data with the given name from the internal data store
Fire 9
  - `time(expr, then, after, loop = FALSE)`: Add a timed evaluation (expr) that will be evaluated
after the given number of seconds (after), potentially repeating if loop = TRUE. After
the expression has evaluated the then function will get called with the result of the expression
and the server object as arguments.
  - `remove_time(id)`: Removes the timed evaluation identified by the id (returned when adding the
evaluation)
  - `delay(expr, then)`<sup>重要</sup>: Similar to time(), except the expr is evaluated immediately at the end of the
loop cycle (see here for detailed explanation of delayed evaluation in fiery).
remove_delay(id) Removes the delayed evaluation identified by the id
  - `async(expr, then)` As delay() and time() except the expression is evaluated asynchronously.
The progress of evaluation is checked at the end of each loop cycle
  - `remove_async(id)`: Removes the async evaluation identified by the id. The evaluation is not necessarily
stopped but the then function will not get called.
  - `set_client_id_converter(converter`:) Sets the function that converts an HTTP request into a
specific client id
  - `set_logger(logger)`: Sets the function that takes care of logging
  - `set_client_id_converter(converter)`: Sets the function that converts an HTTP request into a
specific client id
  - `clone()`: Create a copy of the full Fire object and return that

4.loggers

app日志

```r
logger_null()
logger_console(format = "{time} - {event}: {message}")
logger_file(file, format = "{time} - {event}: {message}")
logger_switch(..., default = logger_null())
common_log_format
combined_log_format

app$set_logger(logger_file('fiery_log.log'))

app$set_logger(logger_console())

app$set_logger(logger_switch(warning =,error = logger_file('errors.log),
    default = logger_file('info.log')))
```

5.plugin

插件接口，使用`attach()`方法将插件添加到Fiery对象中，参数会随着`attach()`传递到插件的`on_attach()`方法。

```r
plugin <- list(
  on_attach = function(server) {
    router <- server$plugins$request_routr
    route <- Route$new()
    route$add_handler('all', '*', function(request, response, arg_list, ...) {
     message('Hello')
     TRUE
})
  router$add_route(route, 1)
},
  name = 'Hello_plugin',
  require = 'request_routr'
)

```

## Reference

[1]. <https://CRAN.R-project.org/package=fiery>

[2]. <https://github.com/thomasp85/fiery>
