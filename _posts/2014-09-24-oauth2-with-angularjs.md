---
layout: post
title: "OAuth2 With AngularJS"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2014-09-24T21:48:32-07:00
comments: true
---
This example shows how to integrate with Googles OAuth 2.0 for client side javascript using AngularJS.
The flow for the authentication is as shown below

![Flow](https://developers.google.com/accounts/images/tokenflow.png)

The flow starts by redirecting the browser to the google servers for authentication.

{% highlight javascript linenos%}
angular.module('angularoauthexampleApp')
  .controller('MainCtrl', function ($scope) {
    $scope.login=function() {
    	var client_id="your client id";
    	var scope="email";
    	var redirect_uri="http://localhost:9000";
    	var response_type="token";
    	var url="https://accounts.google.com/o/oauth2/auth?scope="+scope+"&client_id="+client_id+"&redirect_uri="+redirect_uri+
    	"&response_type="+response_type;
    	window.location.replace(url);
    };
  });
  {% endhighlight %}

The most important parameters in the url are

1. client_id - This is the client id of the application that will identify itself to google
2. scope - this is the scope of permissions that you are requesting the user to grant
3. redirect_url - This is the url which google will call back after the authentication is sucessful
4. response_type - Use token for client side javascript. This will return the token in the query parameters of the redirect url
5. state - optional state that would be received on the callback. Useful for passing state information and for security validation

Use the window.location object to setup the redirect

{% highlight javascript linenos%} 
   	window.location.replace(url);
{% endhighlight %}

When the authentication is sucessful, we need to now configure the routes in angular to parse out the token.

{% highlight javascript linenos%}
angular
  .module('angularoauthexampleApp', [
  ])
  .config(function ($routeProvider) {
    $routeProvider
      .when('/access_token=:accessToken', {
        template: '',
        controller: function ($location,$rootScope) {
          var hash = $location.path().substr(1);
          
          var splitted = hash.split('&');
          var params = {};

          for (var i = 0; i < splitted.length; i++) {
            var param  = splitted[i].split('=');
            var key    = param[0];
            var value  = param[1];
            params[key] = value;
            $rootScope.accesstoken=params;
          }
          $location.path("/about");
        }
      })
  });
{% endhighlight %}

The redirect url will look like http://localhost:4000#access_token=....

As the redirect url contains the # it works well with angular only if the redirect url does not contain a path, and it would get routed to the given controller. All that remains is to parse the access token from the redirect url.

It is also possible to use the html5 mode in the location provider to disable the hashbang mode and work properly with regular urls.

[Fork the example code at GitHub](https://github.com/anandsekar/angularoauthexample)


