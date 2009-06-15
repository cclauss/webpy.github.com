---
layout: default
title: Webpy + Nginx with FastCGI
---

# Webpy + Nginx with FastCGI

This cookbook entry explains how to run web.py on Nginx with Fastcgi.

### Requirements

* Nginx 0.8.0 or 0.7.* (with fastcgi and rewrite module).
* Webpy 0.32
* Spawn-fcgi 1.6.2
* Flup

Older versions may work, but aren't tested.

### Resources

* [Nginx install guide.](http://wiki.nginx.org/NginxInstall)
* [Spawn-fcgi](http://redmine.lighttpd.net/projects/spawn-fcgi/news)
* [Flup](http://trac.saddi.com/flup)

### Notes

* You may replace <code>index.py</code> with your own file name.
* <code>/path/to/www</code> Is the path to the directory where your webpy application is located.
* <code>/path/to/www/index.py</code> is the full path to your python file.
* Do not run anything until you are at *Run*.

## Nginx configuration

	location / {
	    include fastcgi_params;
	    fastcgi_param SCRIPT_FILENAME $fastcgi_script_name;  # [1]
	    fastcgi_param PATH_INFO $fastcgi_script_name;        # [2]
	    fastcgi_pass 127.0.0.1:9002;
	}

To serve static files add this:

	location /static/ {
	    if (-f $request_filename) {
		rewrite ^/static/(.*)$  /static/$1 break;
	    }
	}

Note: the address and port may be different.

## Spawn-fcgi

You can start a process with:

	spawn-fcgi -d /path/to/www -f /path/to/www/index.py -a 127.0.0.1 -p 9002

### Start and shutdown script

Start:

	#!/bin/sh
	spawn-fcgi -d /path/to/www -f /path/to/www/index.py -a 127.0.0.1 -p 9002

Shutdown:

	#!/bin/sh
	kill `pgrep -f "python /path/to/www/index.py"`

Note: You're free to choose which address, port, directory and filename to use, but be sure to adjust the Nginx configuration.

## Hello world!

Save the following code in your www directory and call the file index.py (or whatever you like).
The following line is required: <code>web.wsgi.runwsgi = lambda func, addr=None: web.wsgi.runfcgi(func, addr)</code>.

	#!/usr/bin/env python
	# -*- coding: utf-8 -*-

	import web

	urls = ("/.*", "hello")
	app = web.application(urls, globals())

	class hello:
		def GET(self):
			return 'Hello, world!'

	if __name__ == "__main__":
		web.wsgi.runwsgi = lambda func, addr=None: web.wsgi.runfcgi(func, addr)
		app.run()

Note: make your file executable by doing <code>chmod +x index.py</code>. You'll get errors if it isn't executable.

## Run

1. Start a process with <code>spawn-fcgi</code>.
2. Start Nginx.

To check if it runs do <code>ps aux | grep index.py</code> or simply visit the page in your browser.

To reload your configuration:

	/path/to/nginx/sbin/nginx -s reload

And to stop:

	/path/to/nginx/sbin/nginx -s stop