Title: The blog is back!
Date: 2010-12-03 10:20
Modified: 2016-12-05 19:30
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

Apache would work just fine, but I appriciate the clean look of NGINX's config files and reactor model for servicing requests so I went with NGINX.  Lets install it and add a basic config that will serve your new static blog:

    :::bash
    apt-get install nginx
    mv /etc/nginx/sites-enabled/default ../sites-available
    echo "server {
	listen 80 default_server;
	listen [::]:80 default_server ipv6only=on;

	root /var/www/blog;
	index index.html index.htm;

	server_name localhost;

	location / {
		try_files $uri $uri/ =404;
	    }
    }" > /etc/nginx/sites-enabled/blog
    service nginx reload
    
Since we can now server content, lets generate some.

Generating content
------------------

Create a "hello world" post:

    :::bash
    echo "Title: My super title
    Date: 2010-12-03 10:20
    Modified: 2010-12-05 19:30
    Category: Python
    Tags: pelican, publishing
    Slug: my-super-post
    Authors: Alexis Metaireau, Conan Doyle
    Summary: Short version for index and feeds

    This is the content of my super blog post." > /opt/blog/site/content
    cd /opt/blog/site/
    pelican
    ln -s /opt/blog/site/output/ /var/www/blog
    
You can now navigate to your hosts IP address or and view your content!  This is great, but so far we have only met one requirement, and it was definitely the low hanging fruit.  So lets move on and automate all the things!

Installing a webhook handler:
--------------

To update the blog when changes are pushed to github, I chose to use the webhook functionality rather than polling.  Carlos Jenkins created a handy python program that triggers the execution of a script (in any language that supports a shebang).  The functionality is a fit, so I chose to implement it.  To start, download the software and install its dependencies:

    :::bash
    cd /opt/
    git clone https://github.com/carlos-jenkins/python-github-webhooks.git
    mv python-github-webhooks webhooks
    cd webhooks
    pip install -r requirements.txt
    
To recap, we just cloned the code and installed some Python modules using pip.  We still need to create a production worthy method of running the program though, so lets use WSGI with the NGINX server we setup prior.

Its important that our webhook always be running, and right now we are lacking a way to start our webhook manager through upstart.  To start, create a config file that uWSGI will read to know how many processes to spawn, and what kind of socket to create.  Lets call it webhooks.ini:

    :::bash
    [uwsgi]
    module = webhooks:application

    master = true
    processes = 5

    socket = webhooks.sock
    chmod-socket = 664
    vacuum = true

This configuration file is somewhat self explanatory, but note that the socket is a standard UNIX socket, not a TCP/IP socket.  The distinction is important, because later we will confiugre NGINX to serve requests to the socket created by uWSGI.  Next lets tie our uWSGI application into upstart so we can manage it like a normal Linux program by creating an upstart config called webhooks.conf in /etc/init:

    :::bash
    description "uWSGI instance to service webhooks"

    start on runlevel [2345]
    stop on runlevel [!2345]

    setuid webhooks
    setgid www-data

    script
        cd /opt/webhooks
        uwsgi --ini webhooks.ini
    end script
    
 Last but not least, lets create a webhooks user and give them ownership of the webhooks directory:
 
    :::bash
    useradd -r -s /bin/false webhooks
    usermod -a -G www-data webhooks
    chown -R webhooks:www-data /opt/webhooks
    
    
    usermod -a -G www-data webhooks
     
     
