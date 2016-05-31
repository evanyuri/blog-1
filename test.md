Title: The blog is back!
Date: 2010-12-03 10:20
Modified: 2010-12-05 19:30
Category: Python
Tags: pelican, publishing
Slug: my-super-post
Authors: William Michael Turner
Summary: Create a webhook driven blog using Pelican and a little bit of scripting

Its been way too long since I've updated my blog, so I decided the old iteration should be sent out to pasture.  My old blog was updated manually by uploading markdown to a VPS, which while functional, lacked a certain ease of use and appeal that I have come to expect using services like Github and Jenkins.  With that in mind I'll show you how you can create a blog with the same requirements I had in mind when making this one:

Requirements:

* Markdown files uploaded to a github repository trigger content to be added to the site
* Server must not poll or periodically update the content, but only when new content is pushed
* The blog must be pretty and celean (duh! :P)
* Static content is a must, no full-fledged CMS

With all that in mind, it seems a solution could be found by combining github's webhook feature with a static content generator (Pelican).  Before we do the cool stuff, lets come up with some let's get a basic Pelican blog set up.

Installing Pelican
------------------
I chose to use Ubuntu 14.04 because it (sadly) was the most up-to-date OS my VPS provider had in their catalogue.  It should be trivial to port these commands to CentOS or Fedora (s/apt-get/yum/).

    :::bash
    apt-get install python-pip python-dev
    pip install pelican virtualenv Markdown typography
    
Once the above commands have been run, from a software standpoint, Pelican has been installed.  This doesn't make it useful yet though, so lets configure a pelican site (lets just assume you're site is called "blog"):

    :::bash
    mkdir -p /opt/blog
    virtualenv /opt/blog
    mkdir /opt/blog/site
    cd /opt/blog/site
    pelican-quickstart
    
To recap, we installed Pelican globally, but setup a virtual environment in which the blog will be configured.  If you are hosting multiple blogs on the same host, there are semantics to be debated, but unless you are planning on modifying the pelican package (i'm not!) there's not much point installing it in each virtualenv you create.  The 'pelican-quickstart' command will ask you a series of questions and then generate a skeleton blog. Most of the options are in regards to the location the static content will be published, and in my case I keep it on the VPS host, so most of my answers were "n".  

We need to generate some content and test the site out, but we are SOL without a webserver, so lets...

Install NGINX
-------------

