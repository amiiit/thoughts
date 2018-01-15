Bugs in network activity may not degrade the user-experience of your product but they might still cause big loss to your product or business. Take for example an ad provider: An ad provider basically earns a small amount of money if a banner of one of its clients was viewed by a user and a bit more money if the banner was clicked. It works because the banner is running a script that reports over the internet that it was viewed and when the user clicked any banner.

Type caption for image (optional)
Now what would happen if a banner stopped reporting? Would anyone notice? The user couldn’t care less, the advertiser as well since the ad didn’t loose any aspect of its usability: it would still show relevant offers to users and redirect them to an offer when clicked. In the worst case, the person who will notice this failure is the billing department when they sit down in the end of the month to write bills to all clients and that they have no data in order to bill the advertisers. By that point we might have lost millions.

In this article I will demonstrate how I went beyond the capabilities of Selenium and WebDriver to track and assert the network activity of a web application, a feature which is missing from the most popular tool for automated testing of web applications. This will allow us to assert on all outgoing network activity from our application.

The only pattern I found around the web was using a test proxy. There is a gross sketch of how it goes. I won’t get into details though:

Testing network requests via a test proxy
Start a proxy server together with your test suite
Modify your application code in such way that the requests you want to test are going through your test proxy
Use the data collected by the proxy to do test assertions
While this works well and fairly easy to set up, I was worried about adding complexity to my application code. Having to change my application code just didn’t feel very clean nor scalable.

Use Service Worker and start asserting on real traffic
What I was looking for is a non intrusive way of recording network requests in the browser itself and then reading this information from my tests. Let’s start from the end and see how such a test may look like:

Type caption for embed (optional)
Neat, huh? Now we can sleep better knowing that our tracking request will never ever break again. Let me show you now how do achieve this. It’s not very easy to set up but it’s worth it.

The Service Worker
The service worker itself is quite boring. Nothing much is happening there. It simply listens to all fetch events which include AJAX, Pixels and pretty much any other HTTP request that is going through your page.

Type caption for embed (optional)
In case you are wondering what is this clients and where did it come from? The clients here represent your page and any iframes you might have that load content from your domain. In case you are interested, read more about it in the MDN documentation. To be on the safe side, we message all available clients to notify them about any network request that is going on.

Loading the service worker and listening to messages
There are quite a few Webpack plugins out there that promise easy life with loading service-workers but none of them ever satisfied me. My favorite solution is the one by Michal Zalecki. Here is how I load my service worker:

Type caption for embed (optional)
There are quite a few things going on in the loader:

This rather unusual import statement will provide us a URL to the service worker itself without having to add it to Webpack's configuration. This is taken from Michal’s post mentioned above. Don’t forget to install file-loader as a dev-dependency if you haven’t so far.
{scope: '/'} grants us control to all routes in our domain and all requests that are being initiated from them including any frames that load their source from our domain.
navigator.serviceWorker.controller won’t be set in the first load of the page. Whether it’s a bad design or a feature is arguable. You can read here for more advice about this issue. The solutions mentioned didn’t work for me, but after doing a reload() I could access an active Service Worker and get messages for all fetch events. I'm glad to get feedback if anyone has more elegant solutions.
Most importantly, we’re listening to all message from our service worker and pushing them into a global array, exposed to our tests to read. If your service worker is doing more things, you might consider adding a type property to the messages.
Create an index.html for the e2e tests to load
My index.html is super trivial so I could put a copy of my development index.html in the test folder which has an additional <script src='swLoader.js'> . If you have a lot going on in your index.html you might better use HtmlWebpackPlugin or any other templating solution to keep your codebase DRY.

Type caption for embed (optional)
The main difference to your production index.html is that it loads swLoader.js alongside the application script. swLoader will install the service-worker and expose the network requests as a global object for your tests to use.

Tell Webpack to serve this when on e2e testing
The Webpack part is where we need to do some tweaks. The main thing to understand here is that your local machine (or the CI worker) serves the application code from one IP address while the test-browser might be running on a different IP address on your local network. So pay attention as you read this extract of my Webpack’s configuration:
