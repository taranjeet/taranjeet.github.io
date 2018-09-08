---
layout: post
title:  "Google App Engine and Python app"
date:   2015-10-10 22:42:56
categories: google-app-engine hosting gae python-app
---


This post is about hosting a Python app on Google App Engine. Google App engine is a PaaS(Platform as a Service) like Heroku, OpenShift. Why I choose Google App Engine, this is one of the most interesting question. 

For the present case my requirement was to provide a url, from where I can get the contents. To make it more precise, that content was being parsed from some website. Now comes the tricky part. I need to update that content at a regular interval. Lets say daily. 

So Why GAE. Why not Heroku or Openshift. Heroku provides a very nice support for Celery. Celery can be used with Rabbit mq but for configuring rabbitmq on Heroku, you need a Credit card so that you can verify your authenticity. It is free though.

So I switched to Google App Engine. One of the issues which I was faced was my current app was on Django 1.8 but GAE provides support upto 1.5. I dont know what their "latest" means. But then I figured out that parising logic was not at all related to being a Django app. So I created a Python app. I followed this [tutorial](https://cloud.google.com/appengine/docs/python/gettingstartedpython27/introduction)

This tutorial creates a webapp2 app. One can easily follow this tutorial. My requirement was to do a specific task at a particular url. So i wrote this

```
class MyClass(webapp2.RequestHandler):

	def get(self):

		self.response.headers["Content-Type"] = 'text/plain'
		self.response.write(("Hello World"))

app = webapp2.WSGIApplication([
			('/',MyClass),
	],debug=True)

```

The tutorial is very simple and easy to follow. One can understand more easily if his background is Django based. I will cover some of the tricky part here.

First of all my use case was to parse a webpage, so i needed a module which can read the html of the page. Initially I was using `requests` but it did not worked out. So I used `urllib3`. The url that I was trying to parse was `https` based.

Then the second thing which tricked me was how to install a custom python package in my app. It is very easy. Just create a folder named `lib` and in your main file write the following.

```
import sys

sys.path.insert(0, './lib')

```

One thing that should be taken care of is now you need to copy the folder of the installed package into the lib folder.

The next tricky thing was how to store data. The data can be crawled only once a day, and hence it need to be available in a database or a file. Creating a file is not allowed on Google App Engine. You can only read a bundled file but cannot create a new file. So Now database. Google Cloud SQL is free for 2 months with 300$ after which the charge is on usage basis. So I used GAW NDB. You can read about Google NDb from [here](https://cloud.google.com/appengine/docs/python/ndb/). 

I am writing down what was required by me.

```

class MyModel(ndb.Model):
	data = ndb.JsonProperty()
	dtime = ndb.DateTimeProperty(auto_now_add=True)

	@classmethod
	def latest_data(cls):
		# to get the latest entry
		qry = cls.query().order(-cls.dtime).fetch()[0]
		return qry


# to save it, type
m = MyModel(data={})
m.put()

# to get the latest query, write
f = MyModel.query.latest_data().data
```

One more thing which was required by me for urllib3 was the following snipped of code.

```

from urllib3 import PoolManager
from urllib3.contrib.appengine import AppEngineManager, is_appengine_sandbox

from google.appengine.ext import ndb

if is_appengine_sandbox():
	http = AppEngineManager()
else:
	http = PoolManager()

```

This all was done in a single file named `myproject.py`. To use Google App Engine, you need to download the sdk and then run `dev_appserver.py`. This should be run at one level above your root directory of project so that the name of folder is available.

```

dev_appserver.py myproject

```

Its time to deploy your application. Create a application on your console and copy its id. Now run

```

appcfg.py -A <id of project from google console> update <name of folder>

```

Every python app on google app engine is accompanied by an `app.yaml` file which is very easy and simple to create. The basic helloworld tutorial can be used for reference here.

For creating crons, one need to create a `cron.yaml` file. The contents of the file can be

```

cron:
- description: daily crawl
  url: /myurhere
  schedule: every 24 hours
  timezone: Asia/Kolkata

```

and run 

```
appcfg.py -A <id of project from google console> update_cron <name of folder>

```

Reference :
[urllib3](http://urllib3.readthedocs.org/en/latest/contrib.html)
