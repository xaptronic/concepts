---
layout: post
title: HAProxy and nginx + port_in_redirect
---

Part of my VPS setup includes HAProxy and nginx. HAProxy is what all http (port 80) requests go through to reach a particular server tech. Because I'm running various php, nodejs, ruby, python and perl apps I wanted a way to have all these things transparently accessible via port 80 based on various criteria, most of which have something to do with the domain name. HAProxy has so far been great for this.

While I was setting up and porting this blog from Wordpress (_pukes_) to Jekyll (_pretty cool_) I noticed that nginx would sometimes rewrite directory paths to include a trailing slash. Fine, however it kept putting its port into the redirect url so you would wind up hitting the server directly instead of going through HAProxy. Not all services that are running are directly accessible without going through HAProxy so an answer needs to be found.

nginx has a lot of configuration options and is very flexible. The two options that are key in solving these kinds of problems are server_name_in_redirect and port_in_redirect. If you are running a real load balancing scenario and have www.site.com that balances to one.site.com and two.site.com, you may find yourself redirected to one.site.com in the browser. Setting server_name_in_redirect to off will instruct nginx to use what is in the Host header instead of the server_name for that block. If you are doing what I am doing and using HAProxy as a way to route traffic to different technologies on different ports, then you might find yourself being bounced to site.com:8081/some/cool/stuff. Setting port_in_redirect off will prevent nginx from injecting its server port.

Example server config:

{% highlight bash %}
server {
	listen *:8080;
	server_name codesauce.com;
	root /srv/codesauce/web/codesauce.com;

	server_name_in_redirect off;
	port_in_redirect off;

	location / {
		index index.php;
		if (!-f $request_filename){
				set $rule_1 1$rule_1;
		}
		if (!-d $request_filename){
				set $rule_1 2$rule_1;
		}
		if ($rule_1 = "21"){
				rewrite /. /index.php last;
		}
	}

	location ~ \.php {
		fastcgi_pass 127.0.0.1:9000;
		fastcgi_index index.php;
		include /usr/local/nginx/conf/fastcgi_params;
	}
}
{% endhighlight %}

Great¡!!
