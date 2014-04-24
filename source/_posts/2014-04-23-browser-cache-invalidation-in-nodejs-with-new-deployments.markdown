---
layout: post
title: "Browser cache invalidation in NodeJs with new deployments"
date: 2014-04-23 16:31
comments: true
categories: 
- Node.js
- http
---


Caching in Web browsers is important. Without caching page performance is hurt as each page navigation requires the same set of JavaScripts and CSS files to be re-fetched. 
<!--more-->
We can achieve caching by setting Cache-Control header's max-age in the HTTP response headers. In NodeJS/Express stack, you do this done by passing options map to express.static() middle-ware. Here is an example:

```javascript
app.configure('production', function () {
    ...
    app.use(express.static(__dirname + '/public',
    		{ maxAge: 864000000}  // 10 days!
    ));
    ...
});
```
However, the challenge is to discard or bust this client cache for new code deployments. As HTTP is a stateless protocol, there is no way server can tell clients to ignore old caches and request again. If you do set max-age:0 then we shall not arrive in this situation at all. However, we want to achieve caching and refresh this cache with new code deployments. As HTTP is a stateless protocol, one simple solution is to generate a new URL for the resource. What if we append app version as query string or folder path? 
i.e.
```javascript
/js/main.js?v-1.0 or /js/v-1.0/main.js. 
```
Browsers will treat this as new URL and fetch again. NPM has a plugin `validator`, that exactly does this. The problem with npm:validator is - it requires to increment app version number for each new deployment or your sprint. If you forget to increment version no., you leave your clients buzzing with missing CSS classes. We can achieve this without introducing a new dependency. Let me simplify this and achieve the same using timestamps. Remember, less code keeps footprint limited! Solution In the following code we are trying to achieve a good caching period (30 days). Also this caching should not be applicable for your local development. We will be using app's start time as a substitute of version. (We have been using AWS-EBS for production deployments and each new deployment re-starts the app, giving new time-stamp with each deployment.)

```javascript
//All jade's static resources should have: ?v=#{deployVersion}
app.locals.deployVersion = (new Date).getTime();

app.configure('production', function () {
    ...
    app.use(express.static(path.join(__dirname, 'public'),
    		{ maxAge: 864000000 }
    ));
    ...
});
app.configure('development', function () {
    ...
    app.use(express.static(path.join(__dirname, 'public'),
    		{ maxAge: 0 }
    ));

    app.locals.pretty = true; // why not prettify while coding :)
    ...
});

//We need to suffix all resource path with ?v=#{deployVersion} in all jade files. 
//(Note: App's locals are available across jade.)

link(rel='stylesheet', href='/css/index.css?v=#{deployVersion}')
script(type='text/javascript', src='/js/index.js?v=#{deployVersion}')
script(type='text/javascript', src='/js/bootstrap.min.js?v=#{deployVersion}')
This renders all your resource URLs as follows forcing browser to request new resource.
link(rel='stylesheet', href='/css/index.css?v=1386176288126')
script(type='text/javascript', src='/js/index.js?v=1386176288126')
script(type='text/javascript', src='/js/bootstrap.min.js?v=1386176288126')
```
#What if you have distributed deployment? 
If you planning to deploy your app on AWS Elastic Beanstalk having auto scale (more than one app server), than above shall not work efficiently as each client browser may hit to new app server seeing a different time-stamp! Let's solve this by clustering all app starts in a 5 min span.
```javascript
app.locals.deployVersion = 
           Math.ceil((new Date).getTime()/300000)*300000;
```
#More ideas:
	
1.	Serve files with an MD5 hash suffix. Try: Connect-Assets
2.	Using GIT log as deployVersion. Integrate GIT api in project and read last commit ID
3.	Incrementing app version number in each sprint :down:

Author: <a href="https://twitter.com/@geekpack">@geekpack</a> 
