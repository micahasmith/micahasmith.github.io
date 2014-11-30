---
title: Node.js RabbitMQ Queue Cleanup Script
layout: post
excerpt: A script to clean up dead rmq queues on a RabbitMQ server
---

# Node.js RabbitMQ Queue Cleanup Script

In my original rmq-based architecture at printio I never did queue auto-delete; all the queues stay alive even after they're needed.

The problem with this is that some of the queues are still getting fed by exchanges. Now we have dead queues with waiting messages.

Super simple to script a delete for queues with more than x messages:

{% highlight javascript %}
var req = require('request');

var user = "username";
var pass = "password";

var baseUrl = "http://"+user+":"+pass+"@rabbitmq-server.com:15672/api/";
var queues = "queues?columns=name,messages,messages_ready";

req.get({
	url:baseUrl+queues,
	json:true,
},function(err,res, body){
	body.forEach(function(queue){
		if(queue && queue.name && queue.messages_ready && queue.messages_ready > 240){
			var url = baseUrl+"queues/%2F/"+queue.name;
			req({
				url: url,
				method:"DELETE"
			},function(errr,ress,body){
				//console.log(errr,ress);
				console.log(url,body);
			});
		}
	});
});

{% endhighlight %}


Dead simple.