# SERVICE WORKERS

Using javascript, service workers provide the capability of offering an offline experience to web apps, which wouldn't have been possible earlier.

> Using a Service worker you can easily set an app up to use cached assets first, thus providing a default experience even when offline, before then getting more data from the network (commonly known as Offline First). This is already available with native apps, which is one of the main reasons native apps are often chosen over web apps.

## Service worker requirements:

- Firefox Nightly: Go to about:config and set dom.serviceWorkers.enabled to true; restart browser.
- Chrome Canary: Go to chrome://flags and turn on experimental-web-platform-features; restart browser (note that some features are now enabled by default in Chrome.)
- Opera: Go to opera://flags and enable Support for ServiceWorker; restart browser.
- Microsoft Edge: Go to about:flags and tick Enable service workers; restart browser.
-

**You’ll also need to serve your code via HTTPS — Service workers are restricted to running across HTTPS for security reasons. GitHub is therefore a good place to host experiments, as it supports HTTPS.**

In order to facilitate local development, localhost is considered a secure origin by browsers as well.

## Registering a service worker

```js
// Check if browser supports service worker
if ("serviceWorker" in navigator) {
  /* Register service worker function:-

     first argument -> service worker url (relative to origin NOT js)

     second argument -> object containing 
     optional parameters passed to the 
     function ,scope
     A USVString representing a URL that defines a service worker's registration scope; that is, what range of URLs a service worker can control. This is usually a relative URL. It is relative to the base URL of the application. By default, the scope value for a service worker registration is set to the directory where the service worker script is located. 


  */
  navigator.serviceWorker
    .register("/sw-test/sw.js", { scope: "/sw-test/" })
    .then(function(reg) {
      // registration worked
      console.log("Registration succeeded. Scope is " + reg.scope);
    })
    .catch(function(error) {
      // registration failed
      console.log("Registration failed with " + error);
    });
}
```

- This registers a service worker, **which runs in a worker context, and therefore has no DOM access**. You then run code in the service worker outside of your normal pages to control their loading.

- A single service worker **can control many pages**. **Each time a page within your scope is loaded, the service worker is installed against that page and operates on it**. Bear in mind **therefore that you need to be careful with global variables in the service worker script: each page doesn’t get its own unique worker.**

> Note: Your service worker **functions like a proxy server**, allowing you to modify requests and responses, replace them with items from its own cache, and more.

> Note: One great thing about service workers is that if you use feature detection like we’ve shown above, browsers that don’t support service workers can just use your app online in the normal expected fashion. Furthermore, if you use AppCache and SW on a page, browsers that don’t support SW but do support AppCache will use that, and browsers that support both will ignore the AppCache and let SW take over.

## Service workers basic architecture:

1. The service worker URL is fetched and registered via `serviceWorkerContainer.register()`.
2. If successful, the service worker is executed in a ServiceWorkerGlobalScope; this is basically a special kind of worker context, running off the main script execution thread, with no DOM access.
3. The service worker is now ready to process events.
4. Installation of the worker is attempted when service worker-controlled pages are accessed subsequently. An Install event is always the first one sent to a service worker (this can be used to start the process of populating an IndexedDB, and caching site assets). This is really the same kind of procedure as installing a native or Firefox OS app — making everything available for use offline.
5. When the oninstall handler completes, the service worker is considered installed.
   Next is activation. When the service worker is installed, it then receives an activate event. **The primary use of onactivate is for cleanup of resources used in previous versions of a Service worker script**.
6. The Service worker will now control pages, but only those opened after the register() is successful. i.e. a document starts life with or without a Service worker and maintains that for its lifetime. So documents will have to be reloaded to actually be controlled.

![Lifecycle](https://mdn.mozillademos.org/files/12636/sw-lifecycle.png)

## Why is my service worker failing to register?

This could be for the following reasons:

1. You are not running your application through HTTPS.
2. The path to your service worker file is not written correctly — **it needs to be written relative to the origin, not your app’s root directory.** In our example, the worker is at https://mdn.github.io/sw-test/sw.js, and the app’s root is https://mdn.github.io/sw-test/. **But the path needs to be written as /sw-test/sw.js, not /sw.js.**
3. The service worker being pointed to is on a different origin to that of your app. This is also not allowed.

![ServiceWorkerRequirements](https://mdn.mozillademos.org/files/12630/important-notes.png)

Also note:

- The service worker will only catch requests from clients under the service worker's scope.
- The max scope for a service worker is the location of the worker.
- If your server worker is active on a client being served with the Service-Worker-Allowed header, you can specify a list of max scopes for that worker.
- In Firefox, Service Worker APIs are hidden and cannot be used when the user is in private browsing mode.
