# Installing and populating cache

After your service worker is registered, the browser will attempt to install then activate the service worker for your page/site.

The install event is fired when an install is successfully completed. **The install event is generally used to populate your browser’s offline caching capabilities with the assets you need to run your app offline.**

To do this, we use Service Worker’s brand new storage API — cache — a global object on the service worker that allows us to store assets delivered by responses, and keyed by their requests. This API works in a similar way to the browser’s standard cache, but it is specific to your domain. It persists until you tell it not to — again, you have full control.

```js
self.addEventListener("install", function(event) {
  event.waitUntil(
    caches.open("v1").then(function(cache) {
      return cache.addAll([
        "/sw-test/",
        "/sw-test/index.html",
        "/sw-test/style.css",
        "/sw-test/app.js",
        "/sw-test/image-list.js",
        "/sw-test/star-wars-logo.jpg",
        "/sw-test/gallery/",
        "/sw-test/gallery/bountyHunters.jpg",
        "/sw-test/gallery/myLittleVader.jpg",
        "/sw-test/gallery/snowTroopers.jpg"
      ]);
    })
  );
});
```

1. Here we add an install event listener to the service worker (hence self), and then chain a `ExtendableEvent.waitUntil()` method onto the event — this ensures that the service worker will not install until the code inside `waitUntil()` has successfully occurred.
2. Inside `waitUntil()` we use the `caches.open()` method to create a new cache called v1, which will be version 1 of our site resources cache. This returns a promise for a created cache; once resolved, we then call a function that calls `addAll()` on the created cache, which for its parameter takes an array of origin-relative URLs to all the resources you want to cache.
3. If the promise is rejected, the install fails, and the worker won’t do anything. This is ok, as you can fix your code and then try again the next time registration occurs.
4. After a successful installation, the service worker activates. This doesn’t have much of a distinct use the first time your service worker is installed/activated, but it means more when the service worker is updated (see the Updating your service worker section later on.)

## Custom responses to requests

Now you’ve got your site assets cached, you need to tell service workers to do something with the cached content. This is easily done with the `fetch` event.

![ServiceworkerHijack](https://mdn.mozillademos.org/files/12634/sw-fetch.png)

> A fetch event fires **every time any resource controlled by a service worker is fetched, which includes the documents inside the specified scope, and any resources referenced in those documents (for example if index.html makes a cross origin request to embed an image, that still goes through its service worker**.)

You can attach a fetch event listener to the service worker, then call the `respondWith()` method on the event to hijack our HTTP responses and update them with your own magic.

```js
self.addEventListener("fetch", function(event) {
  event
    .respondWith
    // magic goes here
    ();
});
```

We could start by simply responding with the resource whose url matches that of the network request, in each case:

```js
self.addEventListener("fetch", function(event) {
  event.respondWith(caches.match(event.request));
});
```

> `caches.match(event.request)` allows us to match each resource requested from the network with the equivalent resource available in the cache, if there is a matching one available. The matching is done via url and vary headers, just like with normal HTTP requests.

Let’s look at a few other options we have when defining our magic (see our Fetch API documentation for more information about Request and Response objects.)

The `Response()` constructor allows you to create a custom response. In this case, we are just returning a simple text string:

```js
new Response("Hello from your friendly neighbourhood service worker!");
```

This more complex Response below shows that you can optionally pass a set of headers in with your response, emulating standard HTTP response headers. Here we are just telling the browser what the content type of our synthetic response is:

```js
new Response("<p>Hello from your friendly neighbourhood service worker!</p>", {
  headers: { "Content-Type": "text/html" }
});
```

If a match wasn’t found in the cache, you could tell the browser to simply fetch the default network request for that resource, to get the new resource from the network if it is available:

```js
fetch(event.request);
```

If a match wasn’t found in the cache, and the network isn’t available, you could just match the request with some kind of default fallback page as a response using `match()`, like this:

`caches.match('/fallback.html');`

You can retrieve a lot of information about each request by calling parameters of the Request object returned by the FetchEvent:

```
event.request.url
event.request.method
event.request.headers
event.request.body
```

## Updating / deleting cached service worker responses :

[MDN](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers#Updating_your_service_worker)
