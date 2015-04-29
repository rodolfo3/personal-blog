---
layout: post
title:  "Python app on IBM Bluemix"
date:   2015-04-29 00:00:00 -3
categories: bluemix
---

[Bluemix] is the IBM cloud platform. It allows use [Cloud Foundry], Docker and OpenStack to deploy the applications.
To this, I'll use Cloud Foundry to deploy a Python (using Flask) app to test the service.

Cloud Foundry is an open source PaaS. It works more or less like [Heroku], but without locking in a provider.
In their website, they have tutorials showing how to [deploy] on Amazon AWS and RedHat OpenStack - Bluemix supports it out-of-the-box.

== Install the cf CLI tool ==

Cloud Foundry have a CLI tool written in ruby, but there is no Debian package for it.
I use the same approach used for [jekyll], using environment vars:

``` bash
$ export GEM_HOME="/opt/cf"
$ gem install cf
```

And a "cf" file in /usr/local/bin (with execution permission)

``` bash
#!/bin/sh
export GEM_HOME="/opt/cf"
/opt/cf/bin/cf $@
```

Then check if it works:

``` bash
$ cf --version
cf 5.4.7
```

Finally, configure the API endpoint and login:

``` bash
$ cf target https://api.ng.bluemix.net
$ cf login
```

== The app ==

As made on Heroku, the Python version is selected on `runtime.txt` file.
Python 2.7.9 is the default, but I prefer set if to avoid future problems with version updates.
There is some [versions][python-runtime] available: from 2.7.0 to 3.4.1.
The file only contains

```
python-2.7.9
```

To include the dependencies, there is the `requirements.txt` file:

```
Flask==0.10.1
```

The app should listen in a port defined by the server.
The port is in the `VCAP_APP_PORT` environment var.
The simplest flask app, starting the server in the right port, should be like:

``` python
import os
import flask

app = flask.Flask(__name__)

@app.route('/')
def hello():
    return 'Hello World!'

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=int(os.getenv('VCAP_APP_PORT', 5000))
```

To indicate to cf how to start the app, create a file called `Procfile`:

```
web: python hello.py
```

One more step, to configure the app, is create the `manifest.yml` file.
The file should indicate the right [buildpack].
This step is not mandatory, since the "push" command will make some questions interactivelly and create this file if it does not exists:

``` yaml
pplications:
- name: hello-python
  memory: 128M
  instances: 1
  host: hello-python
  domain: mybluemix.net
  path: "."
  buildpack: https://github.com/cloudfoundry/python-buildpack
```

To deploy, using the CLI:

``` bash
$ cf push
```

And that's it!

Using the IBM Bluemix Dashboard is possible add other domain names to the app, check the load, etc.
It can also add other resources, called "services" (like Heroku "addons").
Most of them are from IBM, but there is some third parts available too (like "mongolab").

The drawback using Python is the [auto-scaling] service: it only allows to scale based in memory measurements.

More tests are needed (this is really too basic to get any real-world information). Next steps include database and other URLs to access the data.


[cloudfoundry-install]: http://docs.cloudfoundry.org/devguide/installcf/install-go-cli.html
[deploy]: http://docs.cloudfoundry.org/deploying/
[python-runtime]: https://www.ng.bluemix.net/docs/#starters/python/index.html#pythonversions
[buildpack]: https://github.com/cloudfoundry/python-buildpack
[auto-scaling]: https://www.ng.bluemix.net/docs/#services/Auto-Scaling/index.html#autoscaling
