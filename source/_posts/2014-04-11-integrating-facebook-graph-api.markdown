---
layout: post
title: "Integrating Facebook Graph API"
date: 2014-04-11 12:37
comments: true
author: Narendra Yadala
categories: 
- Facebook Graph
- Node.js
---

Facebook is undoubtedly the most popular social media site out on the internet and surprisingly it was difficult to find good support on accessing the Graph API using Node.js. So this is the approach we followed to perform basic actions on facebook for our app. 

<!-- more -->

<strong>STEP 1 : CREATE AN APP</strong>
First up is to create an app if you dont have one. This our mode to access facebook externally. So here are the steps
a) Log on to <a href="https://developers.facebook.com">developers facebook page</a> and sign in. Once there click on the app menu in the menu bar and select "create a new app".
b) Follow the subsequent steps and once the app is successfully created, you will be redirected to the dashboard page of your application.
c) You will find the a settings tab on the left hand side. Click on the tab and enter the domain from which you would be trying to access the facebook API. <em> If you are using your local machine for development, enter "localhost"</em>
d) Keep a note of the AppID and AppSecret as we will be using these to exchange information with facebook.

<strong>STEP 2 : VERIFICATION and AUTHORIZATION</strong>
Now that we have set up a facebook app, we now need a user to authorize the app to perform actions on his behalf. A user can authorize the app created using the OAuth protocol. Facebook only supports OAuth 2.0 unlike twitter which only supports OAuth 1.0 and LinkedIN which supports both the versions. So the steps for authorizing are slightly different when compared to twitter and LinkedIN.
We would be needing the following node modules1) node-oauth <a href="https://github.com/ciaranj/node-oauth"> Link on GIT Hub</a>2) request <a href="https://github.com/mikeal/request"> Link on GIT Hub</a>
Changes in package.json :     "oauth": "0.9.10",     "request": "2.30.0"
Since we were using express middleware for our application. Here is the code involved to validate the app.
```javascript
var request = require('request');
var OAuth2 = require('oauth').OAuth2;
var oauth2 = new OAuth2("YOUR Facebook APP ID","YOUR Facebook APP SECRET","","https://www.facebook.com/dialog/oauth","https://graph.facebook.com/oauth/access_token",null);
app.get('/facebook/auth',function (req, res) {    
  var redirect_uri = "Your App domain" + "/Path_To_Be_Redirected_to_After_Verification";    // For eg. "http://localhost:3000/facebook/callback"    
var params = {'redirect_uri': redirect_uri, 'scope':'user_about_me,publish_actions'};    
res.redirect(oauth2.getAuthorizeUrl(params));});
```
Now every time you hit the path "/facebook/auth" from your web application you will hit this method.The redirect_uri parameter mentions which URI to be redirected to by facebook. Remember the domain of this URI must be the one mentioned in the settings of the facebook app. The scope parameter is a comma seperated list of permissions the user has to authorize the app. For example, the permission "user_about_me" allows the app to access the complete profile of the user and the permission "publish_actions" allows the app to post, like , share and perform other actions on the behalf of the user. A comprehensive list of permissions can be referred from the <a href="https://developers.facebook.com/docs/reference/login/">facebook permissions</a> page.
Once facebook successfully verfies these credentials the app is redirected to the redirect_uri with a query parameter called "code" appended to it. This code can then be successfully exchanged for the user's access token.Here's the snippet:
```javascript
app.get("/Path_To_Be_Redirected_to_After_Verification", function (req, res) {	
if (req.error_reason) {		
res.send(req.error_reason);	
}	
if (req.query.code) {		
var loginCode = req.query.code;		
var redirect_uri = same_as_previous_redirect_uri; // Path_To_Be_Redirected_to_After_Verification                // For eg. "/facebook/callback"		
oauth2.getOAuthAccessToken(loginCode,{ grant_type: 'authorization_code', redirect_uri: redirect_uri}, 
     function(err, accessToken, refreshToken, params){				
      if (err) {
             console.error(err);		                    
             res.send(err);				
      }				
      var access_token = accessToken;				
      var expires = params.expires;                                
      req.session.access_token = access_token;                                
      req.session.expires = expires;			   
   });	
 }};)
```
Now when facebook redirects to the redirect_uri we then take the query paramter "code" and exchange it for the users's access token. We use the getOAuthAccessToken method provided by the node-oauth module with the parameters as shown in the snippet above. The important point to be noted is that the redirect URI mentioned here should be same as that of the redirect_uri mentioned while verification as facebook uses this URI to verify if the same web application is accessing it. 
If the user successfully authorizes the app an error will not be returned and the access_token along with its expiry time is returned(which is typically around one month). Post the expiry period a new access token has to be requested.
The access_token has to be stored because to perform any action on behalf of the user we will require it. In this case we are just storing it in the session.

