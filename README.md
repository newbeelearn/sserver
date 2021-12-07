Self hosted netlify clone to serve your static content. **sserver** is simple headless server that can be used to host your static content.
It provides https including certificate renewals, synchronizes content from github repository all in a single binary.
Upcoming features include access control and stripe integration so you can use your favorite static site generator for your site/blog as well as selling your courses. 


# Table of Contents

1.  [Features](#orge20de41)
2.  [Usage](#orgfa94247)
    1.  [github repository without configuration file](#org90e7351)
    2.  [Folder in github repository without config file](#org2504298)
    3.  [Branch of github repository repository without config file (like gh-pages)](#orgd3f1131)
    4.  [github repository with domain name without config file](#org0e264b1)
    5.  [github repository with configuration file](#org7b0ff6a)
3.  [Operational details](#org8d0bb60)
    1.  [Description of files created by **sserver**](#orgbaa8d7e)
    2.  [Configuration](#org9a7de76)
    3.  [Default Values in absence of configuration file](#org4b112ca)
4.  [FAQ](#orgf9ca1c6)
    1.  [Is this opensource?](#org33a8a9c)
    2.  [Why was this created?](#orga61ec0e)
    3.  [What are the supported OS?](#orgc9b2fc4)
    4.  [Where can i request feature? suggestions for improvement?](#org38e4bd9)
    5.  [I provided serverpath in config file and left rest of the values blank why is it not working?](#org5bc7e4e)
    6.  [I have found that i can register and login users should i use them?](#org7670486)
    7.  [What does it costs?](#org9ca4f3a)
5.  [TODO](#org6ab43aa)


<a id="orge20de41"></a>

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


<a id="orgfa94247"></a>

# Usage

Here are some examples of how it can be used, please make sure to set GITHUB<sub>AUTH</sub> before trying them out e.g.

    export GITHUB_AUTH=<github_token>


<a id="org90e7351"></a>

## github repository without configuration file

    ./sserver -token "$GITHUB_AUTH" -repo "https://github.com/newbeelearn/sserver.git"  

Repository should have index.html in its root directory. If hugo/jekyll etc. like static site generator is used, repository should contain generated site.(It would have index.html in root by default)


<a id="org2504298"></a>

## Folder in github repository without config file

    ./sserver -token "$GITHUB_AUTH" -repo "https://github.com/newbeelearn/sserver.git?folder=public"  

Repository should have index.html in folder from where you want to serve the content, typically it is public if hugo/jekyll etc. static site generators are used


<a id="orgd3f1131"></a>

## Branch of github repository repository without config file (like gh-pages)

    ./sserver -token "$GITHUB_AUTH" -repo "https://github.com/newbeelearn/sserver.git?ref=gh-pages"  

Branch should have index.html in its root directory. If hugo/jekyll etc. like static site generator is used, branch should contain generated site.(It would have index.html in root by default)


<a id="org0e264b1"></a>

## github repository with domain name without config file

    ./sserver -token "$GITHUB_AUTH" -repo "https://github.com/newbeelearn/sserver.git?domain=example.com"  

Repository should have index.html in its root directory
Access to the domain from which site is served
**sserver** should have permissions to bind to 443 port, this can be done with following command

    sudo setcap 'cap_net_bind_service=+ep' sserver  


<a id="org7b0ff6a"></a>

## github repository with configuration file

    ./sserver -token "$GITHUB_AUTH" -repo "https://github.com/newbeelearn/sserver.git?folder=configs"

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


<a id="org8d0bb60"></a>

# Operational details


<a id="orgbaa8d7e"></a>

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


<a id="org9a7de76"></a>

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


<a id="org4b112ca"></a>

## Default Values in absence of configuration file

Here are the default values when ssconfig.toml is not provided.

    root = ""
    port = "1313"
    db = "users.db"
    # 12 hours
    syncinterval = "@every 12h"
    serverpath = "wwwss"


<a id="orgf9ca1c6"></a>

# FAQ


<a id="org33a8a9c"></a>

## Is this opensource?

No, only binaries are released and site is used for discussions around the product.


<a id="orga61ec0e"></a>

## Why was this created?

This was created because of the need to host courses

1.  Without giving up control by using online saas services for hosting course.
2.  Avoid using complicated opensource solutions that require constant maintenance.
3.  Use same site generated by static site generators for landing page/blog and protected course content.
4.  Something simple to avoid maintenance overhead.


<a id="orgc9b2fc4"></a>

## What are the supported OS?

linux and macos are supported out of the box. Windows users can use WSL however it is not tested.


<a id="org38e4bd9"></a>

## Where can i request feature? suggestions for improvement?

Create issue and tag it with feature


<a id="org5bc7e4e"></a>

## I provided serverpath in config file and left rest of the values blank why is it not working?

In case ssconfig.toml is provided, please provide all the fields. Default values are used only when ssconfig.toml
is not provided


<a id="org7670486"></a>

## I have found that i can register and login users should i use them?

It is in pre alpha stage and i am gathering requirements from users so there are no compatibility guarantees.
You can give it a try however i would suggest to wait for sometime before using these features in production.
Let us know what feature you are interested in discussions


<a id="org9ca4f3a"></a>

## What does it costs?

This is not yet decided as of now it is free to use. 
Paid product if available will use separate channel.
so if you are downloading from github release it is free forever.
Help us in deciding it, tell us what you would pay for it in discussion board.


<a id="org6ab43aa"></a>

# TODO

-   [ ] Add sample scripts for running in aws/gcp/azure/alibabacloud etc.
-   [ ] Add sample site to demonstrate server functionality
-   [ ] Add performance data

