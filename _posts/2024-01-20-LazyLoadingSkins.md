---
title: "Lazy Loading Handshake Skins"
date: 2024-01-20
tags: Performance JavaScript
---

## What is Lazy Loading?
A possible way to improve the performance and user experience of a landing page with a skin that is not visible on the initial viewport is to use a technique called Lazy Loading. Long attributed to handling large graphic files, Lazy loading means that we only render the skin when it comes into the view of the user, as they scroll down the page. This way, we avoid wasting resources and loading time on something that the user may not see or interact with. Lazy loading can be implemented using JavaScript events that detect when an element is in the viewport, and then trigger the rendering of the skin.

## Setup
Let's say we have a Handshake Skin named SWM-Home-Page-Main that is the top level skin on a Site Page. In that skin we will be loading a number of skins using the RenderSkin widget, including one named SWM-Home-MyKPI.  SWM-Home-MyKPI is at the bottom of the page, and will never be visible until the user scrolls down to see it. 

While you would normally use the Handshake Widget RenderSkin to do this work, we want to "lazy load" this skin, so that it rendered only when the section becomes visible.  

To set this up we replace the normal RenderSkin widget with an EmbeddedHTML element. In the content property for this element we add a DIV and give it a class of ```.lazy-load-skin```.  Along with that class name, we provide several data attributes. Only one, data-skinname, will be required. Any additional data attributes will be passed as input properties to the skin. An optional data attribute name loadingMessage can be used to control what is displayed in the div until the skin is loaded. 

Before setting up the lazy load process, the skin would have been rendered like this:

{% include codeheader.html %}
```xml
<div>
    <RenderSkin skin="SWM-Home-MyKPI">
        <property name="tkid" value="12345" />
        <property name="cardTitle" value="My KPI" />
    </RenderSkin>
</div>
```

For lazy load, we mimic that pretty closely, using standard html markup like this:

{% include codeheader.html %}
```xml
<div>
    <EmbeddedHtml>
        <![CDATA[<div class="lazy-load-skin"
            data-skinname="SWM-Home-MyKPI"
            data-tkid="12345"
            data-cardtitle="My KPI"
            data-loadingmessage = "Loading your key financial metrics, this will take a few moments ..."
        ></div>]]>
    </EmbeddedHtml>
</div>
```

## Code
Now we will need a programmatic way to detect when the element is visible in the viewport, and render the skin.  The JavaScript [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) makes the detection relatively easy. The Handshake function HSRenderSkinAsync makes short work of rendering the component. 

Here is the code, which can be included in JavaScript block of skin which is rendering SWM-Home-MyKPI. 

{% include codeheader.html %}
```js
const LazyLoadSkins = document.querySelectorAll('.lazy-load-skin');

function lazyLoadSkin(entries) {
    entries.map((entry) => {
        if (entry.isIntersecting) {
            let msg = entry.target.dataset.loadingmessage || "Loading content, this might take a moment..."
            LazyLoadSkinIObserver.unobserve(entry.target); 
            
            let element = entry.target.parentElement;
            jQuery(element).html(`<div class="lazy-skin-loading-message">${msg}</div>`);
            for (let key in entry.target.dataset) {
                HSSetProperty(element, key, entry.target.dataset[key])
            }
            let properties = HSComputeProperties(HSFindParent(element));
            properties += "&_HS_ControlType=" + escape(entry.target.dataset.skinname);
            HSRenderSkinAsync(webService, kendo.htmlEncode(properties), element, function () {
                _hsEvalScripts(element);      
            });            
        }

    });
}

// Create an intersection observer
const LazyLoadSkinIObserver = new IntersectionObserver(lazyLoadSkin);

// Start observing the target element
LazyLoadSkins.forEach(renderSkin => LazyLoadSkinIObserver.observe(renderSkin));

{% include codeheader.html %}
```
Let's dissect this code in the order of things happening. First, we get an array of all the elements in the page with the lazy-load-skin class. 

{% include codeheader.html %}
```js
const LazyLoadSkins = document.querySelectorAll('.lazy-load-skin');
```

Next, we need to create an instance of the IntersectionObserver object.  This constructor takes 2 parameters, the first is the callback that will contain the code that does the work.  The second, which we are not using, is the options object passed into the IntersectionObserver() constructor which let you control the circumstances under which the observer's callback is invoked. The defaults will work for us. 

{% include codeheader.html %}
```js
const LazyLoadSkinIObserver = new IntersectionObserver(lazyLoadSkin);
```

Finally, with our intersection observer created, we will setup each element in the LazyLoadSkins array to be observed. 

{% include codeheader.html %}
```js
LazyLoadSkins.forEach(renderSkin => LazyLoadSkinIObserver.observe(renderSkin));
```

Now, there is nothing to do except wait until the element becomes visible in the viewport. Once that happens, the observer will pass the entries of what has been observed to the lazyLoadSkin handler we defined. 

Step by step through the lazyLoadSkin function we perform the following:

Use the array map method to step through each instance on an entry

{% include codeheader.html %}
```js
entries.map((entry) => {
```
If this entry has become visible in the viewport, then entry.isIntersection is true

{% include codeheader.html %}
```js
if (entry.isIntersecting) {
```

We only want to run this code once, and then stop observering it.  This is done with the unobserve method.

{% include codeheader.html %}
```js
LazyLoadSkinIObserver.unobserve(entry.target); 
```

Display a block of text in the target element's parent container, which will automatically be replaced with the skin when it renders.

{% include codeheader.html %}
```js 
let element = entry.target.parentElement;
let msg = entry.target.dataset.loadingmessage || "Loading content, this might take a moment..."            
jQuery(element).html(`<div class="lazy-skin-loading-message">${msg}</div>`);
```

Map the data attributes of the target to the Handshake Properties of the target's Parent. Note: use the value of data-skinname to add the required property name _HS_ControlType.  

{% include codeheader.html %}
```js
for (let key in entry.target.dataset) {
    HSSetProperty(element, key, entry.target.dataset[key])
}
let properties = HSComputeProperties(HSFindParent(element));
properties += "&_HS_ControlType=" + escape(entry.target.dataset.skinname);
```

Finally, use the Handshake Function HSRenderSkinAsynch to render the skin with the computed properties.  The skin element will be rendered as a child to the target's parent element, replacing the original .lazy-skin-load div that was setup in the embedded html.

{% include codeheader.html %}
```js
HSRenderSkinAsync(webService, kendo.htmlEncode(properties), element, function () {
    _hsEvalScripts(element);      
});            
```

* * *

That's it. Thanks for reading, hope you find this technique useful. 
