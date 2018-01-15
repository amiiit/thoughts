# Testing network activity

We all know testing is hard, however only the ones that have a really comprehensive test suite know how great it feels when your test suits reveals bug before they enter the development branch. One of the difficulties I turned into was to ensure my front end code executed network requests as is crucially necessary for the product I am working on.

## Service Workers

Service workers work in the background and are attached to one domain. They can track and manipulate all network activity originating from their domain and can send and receive messages. They make a really good candidate to my use case even though it will require some work to set it ip.

Here is an general overview of the set up and we'll get to the details of each step as we proceed: 
1. Your test suite must load and install a service worker. Once the service-worker is active your page must listen to messages coming from the service worker. These messages will contain the data you need concerning network activity from your page
2. The service worker you've installed in the last step needs to listen to outgoing network activity and report each request to the page you are currently testing
3. Your selenium test will execute some javascript code to be executed on your test page in order to retrieve the data you have access to from step 1.

There is a catch here and everyone that has ever worked with service workers before knows that: Service workers will only work with an encrypted connection over https, or locally on localhost.
 
 
## Installing the service worker and registering network report
```javascript
// swLoader.js

// Global object to register all network reports from the service worker
// this needs to be global for the selenium test to be able to access it
// via browser.execute(function () { return window.__e2eFetchRequests })
window.__e2eFetchRequests = []

navigator.serviceWorker.register(swURL, {scope: '/'})
  .then((registration) => {
    console.info('e2e service worker is running')

    navigator.serviceWorker.ready.then((serviceWorkerRegistration) => {
      // The controller attribute will be available when the service worker is
      // active. If not active, a simple solution will be to reload, even if this
      // is not the most elegant solution, it works for me.
      if (!navigator.serviceWorker.controller) {
        console.info('Will reload now in order to have a service-worker controller')
        window.location.reload()
      }
      navigator.serviceWorker.addEventListener('message', function (event) {
        console.log("Client Received Message: " + event.data.request.url);
        window.__e2eFetchRequests.push(event.data)
      });
    })

  }).catch(err => {
  console.warn('service worker failed', err);
})
```
## The service worker

```javascript
// sw.js
self.onfetch = function (event) {
  if (event.request.method === 'POST') {
    event.request.json().then(requestBody => {
      notifyClients(event, requestBody)
    }, err => {
      console.error('error', err)
    })
  } else {
    notifyClients(event)
  }
}

const notifyClients = (event, requestBody) => {
  // The object `clients` is provided by the browser
  // todo: reference API from MDN
  clients.matchAll().then(cs => {
    cs.forEach(client => {
      // Notify all pages related to the service worker
      // about a networking event
      client.postMessage({
        type: 'fetch',
        request: {
          url: event.request.url,
          body: requestBody
        }
      })
    })
  })
}
```

