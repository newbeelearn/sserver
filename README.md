Self hosted netlify clone to serve your static content. **sserver** is simple headless server that can be used to host your static content.
It provides https including certificate renewals, synchronizes content from github repository all in a single binary.
Upcoming features include access control and stripe integration so you can use your favorite static site generator for your site/blog as well as selling your courses. 


# Table of Contents

1.  [Features](#org1ce4ac5)
2.  [Usage](#org6f278e6)
    1.  [github repository without configuration file](#org6b55533)
    2.  [Folder in github repository without config file](#org663f0de)
    3.  [Branch of github repository repository without config file (like gh-pages)](#org07dec39)
    4.  [github repository with domain name without config file](#orgebccff0)
    5.  [github repository with configuration file](#orgaf9d415)
3.  [Operational details](#org1055ae6)
    1.  [Description of files created by **sserver**](#org5e9e5c6)
    2.  [Configuration](#org218f464)
    3.  [Default Values in absence of configuration file](#org4a08ff1)
4.  [FAQ](#orgc19a98d)
    1.  [Is this opensource?](#org0276cf2)
    2.  [Why was this created?](#orga8c149c)
    3.  [What are the supported OS?](#org93dc109)
    4.  [Where can i request feature? suggestions for improvement?](#org7058acf)
    5.  [I provided serverpath in config file and left rest of the values blank why is it not working?](#org433f03a)
    6.  [I have found that i can register and login users should i use them?](#orgc0ce8fb)
    7.  [What does it costs?](#org6e21814)
5.  [TODO](#orgbf446af)


<a id="org1ce4ac5"></a>

# Features

-   [X] Serve static site
-   [X] Auto https support via letsencrypt
-   [X] Sync site from github repository
-   [ ] Add api to register new users
-   [ ] Add api to authenticate users
-   [ ] Add support for specifying course details
-   [ ] Add api to authorize users to download course content
-   [ ] Add admin api
-   [ ] Add stripe integration for buying course


<a id="org6f278e6"></a>

# Usage

Here are some examples of how it can be used


<a id="org6b55533"></a>

## github repository without configuration file

    ./sserver -token "<github_token>" -repo "https://github.com/newbeelearn/sserver.git"  

Repository should have index.html in its root directory. If hugo/jekyll etc. like static site generator is used, repository should contain generated site.(It would have index.html in root by default)


<a id="org663f0de"></a>

## Folder in github repository without config file

    ./sserver -token "<github_token>" -repo "https://github.com/newbeelearn/sserver.git?folder=public"  

Repository should have index.html in folder from where you want to serve the content, typically it is public if hugo/jekyll etc. static site generators are used


<a id="org07dec39"></a>

## Branch of github repository repository without config file (like gh-pages)

    ./sserver -token "<github_token>" -repo "https://github.com/newbeelearn/sserver.git?ref=gh-pages"  

Branch should have index.html in its root directory. If hugo/jekyll etc. like static site generator is used, branch should contain generated site.(It would have index.html in root by default)


<a id="orgebccff0"></a>

## github repository with domain name without config file

    ./sserver -token "<github_token>" -repo "https://github.com/newbeelearn/sserver.git?domain=example.com"  

Repository should have index.html in its root directory
Access to the domain from which site is served
**sserver** should have permissions to bind to 443 port, this can be done with following command

    sudo setcap 'cap_net_bind_service=+ep' sserver  


<a id="orgaf9d415"></a>

## github repository with configuration file

    ./sserver -token "<github_token>" -repo "https://github.com/newbeelearn/sserver.git?folder=configs"

Repository should contain ssconfig.toml configuration file in its root directory
Sample ssconfig.toml can be found below

    #specify the site
    [site]
    #for local sites that
    root = ""
    #port where you want to serve the site
    port = "1313"
    #sqlite db file for users/auth permissions and course pricing
    db = "users.db"
    #github repository details of the form github.com/owner/repo/ref/folder
    service = ["github.com", "newbeelearn", "sserver", "gh-pages", ""]
    #domain of the site
    domain = "example.com"
    #period to check for new content
    syncinterval = "@every 12h"
    #path where all the sserver files will be created
    serverpath = "wwwss"  


<a id="org1055ae6"></a>

# Operational details


<a id="org5e9e5c6"></a>

## Description of files created by **sserver**

**sserver** creates "wwwss" directory from where it is run unless something else is provided in ssconfig.toml

    drwxrwxr-x 2 test test   4096 Nov 30 17:04 a
    drwxrwxr-x 8 test test   4096 Nov 30 17:04 b
    drwxrwxr-x 2 test test   4096 Nov 30 17:04 certs
    drwxrwxr-x 2 test test   4096 Nov 30 17:04 logs
    -rw-rw-r-- 1 test test 527483 Nov 30 17:04 tmp.zip
    -rw-rw-r-- 1 test test  49152 Nov 30 17:05 users.db

-   If https is used key and certificates can be found in certs
-   Server access logs are inside logs folder
-   Database of users/permissions and course details are in sqlite file with ending ".db"(users.db in this example)
-   tmp.zip is temporary site downloaded from github and is overwritten on every sync
-   a/b folders is from where site is served. Actual folder keeps on alternating between the two.


<a id="org218f464"></a>

## Configuration

Note when ssconfig.toml is provided all parameters must be specified as all default values are set to "" .

Sample ssconfig.toml file

    #specify the site
    [site]
    #for local sites that
    root = ""
    #port where you want to serve the site
    port = "1313"
    #sqlite db file for users/auth permissions and course pricing
    db = "users.db"
    #github repository details of the form github.com/owner/repo/ref/folder
    service = ["github.com", "newbeelearn", "sserver", "gh-pages", ""]
    #domain of the site
    domain = "example.com"
    #period to check for new content
    syncinterval = "@every 12h"
    #path where all the sserver files will be created
    serverpath = "wwwss"  


<a id="org4a08ff1"></a>

## Default Values in absence of configuration file

Here are the default values when ssconfig.toml is not provided.

    root = ""
    port = "1313"
    db = "users.db"
    # 12 hours
    syncinterval = "@every 12h"
    serverpath = "wwwss"


<a id="orgc19a98d"></a>

# FAQ


<a id="org0276cf2"></a>

## Is this opensource?

No, only binaries are released and site is used for discussions around the product.


<a id="orga8c149c"></a>

## Why was this created?

This was created because of the need to host courses

1.  Without giving up control by using online saas services for hosting course.
2.  Avoid using complicated opensource solutions that require constant maintenance.
3.  Use same site generated by static site generators for landing page/blog and protected course content.
4.  Something simple to avoid maintenance overhead.


<a id="org93dc109"></a>

## What are the supported OS?

linux and macos are supported out of the box. Windows users can use WSL however it is not tested.


<a id="org7058acf"></a>

## Where can i request feature? suggestions for improvement?

Create issue and tag it with feature


<a id="org433f03a"></a>

## I provided serverpath in config file and left rest of the values blank why is it not working?

In case ssconfig.toml is provided, please provide all the fields. Default values are used only when ssconfig.toml
is not provided


<a id="orgc0ce8fb"></a>

## I have found that i can register and login users should i use them?

It is in pre alpha stage and i am gathering requirements from users so there are no compatibility guarantees.
You can give it a try however i would suggest to wait for sometime before using these features in production.
Let us know what feature you are interested in discussions


<a id="org6e21814"></a>

## What does it costs?

This is not yet decided as of now it is free to use. 
Paid product if available will use separate channel.
so if you are downloading from github release it is free forever.
Help us in deciding it, tell us what you would pay for it in discussion board.


<a id="orgbf446af"></a>

# TODO

-   [ ] Add sample scripts for running in aws/gcp/azure/alibabacloud etc.
-   [ ] Add sample site to demonstrate server functionality
-   [ ] Add performance data

