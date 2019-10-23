---
title: cheerio简单网页爬虫应用
date: 2016-03-24 12:18:49
tags:
- nodejs
- cheerio
- shadowsocks
---

### cheerio

 **cheerio** 是一个为服务器特别定制的，快速、灵活、封装jQuery核心功能工具包。Cheerio包括了 jQuery核心的子集，从jQuery库中去除了所有DOM不一致性和浏览器不兼容的部分，揭示了它真正优雅的API。Cheerio工作在一个非常简单，一致的DOM模型之上，解析、操作、渲染都变得难以置信的高效。基础的端到端的基准测试显示Cheerio大约比JSDOM快八倍(8x)。Cheerio封装了@FB55兼容的htmlparser，几乎能够解析任何的 HTML 和 XML document。

<!--more-->

### shadowsocks

 **shadowsocks** 一个轻量级科学上网利器，中文名为“影梭”，是使用Python等语言开发的、基于Apache许可证开源的代理软件。Shadowsocks使用socks5代理，用于保护网络流量。在中国大陆被广泛用于突破防火长城（GFW），以浏览被封锁的内容。目前开发者已宣布停止维护。但仍有更新陆续推送。

当然我个人使用一般也之是用用Google，看看Youtube，了解下墙外的世界。
* windows客户端下载：[shadowsocks-windows](https://github.com/shadowsocks/shadowsocks-windows/releases/download/3.0/Shadowsocks-3.0.zip)
* 服务商 [ishadowsocks](http://www.ishadowsocks.net/)，首页也会有免费的账号，每6小时更新一次，我要做的就是抓取页面里的账号信息，修改shadowsocks的json配置文件。

项目源码：<https://git.coding.net/opso/mysock.git>

### cheerioo模块安装

```bash
$ npm install cheerio
```
### 使用http模块
使用node内置模块http请求页面信息
`html-curl.js`

```js
var http = require("http");

function download(url,callback){
  http.get(url,function(res){
    var data = "";
    res.on('data',function(chunk){
      data += chunk;
    });
    res.on('end',function(){
     callback(data);
    });
  }).on('error',function(){
    callback(null);
  });	
}

exports.download = download;
```

### 页面抓取
加载cheerio抓取账号保存到文件
`index.js`

```js
var cheerio = require("cheerio");
var fs = require("fs");
var server = require("./html-curl");
var filepath = './gui-config.json';
var url = "http://www.ishadowsocks.net";
var config = {
"configs" : [
  {
"server" : "US1.ISS.TF",
"server_port" : 443,
"password" : "",
"method" : "aes-256-cfb",
"remarks" : ""}
,
  {
"server" : "HK2.ISS.TF",
"server_port" : 8989,
"password" : "",
"method" : "aes-256-cfb",
"remarks" : ""}
,
  {
"server" : "JP3.ISS.TF",
"server_port" : 443,
"password" : "",
"method" : "aes-256-cfb",
"remarks" : ""}

],
"strategy" : null,
"index" : 0,
"global" : false,
"enabled" : true,
"shareOverLan" : false,
"isDefault" : false,
"localPort" : 1080,
"pacUrl" : null,
"useOnlinePac" : false,
"availabilityStatistics" : false};

 
server.download(url, function(data) {
  if (data) {
    //console.log(data);
  
    var $ = cheerio.load(data);
    var arr = new Array();
    $(".text-center h4").each(function(i, e) {
        var info = $(e).text();
        if(info.indexOf(":")>0){
            //console.log(info);
            var tmp = info.split(":");
            //console.log(tmp);
            arr.push(tmp[1]);
        }
    });
  
    //console.log(arr);
    
    var configs = config['configs'];
    //console.log(configs);
    var arr2 = [];
    for(var i=0;i<arr.length; i+=5){  
       arr2.push(arr.slice(i,i+5));
    }
    //console.log(arr2);
    for(var e = 0;e <configs.length; e++){  
        console.log(arr2[e][2]);
        configs[e].server = arr2[e][0].toUpperCase();
        configs[e].server_port = parseInt(arr2[e][1]);
        configs[e].password = arr2[e][2];
    }
    config['configs'] = configs;
    fs.writeFile(filepath, JSON.stringify(config, null, 4), function(err) {
    if(err) {
      console.log(err);
    } else {
      console.log("JSON saved to " + filepath);
    }
});
    // console.log(config);    
  } else {
      console.log("error");
  } 
});
```

操作Dom出乎意料的方便有没有！