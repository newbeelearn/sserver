Self hosted netlify clone to serve your static content. **sserver** is simple headless server that can be used to host your static content.
It provides https including certificate renewals, synchronizes content from github repository all in a single binary.
Upcoming features include access control and stripe integration so you can use your favorite static site generator for your site/blog as well as selling your courses. 


# Table of Contents

1.  [Features](#org7f6c44b)
2.  [Usage](#orgcef91cf)
    1.  [github repository without configuration file](#org0a4a158)
    2.  [Folder in github repository without config file](#orgfd183fa)
    3.  [Branch of github repository repository without config file (like gh-pages)](#orge988b74)
    4.  [github repository with domain name without config file](#orgbc4dadd)
    5.  [github repository with configuration file](#orgc1ff168)
3.  [Operational details](#org3f61757)
    1.  [Description of files created by **sserver**](#org368408a)
    2.  [Configuration](#org5ea7bc0)
    3.  [Default Values in absence of configuration file](#org29801eb)
4.  [FAQ](#orgb35371a)
    1.  [Is this opensource?](#orge5443d4)
    2.  [Why was this created?](#org80e90a5)
    3.  [What are the supported OS?](#org913867a)
    4.  [Where can i request feature? suggestions for improvement?](#org7fc3cc4)
    5.  [I provided serverpath in config file and left rest of the values blank why is it not working?](#org8254e21)
    6.  [I have found that i can register and login users should i use them?](#org28f44f4)
    7.  [What does it costs?](#org56ea680)
5.  [TODO](#org5451c0a)


<a id="org7f6c44b"></a>

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


<a id="orgcef91cf"></a>

# Usage

Here are some examples of how it can be used, please make sure to set GITHUB<sub>AUTH</sub> before trying them out e.g.

    export GITHUB_AUTH=<github_token>


<a id="org0a4a158"></a>

## github repository without configuration file

    ./sserver -token "$GITHUB_AUTH" -repo "https://github.com/newbeelearn/sserver.git"  

Repository should have index.html in its root directory. If hugo/jekyll etc. like static site generator is used, repository should contain generated site.(It would have index.html in root by default)


<a id="orgfd183fa"></a>

## Folder in github repository without config file

    ./sserver -token "$GITHUB_AUTH" -repo "https://github.com/newbeelearn/sserver.git?folder=public"  

Repository should have index.html in folder from where you want to serve the content, typically it is public if hugo/jekyll etc. static site generators are used


<a id="orge988b74"></a>

## Branch of github repository repository without config file (like gh-pages)

    ./sserver -token "$GITHUB_AUTH" -repo "https://github.com/newbeelearn/sserver.git?ref=gh-pages"  

Branch should have index.html in its root directory. If hugo/jekyll etc. like static site generator is used, branch should contain generated site.(It would have index.html in root by default)


<a id="orgbc4dadd"></a>

## github repository with domain name without config file

    ./sserver -token "$GITHUB_AUTH" -repo "https://github.com/newbeelearn/sserver.git?domain=example.com"  

Repository should have index.html in its root directory
Access to the domain from which site is served
**sserver** should have permissions to bind to 443 port, this can be done with following command

    sudo setcap 'cap_net_bind_service=+ep' sserver  


<a id="orgc1ff168"></a>

## github repository with configuration file

    ./sserver -token "$GITHUB_AUTH" -repo "https://github.com/newbeelearn/sserver.git?ref=test-config"

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


<a id="org3f61757"></a>

# Operational details


<a id="org368408a"></a>

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


<a id="org5ea7bc0"></a>

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


<a id="org29801eb"></a>

## Default Values in absence of configuration file

Here are the default values when ssconfig.toml is not provided.

    root = ""
    port = "1313"
    db = "users.db"
    # 12 hours
    syncinterval = "@every 12h"
    serverpath = "wwwss"


<a id="orgb35371a"></a>

# FAQ


<a id="orge5443d4"></a>

## Is this opensource?

No, only binaries are released and site is used for discussions around the product.


<a id="org80e90a5"></a>

## Why was this created?

This was created because of the need to host courses

1.  Without giving up control by using online saas services for hosting course.
2.  Avoid using complicated opensource solutions that require constant maintenance.
3.  Use same site generated by static site generators for landing page/blog and protected course content.
4.  Something simple to avoid maintenance overhead.


<a id="org913867a"></a>

## What are the supported OS?

linux and macos are supported out of the box. Windows users can use WSL however it is not tested.


<a id="org7fc3cc4"></a>

## Where can i request feature? suggestions for improvement?

Create issue and tag it with feature


<a id="org8254e21"></a>

## I provided serverpath in config file and left rest of the values blank why is it not working?

In case ssconfig.toml is provided, please provide all the fields. Default values are used only when ssconfig.toml
is not provided


<a id="org28f44f4"></a>

## I have found that i can register and login users should i use them?

It is in pre alpha stage and i am gathering requirements from users so there are no compatibility guarantees.
You can give it a try however i would suggest to wait for sometime before using these features in production.
Let us know what feature you are interested in discussions


<a id="org56ea680"></a>

## What does it costs?

This is not yet decided as of now it is free to use. 
Paid product if available will use separate channel.
so if you are downloading from github release it is free forever.
Help us in deciding it, tell us what you would pay for it in discussion board.


<a id="org5451c0a"></a>

# TODO

-   [ ] Add sample scripts for running in aws/gcp/azure/alibabacloud etc.
-   [ ] Add sample site to demonstrate server functionality
-   [ ] Add performance data

