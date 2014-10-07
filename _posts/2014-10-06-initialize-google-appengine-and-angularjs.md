---
layout: post
title: "Initialize Google Cloud Endpoints and AngularJS"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2014-10-06T21:44:16-07:00
comments: true
---

Google Cloud endpoints has specific instructions to initialize itself as described [here.](https://cloud.google.com/appengine/docs/java/endpoints/consume_js)
This involves defining the script
{% highlight javascript linenos%}
<script src="https://apis.google.com/js/client.js?onload=init">
</script>
{% endhighlight %}
 and initializing the apis in the init method
{% highlight javascript linenos%}
 var ROOT = 'https://your_app_id.appspot.com/_ah/api';
gapi.client.load('your_api_name', 'v1', function() {
  doSomethingAfterLoading();
}, ROOT);
{% endhighlight %}

However integrating this with AngularJS is not that straight forward. For one the javascript cloud endpoint does not support promises. This makes it difficult to initialize many services. Also the since the cloud endpoint initialization involves the use of a callback function, it is not straight forward to intialize within angular.

##Decorating Cloud Endpoint client with promises 

A good way to decorate the endpoint api with promises is to write a service
The below implementation defines three methods

1. initialize
2. perform a gapi call with authenitcation
3. do a sign in using oauth 2.0

{% highlight javascript linenos%}
angular.module('webApp')
  .factory('cloudendpoint', function ($q) {
    return {
        init:function() {
            var ROOT = '//' + window.location.host + '/_ah/api';
            var hwdefer=$q.defer();
            var oauthloaddefer=$q.defer();
            var oauthdefer=$q.defer();
            
            gapi.client.load('helloworld', 'v1', function() {
                hwdefer.resolve(gapi);
            }, ROOT);
            gapi.client.load('oauth2', 'v2', function(){
                oauthloaddefer.resolve(gapi);
            });
            var chain=$q.all([hwdefer.promise,oauthloaddefer.promise]);
            return chain;
        },
    	doCall: function(callback) {
		    var p=$q.defer();
            gapi.auth.authorize({client_id: 'clientid', scope: 'https://www.googleapis.com/auth/userinfo.email',
            immediate: true}, function(){
                var request = gapi.client.oauth2.userinfo.get().execute(function(resp) {
                if (!resp.code) {
                    p.resolve(gapi);
                } else {
                    p.reject(gapi);
                }
                });
            });
			return p.promise;
    	},
        sigin:function(callback) {
            gapi.auth.authorize({client_id: 'clientid', scope: 'https://www.googleapis.com/auth/userinfo.email',
                     immediate: false}, callback);
        }
    }
});
{% endhighlight %}

An easy way to wrap multiple calls synchronously using promises is by using $q.all(..)

##Initializing Google API client in angular

To intialize the google api client in angular the scripts must be loaded in the following order

1. AngularJS
2. Your Controllers
3. GAPI
{% highlight javascript linenos%}
<script src="bower_components/angular/angular.js"></script
<script src="scripts/app.js"></script>
<script src="scripts/services/api.js"></script>
<script src="scripts/controllers/main.js"></script>
<script src="https://apis.google.com/js/client.js?onload=init"></script>
{% endhighlight %}

In one of your controllers define the init function that would serve as the callback for gapi. This function would delegate the call to the init function that is defined on the window object inside the controller
{% highlight javascript linenos%}
'use strict';
function init() {
	window.init(); // Calls the init function defined on the window

}
angular.module('webApp')
  .controller('LoginCtrl',['$scope','$rootScope','$location','cloudendpoint','$window',function ($scope,$rootScope,$location,cloudendpoint,$window) {
  	// the init function that will bridge the gapi initialize event with angular
    $window.init= function() {
  		$scope.$apply($scope.initgapi);
	};
	$scope.initgapi=function() {
		cloudendpoint.init().then(function(){alert('inited');},function(){alert('notinited')});
	}
    $scope.greeting=function(greetingform) {
      cloudendpoint.doCall().then(function() {
        gapi.client.helloworld.greetings.authed({}).execute(function(resp){
          alert(resp);
        });
      });
    $scope.signin=function(loginform) {
   	  gapi.auth.authorize({client_id: 'clientid',
		    scope: 'https://www.googleapis.com/auth/userinfo.email', immediate: false},
		    function(){alert('hello');});
	 }
	};
  }]);
{% endhighlight %}

Voila AngularJS and Cloudendpoints are now initialized.



