---
layout: post
title: "Rweixin and fiery"
img: lm.jpg 
date: 2017-12-16 9:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [R,CRAN,Package]
---

**徐静**

最近第十届R语言会议（上海)刚刚结束，会议中郎大为的报告让我印象深刻，他用了Fiery包和自己开发的Rweixin实现了一个个人公众号的视觉问答系统，让我眼前一亮，我最近也在用Python的itchat模块加爬虫为业务部门做事情，所以对此比较感兴趣，用周末时间研究了一下这两个包，做个笔记，以免遗忘。

## Fiery包

是一个可以构建各种符合http要求的服务器的R包，类似的包有

+ shiny: 依赖于httpuv,一个websocks的服务

+ openCPU: 模型线上滑的部署可以用到，部分服务器的形式比较复杂

+ plumbr: 也可以构建服务器

官方样例：

```r
library(fiery)

# Create a New App
app <- Fire$new(host='127.0.0.1',port = 8080L)
#logging
app$set_logger(logger_console())

# Setup the data every time it starts
app$on('start', function(server, ...) {
  server$set_data('visits', 0)
  server$set_data('cycles', 0)
})

# Count the number of cycles (internal loops)
app$on('cycle-start', function(server, ...) {
  server$set_data('cycles', server$get_data('cycles') + 1)
  server$log('info', paste0('request from ', id, ' received'), request)
  
})

# Count the number of requests
app$on('before-request', function(server, ...) {
  server$set_data('visits', server$get_data('visits') + 1)
})

# Handle requests
app$on('request', function(server, request, ...) {
  response <- request$respond()
  response$status <- 200L
  response$body <- paste0('<h1>This is indeed a test. You are number ',
         server$get_data('visits'), '</h1>')
  response$type <- 'html'
})

# Show number of requests in the console
app$on('after-request', function(server, ...) {
  message(server$get_data('visits'))
  flush.console()
})

# Terminate the server after 50 cycles
app$on('cycle-end', function(server, ...) {
  if (server$get_data('cycles') > 50) {
    message('Ending...')
    flush.console()
    server$extinguish()
  }
})

# Be polite
app$on('end', function(server) {
  message('Goodbye')
  flush.console()

})

app$ignite(showcase = TRUE)
#> Fire started at 127.0.0.1:8080
#> 1
#> Ending...
#> Goodbye



```


## Rweixin

一个用于微信的用户管理素材管理的R包，是由李舰和郎大为开发的包，目前托管在Github上，我们还是深挖包的源码，看看都有那些好玩的功能

身份验证：

![id验证](({{ site.url }}/assets/bowen20/output1.png))

微信服务器传递的消息：

```
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[fromUser]]></FromUserName>
<CreateTime>1348831860</CreateTime>
<MsgType><![CDATA[text]]></MsgType>
<Content><![CDATA[this is a test]]></Content>
<MsgId>1234567890123456</MsgId>
</xml>

```

反馈的消息：

```
output = sprintf("<xml>
<ToUserName><![CDATA[%s]]></ToUserName>
<FromUserName><![CDATA[%s]]></FromUserName>
<CreateTime>%s</CreateTime>
<MsgType><![CDATA[text]]></MsgType>
<Content><![CDATA[介尼玛似一个%s]]></Content>
</xml>",user,ori,as.numeric(Sys.time()),showPic(filename))

```

feiry代码结构：

```r
app <- Fire$new()
app$on('request', function(server, request, ...) {
#1. 验证身份
#2. 微信服务器传递信息
#3. 内部处理（存储，反馈）
#4. 回复相应信息
})
app$on('end', function(server) {
flush.console()
})
app$ignite(showcase = TRUE)


```

Rweixin函数

```
library(Rweixin)
AppID = '[**你的APPID**]'
AppSecret = '[**你的APPKEY**]'
registerAccounts("xiangmax", AppID, AppSecret)
w1 <- createWeixin("xiangmax", ssl.verifypeer = F)
u1 <- getUsers(w1, ssl.verifypeer = F)  #获取关注列表
head(u1)

#其他函数

##getMaterialList 获取素材列表
##getMaterialNum 获取素材数量
##uploadImage 上传图片
##uploadNews 上传图文
##sendNews 群发信息
```

## feiry + Rweixin 打造视觉问答系统

首先使用别人训练好的Google Inception CNN model.

api.R

