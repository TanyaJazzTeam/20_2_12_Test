---
"$title": Use AMP as a Data Source for PWA
"$order": one
description: "If you've invested in AMP but haven't built a Progressive Web App yet, your AMP Pages can greatly simplify the development of your Progressive Web App."
formats:
  - Web sites
author: pbackaus
---

If you've invested in AMP but haven't built a Progressive Web App yet, your AMP Pages can greatly simplify the development of your Progressive Web App. In this tutorial, you will learn how to use AMP in your Progressive Web App and use existing AMP pages as a data source.

## JSON to AMP

In the most common scenario, a progressive web application is a single page application that connects to a JSON API via Ajax. This JSON API then returns datasets for navigation control and actual content for displaying articles.

Test segment 1

Then you have to convert the raw content to usable HTML and display it on the client. This process is expensive and often difficult to maintain. Instead, you can reuse existing AMP pages as a content source. Best of all, AMP allows you to do this with just a few lines of code.

Test segment 2

## Incorporate Shadow AMP into your Progressive Web App

The first step is to include a special version of AMP, which we call "Shadow AMP", in your Progressive Web App. Yes, that's right - you're loading the AMP library into the top-level page, but it doesn't actually control the top-level content. He will only "expand" those parts of our page that you are telling him about.

Include Shadow AMP in the title of your page, for example:

[source code: html]

<!-- Asynchronously load the AMP-with-Shadow-DOM runtime library. -->

&lt;script async = "" src = "https://cdn.ampproject.org/shadow-v0.js"&gt; &lt;/script&gt;

[/source]

### How do I know the Shadow AMP API is ready to use?

We recommend that you download the Shadow AMP library with the `async` attribute set. However, this means that you need to take a specific approach to understand when the library is fully loaded and ready to use.

The right signal to watch is the availability of the global `AMP` variable, and Shadow AMP uses an " [asynchronous function loading approach](http://mrcoles.com/blog/google-analytics-asynchronous-tracking-how-it-work/) " to help with this. Consider this code:

[source: javascript] (window.AMP = window.AMP || []). push (function (AMP) {// AMP is now available.}); [/source]

This code will work, and any number of callbacks added this way will actually fire when AMP is available, but why?

This code translates as:

1. "If window.AMP doesn't exist, create an empty array to take its position."
2. "then insert a callback function into the array to be executed when the AMP is ready."

It works because the Shadow AMP library, upon actual load, will realize there's already an array of callbacks under `window.AMP` , then process the entire queue. If you later execute the same function again, it will still work, as Shadow AMP replaces `window.AMP` with itself and a custom `push` method that simply fires the callback right away.

[tip type = "tip"] **TIP.** To make the above code example practical, we recommend that you wrap it in a Promise and then always use that promise before working with the AMP API. Check out our [React demo code](https://github.com/ampproject/amp-publisher-sample/blob/master/amp-pwa/src/components/amp-document/amp-document.js#L20) for an example. [/tip]

## Control navigation in your Progressive Web App

You will still have to complete this step manually. At the end of the day, it's up to you how you present links to content in your navigation concept. Number of lists? Lots of cards?

In a typical scenario, you end up with some JSON that returns ordered URLs with some metadata. Ultimately, you should receive a function callback that fires when the user clicks on one of the links, and the specified callback should include the URL of the requested AMP page. If you have one, you are ready for the last step.

## Use the Shadow AMP API to render the page in line

Finally, if you want to display content after user action, it's time to get the appropriate AMP document and let Shadow AMP take over. First, implement a function to get the page like this:

[source: javascript] function fetchDocument (url) {

// unfortunately fetch () does not support fetching documents, // so we have to resort to good old XMLHttpRequest. var xhr = new XMLHttpRequest ();

return new promise (function (allow, reject) {xhr.open ('GET', url, true); xhr.responseType = 'document'; xhr.setRequestHeader ('Accept', 'text / html'); xhr.onload = function () {// .responseXML contains a ready-to-use Document object resolve (xhr.responseXML);}; xhr.send ();}); } [/source]

[tip type = "important"] **IMPORTANT -** To simplify the above code example, we have omitted error handling. You should always be careful to catch and handle errors correctly. [/tip]

Now that we have our `Document` object ready to use, it's time to let AMP take over the rendering. Get a reference to the DOM element that is serving as a container for the AMP document, then call `AMP.attachShadowDoc()` , for example:

[source: javascript] // This can be any DOM element var container = document.getElementById ('container');

// AMP page you want to display var url = "https: //my-domain/amp/an-article.html";

// Use our fetchDocument method to get the document fetchDocument (url) .then (function (doc) {// Let AMP take over and render the page var ampedDoc = AMP.attachShadowDoc (container, doc, url);}); [/source]

[tip type = "tip"] **TIP.** Before actually submitting the AMP document, it's time to remove the page elements that make sense when rendering the AMP page offline but not inline: for example, footers and headers. ... [/tip]

And it's all! Your AMP page appears as a child page of your generic progressive web app.

## Get out after yourself

Chances are, your user will switch from AMP to AMP in your Progressive Web App. When abandoning a previously rendered AMP page, always inform AMP about it, for example:

[source: javascript] // ampedDoc is the link returned from AMP.attachShadowDoc ampedDoc.close (); [/source]

This will inform AMP that you are no longer using this document and will free up memory and CPU usage.

## See it in action

[video src = "/ static / img / docs / pwamp_react_demo.mp4" width = "620" height = "1100" loop = "true", controls = "true"]

You can see the AMP in PWA pattern in action in the [React sample](https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa) we created. It demonstrates smooth transitions during navigation and comes with a simple React component that completes the steps above. It's the best of both worlds - flexible custom JavaScript in a progressive web app and AMP for content management.

- Download the source code here: [https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa](https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa)
- Use a standalone React component via npm: [https://www.npmjs.com/package/react-amp-document](https://www.npmjs.com/package/react-amp-document)
- See how it works here: [https://choumx.github.io/amp-pwa/](https://choumx.github.io/amp-pwa/) (best on phone or mobile emulation)

You can also see a sample PWA and AMP using the Polymer framework. The example uses [amp-viewer](https://github.com/PolymerLabs/amp-viewer/) to embed AMP pages.

- Take the code here: [https://github.com/Polymer/news/tree/amp](https://github.com/Polymer/news/tree/amp)
- See it in action here: [https://polymer-news-amp.appspot.com/](https://polymer-news-amp.appspot.com/)