<strong>STEP 3 : GET USER PROFILE</strong>
Now that we have a valid access_token impossible is nothing, well atleast for the permissions you have authorization. Since the user has already given us access for viewing his profile let us fetch it using the graph API. Graph API being restful allows us to access/modify it using basic REST methods like GET, POST, PUT or DELETE.
For getting the user profile check the following code snippet
```javascript
app.get('/get_fb_profile', function(req, res) 
{	
 oauth2.get("https://graph.facebook.com/me", req.session.accessToken, function(err, data ,response) {
   if (err) {			
       console.error(err);			
       res.send(err);		
   } else {			
       var profile = JSON.parse(data);			
       console.log(profile);                        
       var profile_img_url = "https://graph.facebook.com/"+profile.id+"/picture;		
   }	
 });});
```

As we can see, we just need to send a get request to "https://graph.facebook.com/me" with the access token as a query parameter and the response will contain the user profile as a JSON string. The profile_img_url is not explicitly returned using this request but can be obtained as shown in the snippet above.

<strong>STEP 4 : POST/SHARE ON USER'S WALL</strong>
A post on the user's wall can be done by using a simple http POST method with the post parameters in the request body. Now the node-auth module is not very user friendly on performing custom http requests. So we decided to go with the "request" module. Check out the following snippet, you'll understand better
```javascript
app.post('/post_to_fb', function(req,res) {	
  var url = 'https://graph.facebook.com/me/feed';	
  var params = {	        
    access_token: req.session.access_token,message: req.body.text,link: req.body.url	        	
  };	
  request.post({url: url, qs: params}, function(err, resp, body) {	      
  if (err) {
    console.error(err)	    	  
    return;	      
  }	      
   body = JSON.parse(body);	      
   if (body.error) {		  
    var error = body.error.message;	    	  
    console.error("Error returned from facebook: "+ body.error.message);	    	  
    if (body.error.code == 341) {	    	     
      error = "You have reached the post limit for facebook. Please wait for 24 hours before posting again to facebook."  		     
      console.error(error);	    	  
    }	    	  
    res.send(error);
   }	      
   var return_ids = body.id.split('_');	      
   var user_id = return_ids[0];	      
   var post_id = return_ids[1];	      
   var post_url = "https://www.facebook.com/"+user_id+"/posts/"+post_id;	      
   res.send(post_url);	
  });});
```

So, with the help of "request" we are sending a http POST request to the graph API on "https://graph.facebook.com/me/feed". We are assuming that all the parameters are accessed from the body of the node request object(req). The access token however is accessed from the session.  A complete set of parameters that can be sent to the facebook POST object can be accessed on the <a href="https://developers.facebook.com/docs/reference/api/post/">facebook post object</a> page. 
An important point to be noted is that the facebook bot actually crawls through the parameter mentioned in the parameter link and constructs  the facebook card. So be careful not to give "localhost" domain. Cause this will throw an error. A precaution has to be taken not to post wildly(spam) because facebook may then consider you to be a bot and prevent you from posting for sometime(Mostly around 24 hours). So be careful when you aure developing.
If the post to the user's wall is successful, facebook will return two long numbers seperated by an underscore(_). The first number is the user_id and the second number is the post_id. So the post can be directly accessed by the URL constructed in the code snippet.
Sharing is similar to posting but the only difference is that in the parameter link, the url of the feed to be shared has to be mentioned. The response obtained is also in the same format as that of posting.

<strong>STEP 5 : LIKE/UNLIKE A FEED</strong>
Now that we've posted something on facebook lets check out how can like it and then unlike it. Check out the snippet shown below.
```javascript
app.post('/like_unlike_post', function(req, res) {	
  var post_id = req.body.post_id;	
  var action = req.body.action;	var url = 'https://graph.facebook.com/'+post_id+'/likes';	
  var method;	if ( action == 'like' ) {
    method = 'POST';	
  } else if ( action == 'unlike' ) {		
    method = 'DELETE';	}	
  if (method) {	
    var params = {
      access_token: access_token,		
    }; 		
  request({url:url , method:method, qs: params}, function(err, resp, body) {			
    if (err) {
       console.error("Error while performing "+action+" on fb :" + err);				
       res.send(err);			
    }			
    body = JSON.parse(body);			
    if (body.error) {				
       console.error("Error while performing "+action+" on fb :" + JSON.stringify(body.error));
    if (body.error.code == 200) {
       res.send("You do not have the permission to like the post.");
    } else {					
       res.send(body.error.message);				
    } return;			
   }			
   res.send(action + " Successful");		
   });	
  } else {		
 res.send("Action not mentioned");	
}});
```

Like and unlike are performed on the same graph api url, the only difference being the request method used to access it. For "like", a POST request has to be sent and for "unlike" a DELETE request. If the user has decided to keep his post private or if the post has been deleted, facebook will return an error with error code 200. So you can handle it accordingly.
That kinda sums up the basics of using the graph api using node js. There is much more we can do using the graph API but now we can atleast use this as a stable and simple platform to work on the graph API.

<strong>SHOUT OUTS</strong>
Thanks to <a href="https://twitter.com/@non_local">@non_local</a> and <a href="https://twitter.com/@geekpack">@geekpack</a> for allowing me to work on this endeavor of theirs.

Author: <a href="https://twitter.com/@non_local">@non_local</a>
