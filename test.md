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

* an asterisk starts an unordered list
* and this is another item in the list
+ or you can also use the + character
- or the - character

* Markdown files uploaded to a github repository trigger content to be added to the site
* Server must not poll or periodically update the content, but only when new content is pushed
* The blog must be pretty and celean (duh! :P)
* Static content is a must, no full-fledged CMS

To display line numbers, use a path-less shebang instead of colons:

    #!python
    print("The path-less shebang syntax *will* show line numbers.")
