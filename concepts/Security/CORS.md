# Cross-Origin Resource Sharing (CORS)

<!--
Your default Sails setup is already equipped to handle AJAX requests from a web page on the same domain.  But what if you need to handle AJAX requests originating from other domains?  You could set up your browser JSONP That's where [CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) comes in.
-->

[CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) is a mechanism that allows browser scripts on pages served from other domains (e.g. myothersite.com) to talk to your server (e.g. api.mysite.com).  Like JSONP, the goal of CORS is to function as a secure method to circumvent the [same-origin policy](http://en.wikipedia.org/wiki/Same-origin_policy); allowing your Sails server to successfully respond to requests from client-side JavaScript code running on a page from some other domain.  But unlike JSONP, it works with more than just GET requests.

Sails can be configured to allow cross-origin requests from a list of domains you specify, or from every domain.  This can be done on a per-route basis, or globally for every route in your app.


### Enabling CORS

For security reasons, CORS is disabled by default in Sails.  But enabling it is dead-simple.

To allow cross-origin requests from _any_ domain to _any_ route in your app, simply enable `allRoutes` in [`config/cors.js`](http://sailsjs.org/documentation/reference/sails.config/sails.config.cors.html):

```javascript
allRoutes: true
```

> Note that if this is all you do, your app will not be protected against attacks that exploit CORS.  To prevent third-party sites from being able to trick your logged-in users into making unauthorized requests to your app, you should either set `credentials: false` (for requests that don't require a session) or set `origin` to a specific set of trusted domains.  You can also configure these settings on a per-route basis (see below).

To allow cross-origin requests to _any_ route in your app from a whitelist of trusted domains (**the recommended approach**), use:

```javascript
allRoutes: true,
origin: 'sailsjs.com,beta.sailsjs.com,0.11.sailsjs.com'
```

See [`sails.config.cors`](http://sailsjs.org/documentation/reference/sails.config/sails.config.cors.html) for a comprehensive reference of all available options.


### Configuring CORS For Individual Routes
Besides the global CORS configuration, you can set up individual routes in `config/routes.js` to accept (or deny) cross-origin requests.  To indicate that a route should accept CORS requests using the configuration parameters in `config/cors.js`, set its `cors` property to `true`:

```javascript
"get /foo": {
   controller: "FooController",
   action: "index",
   cors: true
}
```

If you have the `allRoutes` parameter set to `true` in `config.cors.js`, but you want to exempt a specific route, you can do so by explicitly setting its `cors` property to `false`:

```javascript
"get /foo": {
   controller: "FooController",
   action: "index",
   cors: false
}
```

To override specific CORS configuration parameters for a route, add a `cors` property object:

```javascript
"get /foo": {
   controller: "FooController",
   action: "index",
   cors: {
     origin: "http://sailsjs.org, http://sailsjs.com",
     credentials: false
   }
}
```

### Security Levels

The browser's cross-origin policy allows requests to be sent-- it simply does not allow the browser to receive the response.  Similarly, Sails will still process all the requests that come in regardless of domain, even with CORS enabled.  But for routes with CORS enabled, it will simply set the appropriate headers on the response so that the *browser* can decide whether or not to expose the response.  For example, if you send a `GET` request to `/foo/bar` from a domain that is not in your CORS whitelist (`origin`), the `bar` action in your `FooController.js` file will still run, but the browser will throw away the result.  This may seem counterintuitive, but it is important because it allows non-browser-based clients (like [Postman](https://www.getpostman.com) and [curl](http://curl.haxx.se/)) to work while still blocking the kind of attacks that the [Same-Origin Policy](http://en.wikipedia.org/wiki/Same-origin_policy) is meant to protect against.

If you want to completely prevent Sails from processing requests from disallowed domains, you can use the `securityLevel` setting:

```javascript
module.exports.cors = {
  allRoutes: true,
  origin: "http://sailsjs.org",
  securityLevel: 1
}
```

Security level 1 (high) will respond with a 403 status code to any request from a disallowed origin prefixed with the `http` or `https` protocol.  Security level 2 (very high) will do the same, but extended to *all* protocols (so things like Postman and curl won't work).


<docmeta name="displayName" value="CORS">
