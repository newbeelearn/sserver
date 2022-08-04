Serve your site and sell courses with self hosted server.

**sserver** is simple headless server for hosting courses and associated blog/static content from private github repository with minimal overhead.

It provides https out of the box so you don't have to deal with installing/managing certificates.
It syncs the content automatically from github so you don't have to upload your content to server. It also supports premium content with simple configuration file without affecting your content workflow for e.g. if you are using static site generator like hugo you can hide the premium content from public by specifying it in config file. It has stripe integration so you can sell your premium content. It has user management built in for adding/authenticating users.
It also has admin api so you can monitor orders/users of your site.


# Table of Contents

1.  [Features](#org32277aa)
2.  [Usage](#org69a7823)
    1.  [github repository](#org2a30f11)
    2.  [Folder in github repository with config file](#orgcb6b2c0)
    3.  [Branch of github repository repository with config file (like gh-pages)](#org7545ff7)
    4.  [github repository with domain name](#orgba95c80)
    5.  [local files](#org0bb8ce2)
3.  [Operational details](#org97e80a4)
    1.  [Description of files created by **sserver**](#org8904abd)
    2.  [Configuration](#orgca529a8)
    3.  [Default Values](#org1002ae7)
4.  [API](#orge3ff3df)
    1.  [API Endpoints](#org3abf525)
5.  [FAQ](#orgb7e0eea)
    1.  [Is this opensource?](#org235a80d)
    2.  [What is the current status?](#org16054e1)
    3.  [Why was this created?](#org2c40628)
    4.  [Can i use it for saas it already has user mangement and billing?](#org4b48f88)
    5.  [What are the use cases?](#org3708e0b)
    6.  [What are the supported OS?](#orgd608561)
    7.  [Where can i request feature? suggestions for improvement?](#orgba20720)
    8.  [What does it costs?](#orgeb601ff)
6.  [TODO](#orgf59662a)


<a id="org32277aa"></a>

# Features

-   [X] Serve static content from github/local directory
-   [X] Auto https support via letsencrypt
-   [X] Auto sync content from github repository
-   [X] Support for gated content/course
-   [X] Stripe integration for selling course
-   [X] User registration and authentication
-   [X] Admin apis to get overall view of the store


<a id="org69a7823"></a>

# Usage

Here are some examples of how it can be used, please make sure to set SS\_<> environment variables before trying them out e.g.

    export SS_GITHUB_TOKEN=<github_token>
    export SS_STRIPE_TOKEN=<stripe_token>
    export SS_SMTP_FROM=<email_id>
    export SS_SMTP_USER=<smtp_username>
    export SS_SMTP_PWD=<smtp_password>
    export SS_SMTP_HOST=<smtp_host_address>
    export SS_SMTP_PORT=<smtp_port>
    export SS_ADMIN_EMAIL=<admin_email>
    export SS_ADMIN_PWD=<admin_password_for_sserver>


<a id="org2a30f11"></a>

## github repository

    ./sserver -repo "https://github.com/newbeelearn/sserver.git"

Repository should have index.html and ssconfig.toml in its root directory. If hugo/jekyll etc. like static site generator is used, repository should contain generated site.(It would have index.html in root by default)


<a id="orgcb6b2c0"></a>

## Folder in github repository with config file

    ./sserver -repo "https://github.com/newbeelearn/sserver.git?folder=public"  

Repository should have index.html in folder from where you want to serve the content, typically it is public if hugo/jekyll etc. static site generators are used. It should have ssconfig.toml in the root directory


<a id="org7545ff7"></a>

## Branch of github repository repository with config file (like gh-pages)

    ./sserver -repo "https://github.com/newbeelearn/sserver.git?ref=test-config"  

Branch should have index.html and ssconfig.toml in its root directory. If hugo/jekyll etc. like static site generator is used, branch should contain generated site.(It would have index.html in root by default)


<a id="orgba95c80"></a>

## github repository with domain name

    ./sserver -repo "https://github.com/newbeelearn/sserver.git?domain=example.com"  

Repository should have index.html and ssconfig.toml in its root directory
Access to the domain from which site is served
**sserver** should have permissions to bind to 443 port, this can be done with following command

    sudo setcap 'cap_net_bind_service=+ep' sserver  


<a id="org0bb8ce2"></a>

## local files

    ./sserver -repo "file:///workspace/projects/newbeelearn.com/sserver"

Repository should have index.html and ssconfig.toml in its root directory.
All the options i.e. folder/domain etc. can be specified in case of local files as well

Sample ssconfig.toml can be found below

    # specify the site
    [site]
    # period to check for new content
    syncinterval = "@every 12h"
    # product/course details
    [[site.prod]]
    name = "course1"
    # path from root, this will be accessible to users who have bought the course
    path = "courses/course1"
    # can be draft/active, buying functionality will be enabled when status is active
    status = "active"
    # unique identifier for the course
    sku = "prod-course-1"
    # price in cents
    price = 10000
    # currency
    currency = "USD"


<a id="org97e80a4"></a>

# Operational details


<a id="org8904abd"></a>

## Description of files created by **sserver**

**sserver** creates "wwwss" directory from where it is run

    drwxrwxr-x 2 test test   4096 Nov 30 17:04 a
    drwxrwxr-x 8 test test   4096 Nov 30 17:04 b
    drwxrwxr-x 2 test test   4096 Nov 30 17:04 certs
    drwxrwxr-x 2 test test   4096 Nov 30 17:04 logs
    -rw-rw-r-- 1 test test 527483 Nov 30 17:04 tmp.zip
    -rw-rw-r-- 1 test test  49152 Nov 30 17:05 ssapp.db

-   If https is used key and certificates can be found in certs
-   Server access logs are inside logs folder
-   Database of users/permissions and course details are in sqlite file ssapp.db
-   tmp.zip is temporary site downloaded from github in zip format and is overwritten on every sync
-   a/b folders is from where site is served. Actual folder keeps on alternating between the two.


<a id="orgca529a8"></a>

## Configuration

Configuration file is used to specify the products/courses that you want to sell
as well as some server parameters like how often the site should be synced etc.


### Server parameters are

-   **syncinterval** specifies auto syncing period for content default is 12 hours
-   **login** specifies login/sign-in page to be redirected to if user tries to access protected content if not specified it redirects
    to home page of the site
-   **postlogin** specifies page user should be redirected to in case of successful login if not specified it returns json response
    which needs to be processed from client side
-   **postsignup** specifies page user should be redirected to after completing the signup if not specified it returns json response
    which needs to be interpreted on client side


### Product/Course parameters are

-   **name** is the name of the product course
-   **path** is the path of course content relative to the root directory. This will not be accessible without
    registering and buying the course.
-   **status** can be either draft or active. Buying functionality becomes available only when product status is active
-   **sku** this should be unique identifier for the course/product. It creates stripe product with the identifier specified in
    sku. If you already have a product in stripe use its id as sku otherwise duplicate products will be created.
-   **price** is in cents for USD, i.e. 10.58USD course should set the price as 1058. For other currencies please check stripe documentation
    as the software currently assumes stripe convention for currency and price is followed.
-   **currency** should follow stripe convention


### Sample config file

Sample ssconfig.toml file is shown below

    #specify the site
    [site]
    #period to check for new content default is 12 hours
    syncinterval = "@every 12h"
    [[site.prod]]
    name = "course1"
    path = "courses/course1"
    status = "active"
    sku = "prod-course-1"
    price = 10000
    currency = "USD"


<a id="org1002ae7"></a>

## Default Values

-   If domain is not specified content will be served from port 54545 else port 443
-   If local file is specified content will be served from that directory else
    it would be served from wwwss folder which will autosync content from github repository
-   If syncinterval is specified it will sync content from github/check content from local file
    with syncinterval duration else it will sync/check for new content every 12 hours
-   Syncing between stripe events i.e. refund done through stripe site etc. are synced every
    6 hours
-   If SMTP details are not specified, mail sending functionality on registration/reset etc. will be disabled


<a id="orge3ff3df"></a>

# API

All api endpoints are relative to the domain used for serving the content
i.e. if domain is example.com and api is `/api/v1/product/list` request would be <https://example.com/api/v1/product/list>

Role hierarchy is like this admin > user > guest
Any api accessible to guest is also accessible to user and any api accessible to user is also accessible to admin
For accessing user/admin api's session cookie obtained after logging in must be supplied with each request
in practice this will be taken care by the browser


<a id="org3abf525"></a>

## API Endpoints

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">Description</th>
<th scope="col" class="org-left">Request</th>
<th scope="col" class="org-left">Role</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left"><a href="#orgd881a1a">Register User</a></td>
<td class="org-left">POST <code>/api/v1/user/register</code></td>
<td class="org-left">guest</td>
</tr>


<tr>
<td class="org-left"><a href="#orgca35ad3">Login User</a></td>
<td class="org-left">POST <code>/api/v1/user/login</code></td>
<td class="org-left">guest</td>
</tr>


<tr>
<td class="org-left"><a href="#org3ceeb49">Logout user</a></td>
<td class="org-left">GET  <code>/api/v1/user/logout</code></td>
<td class="org-left">guest</td>
</tr>


<tr>
<td class="org-left"><a href="#orgafed125">Verify user</a></td>
<td class="org-left">GET  <code>/api/v1/user/verify/:id</code></td>
<td class="org-left">guest</td>
</tr>


<tr>
<td class="org-left"><a href="#org7b461f8">Reset user password</a></td>
<td class="org-left">POST <code>/api/v1/user/reset</code></td>
<td class="org-left">guest</td>
</tr>


<tr>
<td class="org-left"><a href="#orgf25920d">Get product list</a></td>
<td class="org-left">GET  <code>/api/v1/product/list</code></td>
<td class="org-left">guest</td>
</tr>


<tr>
<td class="org-left"><a href="#orge4913f9">Create order</a></td>
<td class="org-left">POST <code>/api/v1/order/id</code></td>
<td class="org-left">guest</td>
</tr>


<tr>
<td class="org-left"><a href="#orge5f9c48">Modify order</a></td>
<td class="org-left">PUT  <code>/api/v1/order/id</code></td>
<td class="org-left">user</td>
</tr>


<tr>
<td class="org-left"><a href="#orgaeb3081">Checkout order</a></td>
<td class="org-left">POST <code>/api/v1/order/checkout</code></td>
<td class="org-left">user</td>
</tr>


<tr>
<td class="org-left"><a href="#org2075173">Get order by order id</a></td>
<td class="org-left">GET  <code>/api/v1/order/id/:id</code></td>
<td class="org-left">user</td>
</tr>


<tr>
<td class="org-left"><a href="#org0a08798">Get all orders by user</a></td>
<td class="org-left">GET  <code>/api/v1/order/id/list</code></td>
<td class="org-left">user</td>
</tr>


<tr>
<td class="org-left"><a href="#orgfce6df7">Change user password</a></td>
<td class="org-left">POST <code>/api/v1/user/changepwd</code></td>
<td class="org-left">user</td>
</tr>


<tr>
<td class="org-left"><a href="#org4f62fd2">Get all orders</a></td>
<td class="org-left">GET  <code>/api/v1/order/list</code></td>
<td class="org-left">admin</td>
</tr>


<tr>
<td class="org-left"><a href="#orgc594f6a">Get all users</a></td>
<td class="org-left">GET  <code>/api/v1/user/list</code></td>
<td class="org-left">admin</td>
</tr>
</tbody>
</table>


<a id="orgd881a1a"></a>

### Register user

Register new user with email and password. Sends mail to the email used to register with verification
code if SMTP server configuration is set.

-   Example Request
    
        curl 'http://localhost:54545/api/v1/user/register' \
        -H 'Content-Type: application/x-www-form-urlencoded' \
        -X POST \
        --data-raw 'email=stripe%40newbeelearn.com&password=test&confirm-password=test&remember=on'

-   Example Response
    
        {"status":"success"}


<a id="orgca35ad3"></a>

### Login user

Login user with email and password. Returns json if "postlogin" field is not set in config file
otherwise redirects to the page specified in "postlogin"

-   Example Request
    
        curl 'http://localhost:54545/api/v1/user/login' \
        -H 'Content-Type: application/x-www-form-urlencoded' \
        -X POST \
        --data-raw 'email=admin%40example.com&password=admin'

-   Example Response
    
        {
          "data": {
            "user_id": "1",
            "username": ""
          },
          "msg": "user found",
          "status": "success"
        }


<a id="org3ceeb49"></a>

### Logout user

Logs out logged in user and redirects to homepage

-   Example Request
    
        curl 'http://localhost:54545/api/v1/user/logout' \
        -H 'Cookie: session_id=9e8b22a3-15ac-442f-bf65-15c37dbfc889; max-age=300; path=/; secure; SameSite=Lax'

-   Example Response  
    
        <!doctype html>
        <html lang="en">
          <head>
          </head>
          <body>
          </body>
        </html>


<a id="orgafed125"></a>

### Verify user

Verifies the email of registered user by sending url if domain is set or code that should be appended after the verify api
if domain is not set

-   Example Request
    
        curl 'http://localhost:54545/api/v1/user/verify/cafj5grn0gpog1j3a0m0'

-   Example Response
    
        {"status":"success"}


<a id="org7b461f8"></a>

### Reset user password

Resets the user password and sends the new temporary code for login. User needs to use this code
on next login and change the password

-   Example Request
    
        curl 'http://localhost:54545/api/v1/user/reset' \
        -H 'Content-Type: application/x-www-form-urlencoded' \
        -X POST \
        --data-raw 'email=stripe%40newbeelearn.com'

-   Example Response
    
        {
        "data": null,
        "msg": "password reset successful",
        "status": "success"
        }


<a id="orgf25920d"></a>

### Get product list

Get all the products listed for sale on website. Takes "limit" and "offset" as queries.
If query is not set default values of limit is set to 10 and offset to 0

-   Example Request
    
        curl 'http://localhost:54545/api/v1/product/list?limit=1&offset=0'

-   Example Response
    
        {
        "data": [
          {
            "prd_id": 1,
            "prd_name": "course1",
            "sku": "prod-course-1",
            "permalink": "users/list/",
            "price": 10000,
            "currency": "USD",
            "period": 365,
            "status": "active"
          }
        ],
        "msg": "Order found",
        "status": "success"
        }


<a id="orge4913f9"></a>

### Create order

Creates new order with products listed in `"line_item"` field. Request should be valid json.
Actual `order_id/user_id` etc. fields are populated by the server any dummy value can be passed.

-   Example Request
    
        curl 'http://localhost:54545/api/v1/order/id' \
        -H 'Content-Type: application/json; charset=utf-8' \
        -H 'Cookie: session_id=cad8439e-dcc4-475e-94fc-12b75f85bb20; max-age=300; path=/; secure; SameSite=Lax' \
        -X POST \
        --data-raw '  {
          "order_id": 1,
          "user_id": 3,
          "currency": "USD",
          "line_items": [
            {
              "sku": "prod-course-1"
            },
            {
              "sku": "prod-course-3"
            }
          ]
        }'

-   Example Response
    
        {
        "data": {
          "order_id": 1,
          "user_id": 3,
          "created_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
          "modified_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
          "status": "active",
          "currency": "USD",
          "order_number": "cafjktbn0gpp5hq3dt4g",
          "grand_total": 11000,
          "line_items": [
            {
              "line_id": 1,
              "order_id": 1,
              "prd_id": 1,
              "created_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
              "modified_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
              "grand": 10000,
              "enabled": true,
              "sku": "prod-course-1"
            },
            {
              "line_id": 2,
              "order_id": 1,
              "prd_id": 2,
              "created_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
              "modified_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
              "grand": 1000,
              "enabled": true,
              "sku": "prod-course-3"
            }
          ]
        },
        "msg": "Order found",
        "status": "success"
        }


<a id="orge5f9c48"></a>

### Modify order

Modifies existing order by adding/deleting products in `line_item` field. User must be logged in to
modify the order and order should be in active state. Use the previous response of create order or get order
to add/delete products

-   Example Request
    
        curl 'http://localhost:54545/api/v1/order/id' \
        -H 'Content-Type: application/json; charset=utf-8' \
        -H 'Cookie: session_id=cad8439e-dcc4-475e-94fc-12b75f85bb20; max-age=300; path=/; secure; SameSite=Lax' \
        -X PUT \
        --data-raw '  {
            "order_id": 1,
            "user_id": 3,
            "created_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
            "modified_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
            "status": "active",
            "currency": "USD",
            "order_number": "cafjktbn0gpp5hq3dt4g",
            "grand_total": 11000,
            "line_items": [
        	 {
        	"line_id": 2,
        	"order_id": 1,
        	"prd_id": 2,
        	"created_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
        	"modified_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
        	"grand": 1000,
        	"enabled": true,
        	"sku": "prod-course-3"
              }
            ]
          }'

-   Example Response
    
        {
        "data": {
          "order_id": 1,
          "user_id": 3,
          "created_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
          "modified_at": "2022-06-07 11:48:05.425488765 +0000 UTC",
          "status": "active",
          "currency": "USD",
          "order_number": "cafjktbn0gpp5hq3dt4g",
          "grand_total": 1000,
          "line_items": [
            {
              "line_id": 2,
              "order_id": 1,
              "prd_id": 2,
              "created_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
              "modified_at": "2022-06-07 11:48:05.425488765 +0000 UTC",
              "grand": 1000,
              "enabled": true,
              "sku": "prod-course-3"
            }
          ]
        },
        "msg": "Order found",
        "status": "success"
        }


<a id="orgaeb3081"></a>

### Checkout order

Gets stripe url for payment of order created by create order api.
Use the response from create order/modify order/get order api to send the request.
Do not modify the response in this request it will result in failure.

-   Example Request
    
        curl 'http://localhost:54545/api/v1/order/checkout' \
        -H 'Content-Type: application/json; charset=utf-8' \
        -H 'Cookie: session_id=2f1be070-7256-4e84-a4ef-c14754cabcdb; max-age=300; path=/; secure; SameSite=Lax' \
        -X POST \
        --data-raw ' {
        	"order_id": 1,
        	"user_id": 3,
        	"created_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
        	"modified_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
        	"status": "active",
        	"currency": "USD",
        	"order_number": "cafjktbn0gpp5hq3dt4g",
        	"grand_total": 11000,
        	"line_items": [
        	     {
        	    "line_id": 2,
        	    "order_id": 1,
        	    "prd_id": 2,
        	    "created_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
        	    "modified_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
        	    "grand": 1000,
        	    "enabled": true,
        	    "sku": "prod-course-3"
        	  }
        	]
              }'

-   Example Response
    
        {
        "data": {
          "url": "https://checkout.stripe.com/pay/cs_test_a17D2l74NsKMv29YJ1c5rSBPx7BGSsNAsObGAsOanEJqyFNXKEYDLji4BZ#fidkdWxOYHwnPyd1blpxYHZxWjA0TlVKPHNMaW9vYEd1YmhdUWQ3UUJqSEpMYTMza11ObGAyXDFPcXA8bz1yY1VicVZVdDN8c1NkaUZEazxIQWdjM04wdz1DTmF3PXxHaVE9bTVuZz1pUWw3NTUybHZLZldgaicpJ2N3amhWYHdzYHcnP3F3cGApJ2lkfGpwcVF8dWAnPyd2bGtiaWBabHFgaCcpJ2BrZGdpYFVpZGZgbWppYWB3dic%2FcXdwYHgl"
        },
        "msg": "Order found",
        "status": "success"
        }


<a id="org2075173"></a>

### Get order by order number

Get order details by order number

-   Example Request
    
        curl 'http://localhost:54545/api/v1/order/id/cafjktbn0gpp5hq3dt4g' \
        -H 'Content-Type: application/json; charset=utf-8' \
        -H 'Cookie: session_id=fe9b9fff-c5c0-4745-becf-ecb0e5abca81; max-age=300; path=/; secure; SameSite=Lax'

-   Example Response
    
         {
         "data": {
           "order_id": 1,
           "user_id": 3,
           "created_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
           "modified_at": "2022-06-07 11:48:05.425488765 +0000 UTC",
           "status": "active",
           "currency": "USD",
           "order_number": "cafjktbn0gpp5hq3dt4g",
           "grand_total": 1000,
           "line_items": [
             {
               "line_id": 2,
               "order_id": 1,
               "prd_id": 2,
               "created_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
               "modified_at": "2022-06-07 11:48:05.425488765 +0000 UTC",
               "grand": 1000,
               "enabled": true
             }
           ]
         },
         "msg": "Order found",
         "status": "success"
        }


<a id="org0a08798"></a>

### Get all orders by user

Get all orders by user. Takes limit and offset as query parameters default values are 10 and 0 respectively

-   Example Request
    
        curl 'http://localhost:54545/api/v1/order/id/list?limit=1&offset=0' \
        -H 'Content-Type: application/json; charset=utf-8' \
        -H 'Cookie: session_id=fe9b9fff-c5c0-4745-becf-ecb0e5abca81; max-age=300; path=/; secure; SameSite=Lax'

-   Example Response
    
         {
         "data": [
           {
             "order_id": 1,
             "user_id": 3,
             "created_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
             "modified_at": "2022-06-07 11:48:05.425488765 +0000 UTC",
             "status": "active",
             "currency": "USD",
             "order_number": "cafjktbn0gpp5hq3dt4g",
             "grand_total": 1000
           }
         ],
         "msg": "Order found",
         "status": "success"
        }


<a id="orgfce6df7"></a>

### Change user password

Change user password. User must be logged in to make this request

-   Example Request
    
        curl 'http://localhost:54545/api/v1/user/changepwd' \
        -H 'Content-Type: application/x-www-form-urlencoded' \
        -H 'Cookie: session_id=fe9b9fff-c5c0-4745-becf-ecb0e5abca81; max-age=300; path=/; secure; SameSite=Lax' \
        -X POST \
        --data-raw 'oldpassword=cafjjjbn0gpp5hq3dt40&password=test123'

-   Example Response
    
        {
        "data": null,
        "msg": "password change successful",
        "status": "success"
        }


<a id="org4f62fd2"></a>

### Get all orders

Get all orders created in the store. Takes limit and offset as query parameters default values are 10 and 0 respectively
Available to admin users only.
Get all orders

-   Example Request
    
        curl 'http://localhost:54545/api/v1/order/list?limit=1&offset=0' \
        -H 'Content-Type: application/json; charset=utf-8' \
        -H 'Cookie: session_id=e4ecd3a4-b8be-493e-a33d-518ab11c65e8; max-age=300; path=/; secure; SameSite=Lax'

-   Example Response
    
        {
        "data": [
          {
            "order_id": 1,
            "user_id": 3,
            "created_at": "2022-06-07 11:45:57.601996759 +0000 UTC",
            "modified_at": "2022-06-07 11:48:05.425488765 +0000 UTC",
            "status": "active",
            "currency": "USD",
            "order_number": "cafjktbn0gpp5hq3dt4g",
            "price_total": 1000,
            "discount_total": 0,
            "sub_total": 1000,
            "taxes_total": 0,
            "grand_total": 1000,
            "refunds_total": 0,
            "created_channel": "",
            "payment_provider": ""
          }
        ],
        "msg": "Order found",
        "status": "success"
        }


<a id="orgc594f6a"></a>

### Get all users

Get all users registered on the website. Takes limit and offset as query parameters default values are 10 and 0 respectively
Available to admin users only.

-   Example Request
    
        curl 'http://localhost:54545/api/v1/user/list?limit=1&offset=0' \
        -H 'Content-Type: application/json; charset=utf-8' \
        -H 'Cookie: session_id=e4ecd3a4-b8be-493e-a33d-518ab11c65e8; max-age=300; path=/; secure; SameSite=Lax'

-   Example Response
    
        {
        "data": [
          {
            "user_id": 1,
            "email": "admin@example.com",
            "created_at": "2022-06-07 10:53:00.480128762 +0000 UTC",
            "username": "",
            "last_password_set": "2022-06-07 10:53:00.480128762 +0000 UTC",
            "last_login": "2022-06-07 10:53:00.480128762 +0000 UTC",
            "verified": 0,
            "reset": 0,
            "user_role": "admin"
          },
          {
            "user_id": 2,
            "email": "guest@example.com",
            "created_at": "2022-06-07 10:53:00.532691788 +0000 UTC",
            "username": "",
            "last_password_set": "2022-06-07 10:53:00.532691788 +0000 UTC",
            "last_login": "2022-06-07 10:53:00.532691788 +0000 UTC",
            "verified": 0,
            "reset": 0,
            "user_role": "guest"
          },
          {
            "user_id": 3,
            "email": "stripe@newbeelearn.com",
            "created_at": "2022-06-07 11:13:06.947313364 +0000 UTC",
            "username": "",
            "last_password_set": "2022-06-07 11:13:06.947313364 +0000 UTC",
            "last_login": "2022-06-07 11:13:06.947313364 +0000 UTC",
            "verified": 2,
            "reset": 1,
            "user_role": "user"
          }
        ],
        "msg": "Order found",
        "status": "success"
        }


<a id="orgb7e0eea"></a>

# FAQ


<a id="org235a80d"></a>

## Is this opensource?

No, only binaries are released and site is used for discussions around the product.


<a id="org16054e1"></a>

## What is the current status?

It's in alpha stage functionality is complete however it may contain bugs


<a id="org2c40628"></a>

## Why was this created?

This was created because of the need to host courses

1.  Without giving up control by using online saas services for hosting course.
2.  Avoid using complicated solutions that require constant maintenance.
3.  Use same site generated by static site generators for landing page/blog and protected course content.
4.  Something simple to avoid maintenance overhead.


<a id="org4b48f88"></a>

## Can i use it for saas it already has user mangement and billing?

Not right now because subscription is not supported it is for one time digital products only.
It also doesn't have route forwarding functionality where your own saas can be plugged in.
However these changes can be added if there is sufficient interest
plase start discussion if you would like to have these features as of now it is not on roadmap.


<a id="org3708e0b"></a>

## What are the use cases?

It can be used for hosting course and associated blogs. Blogs with newsletter. Blogs with premium content.
Landing page of startup and associated blog. Selling themes etc.


<a id="orgd608561"></a>

## What are the supported OS?

linux and macos are supported out of the box. Windows users can use WSL however it is not tested.


<a id="orgba20720"></a>

## Where can i request feature? suggestions for improvement?

Create issue and tag it with feature


<a id="orgeb601ff"></a>

## What does it costs?

This is not yet decided as of now it is free to use. 
Paid product if available will use separate channel.
so if you are downloading from github release it is free forever.
Help us in deciding it, tell us what you would pay for it in discussion board.


<a id="orgf59662a"></a>

# TODO

-   [ ] Add sample scripts for running in aws/gcp/azure/alibabacloud etc.
-   [ ] Add sample site to demonstrate server functionality
-   [ ] Add performance data
-   [ ] Add automatic backups
-   [ ] Add support for subscription in case there is sufficient interest from community