```r
library(fiery)

source('config.ini')
source("utils.R")
# Create a New App
app <- Fire$new()
app$host = IP
app$port = port


app$on('start', function(server, ...) {
  server$set_data('visits', 0)
  server$set_data('cycles', 0)
})

# Count the number of cycles (internal loops)
app$on('cycle-start', function(server, ...) {
  server$set_data('cycles', server$get_data('cycles') + 1)
})

# Count the number of requests
app$on('before-request', function(server, ...) {
  server$set_data('visits', server$get_data('visits') + 1)
})

# Handle requests
app$on('request', function(server, request, ...) {
  # request$set_body(request)
  # servera<<-server
  # aaa<<-request
  
  request$parse_raw()
  message(request$as_message())
  print(rawToChar(request$body))

  msgXML = rawToChar(request$body)
  ori = extractWeixin(msgXML, "ToUserName")
  user = extractWeixin(msgXML, "FromUserName")
  time = as.POSIXct(as.numeric(extractWeixin(msgXML, "CreateTime",F)),
                    origin = "1970-01-01")
  msgType = extractWeixin(msgXML, "MsgType")
  content = extractWeixin(msgXML, "Content")
  messageId =  extractWeixin(msgXML, "MsgId",F)
  PicUrl = extractWeixin(msgXML, "PicUrl")
  print(PicUrl)
  # output = sprintf('"%s","%s","%s","%s","%s"\n',user,time,msgType,content,messageId)
  # cat(output)
  #cat(rawToChar(request$body))
  #cat('\n')
  # print(names(request))
  response <- request$respond()
  response$status <- 200L
  
  
  theQuery = request$query
  # theQ <<- theQuery
  if('echostr' %in% names(theQuery)){
    
    response$body = regmatches(request$querystring,
                               gregexpr("(?<=echostr=).+(?=&t)",
                                        request$querystring,
                                        perl = TRUE))
  }else{
    print(123)
    response$body = returnMsg(ori,user, time, msgType, content, messageId, PicUrl)
  }
  # cat(response$body)
  # response$body <- 'success'
  response$type <- 'html'
})

# Show number of requests in the console
app$on('after-request', function(server, ...) {
  #message(server$get_data('visits'))
  flush.console()
})

# Terminate the server after 50 cycles
# app$on('cycle-end', function(server, ...) {
#   if (server$get_data('cycles') > 500) {
#     message('Ending...')
#     flush.console()
#     server$extinguish()
#   }
# })

# Be polite
app$on('end', function(server) {
  # sink()
  #message('Goodbye')
  flush.console()
})

# sink('data.txt', append = T)
app$ignite(showcase = TRUE)
#> Fire started at 127.0.0.1:8080
#> 1
#> Ending...
#> Goodbye

```


utils.R

```r
# 微信数据截取函数
extractWeixin = function(msgXML, pattern,CDATA = T){
  if(CDATA)
    regPattern = paste0("(?<=", pattern, "><\\!\\[CDATA\\[)[\\s\\S]+(?=\\]\\]></",
                        pattern, ")")
  else
    regPattern = paste0("(?<=", pattern, ">).+(?=</",
                        pattern, ")")
  regmatches(msgXML,
             gregexpr(regPattern,
                      msgXML,
                      perl = TRUE))[[1]]
}


library(mxnet)
library(imager)
# 载入模型
model = mx.model.load("Inception/Inception_BN", iteration=39)
# 载入mean image
mean.img = as.array(mx.nd.load("Inception/mean_224.nd")[["mean_img"]])

im <- load.image(system.file("extdata/parrots.png", package = "imager"))
plot(im)


preproc.image <- function(im, mean.image) {
  # crop the image
  shape <- dim(im)
  short.edge <- min(shape[1:2])
  xx <- floor((shape[1] - short.edge) / 2)
  yy <- floor((shape[2] - short.edge) / 2)
  cropped <- crop.borders(im, xx, yy)
  # resize to 224 x 224, needed by input of the model.
  resized <- resize(cropped, 224, 224)
  # convert to array (x, y, channel)
  arr <- as.array(resized) * 255
  dim(arr) <- c(224, 224, 3)
  # subtract the mean
  normed <- arr - mean.img
  # Reshape to format needed by mxnet (width, height, channel, num)
  dim(normed) <- c(224, 224, 3, 1)
  return(normed)
}

normed <- preproc.image(im, mean.img)

prob <- predict(model, X=normed)
max.idx <- max.col(t(prob))
synsets <- readLines("Inception/synset.txt")
print(paste0("Predicted Top-class: ", synsets[[max.idx]]))

showPic = function(input){
  cat <- load.image(input)
  # plot(cat)
  normed <- preproc.image(cat, mean.img)
  prob <- predict(model, X=normed)
  max.idx <- max.col(t(prob))
  print(paste0("Predicted Top-class: ", synsets[[max.idx]]))
  output = strsplit(synsets[[max.idx]]," ")[[1]]
  output[1] = ''
  return(paste(output,collapse=" "))
}



returnMsg = function(ori,user, time, msgType, content, messageId,PicUrl){
  if(length(PicUrl)==0){
    cat(234)
    return('success')
  }
  cat(123)
  filename = paste0("data/",format(Sys.time(),"%Y%m%d%M"))
  download.file(PicUrl,destfile = filename)
  
  output = sprintf("<xml>
    
    <ToUserName><![CDATA[%s]]></ToUserName>
    
    <FromUserName><![CDATA[%s]]></FromUserName>
    
    <CreateTime>%s</CreateTime>
    
    <MsgType><![CDATA[text]]></MsgType>
    
    <Content><![CDATA[介尼玛似一个%s]]></Content>
    
    </xml>",user,ori,as.numeric(Sys.time()),showPic(filename))
  return(output)
  
}
```

## 最后附上fiery包的原文档及郎大为的演讲slides

可以直接 [下载 fiery的PDF]({{ site.url }}/assets/bowen20/fiery.pdf)

可以直接[下载 郎大为的slides]({{ site.url }}/assets/bowen20/slide_langdawei.pdf)

声明：郎大为slides来源于统计之都共享第十届R语言资料



