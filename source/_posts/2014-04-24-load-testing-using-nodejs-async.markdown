---
layout: post
title: "Load Testing Using NodeJs Async"
date: 2014-04-24 16:27
comments: true
author: Ankit Jain
categories: 
 - Nodejs
 - http
 - load-testing
---

NodeJS is a platform for building applications using JavaScript. JavaScript is single threaded and has callback architecture. How about using this for load testing a Web server? Often we are limited by the tools and their feature set. If you are familiar with NodeJs, here is my quick solution for customized load testing. 
<!--more-->

### Background ###

[Async](https://github.com/caolan/async) is one of the most popular node module for code organization in JavaScript otherwise you end up in callback hell. Let's create a nodejs project and add async as one of the dependencies.

```
<projectDir>
  |
  +-- package.json
  +-- app.js
```
  
package.json: 
```
{
  "name": "CrowdChat",
  "version": "0.0.1",
  "private": true,
  "dependencies": {
    "async": "0.2.9"
  }
}
```

### Making HTTP requests in NodeJS ###

Let's make our HTTP request:

```
function getResponse(param, callback) {
    var options = { 
        hostname: 'customer.crowdspots.com',
        path: '/someApi?query=' + encodeURIComponent(param),
        method: 'GET'
		}
    var results = ''; 
    var req = http.request(options, function(res) {
        res.on('data', function (chunk) {
            results = results + chunk;
        }); 
        res.on('end', function () {
            callback.apply(this, [null, results]);
        }); 
    });

    req.on('error', function(e) {
        console.error('Error in fetching: ' + param + e);
        callback.apply(this, [e, results]);
    });
}
```

If you are not interested in response data, you can skip `res.on('data',..)` event and just have `end` event for callback.

That's how simple is to make HTTP GET requests. For our usecase we needed to fire a test only after user logs in. Hence, I hacked the user cookie from Web Browser and passed in request options. You can achieve further customizations using HTTP Headers (accepts language, cookie, caching control, connections alive, etc.). 

```
    var options = { 
        hostname: 'customer.crowdspots.com',
        path: '/somePage?query=' + encodeURIComponent(param),
        method: 'GET',
        headers: {'Cookie': 'JSESSIONID=7F868662EB6182B14E9A15E9E68E19EA'}
		}
    };
```

### Firing parallel request ###

The async library comes up with asynchronous control flow patterns like serial, each, parallel, waterfall, each, eachLimit, etc. For load testing one particular we are interested is `eachLimit` with limit.

```
eachLimit(arr, limit, iterator, callback)
```

The `eachLimit` executes the `iterator` function in parallel for each of the items in the array. However it will ensure not more than `limit` parallel iterators running. Let's assume we have a collection of parameters and each parameter represent independent argument for request to the web server.

```
	var PARALLEL_N = 20;
    var paramsArray = [ "Hyderabad", "India", "Gujarat" , "Ahmedabad" , "USA", "New York" ];
	var successCount = 0;
	var errorCount = 0;
	var totalTime = 0;
	async.eachLimit(paramsArray, PARALLEL_N, 
			function(item, myCallback){
				var startTime = new Date();
				getResponse(item, function(err, result){
						var time = (new Date() - startTime);
						totalTime = totalTime + time;
						if(err) {
						}
						if(result) {
							successCount++;
						}
						myCallback();
				});
			},
			function(err, result) {
				console.log('AvgTime=' + (totalTime/paramsArray.length) + ', SuccessCount=' + nonZeroCount + ', errorCount=' + errorCount);
			}
	);
```

Store this code in app.js and run as follows:

```
$ node app.js
```

Start your test with different values of PARALLEL_N = 5, 10, 25, 50, 75, 100, 200...

### Validating Results ###

Ensure following points to prevent skewed results.

* The above example is a small demonstration with just 6 `paramsArray`. Typically you should do testing by making at least `100*PARALLEL_N` requests.
* There is *startup overhead* where you should skip initial `PARALLEL_N` which are responsible for warming up the load.
* If the HTTP responses are big, keep an eye on network bandwidth. Network bandwidth can also be a bottleneck to skew results.
 


Author: [@geekpack](https://twitter.com/@geekpack), [Ankit Jain](https://ankitjain.info/)

