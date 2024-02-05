---
title: "Injecting a tab action in an HTML5TabStrip widget"
date: 2024-02-05
tags: HTML5TabStrip JavaScript
---

## Summary
The HTML5TabStrip component in Handshake is a powerful tool for creating and managing tabs with different types of content. It uses the [Kendo TabStrip](https://docs.telerik.com/kendo-ui/controls/tabstrip/overview) component to render the tabs and their content, which can be static or dynamic. The **tabrepeater** element allows you to create tabs dynamically based on some data source.

However, not all tabs need to show content. Sometimes, you may want a tab to perform an action instead, such as navigating to another page or running some JavaScript code. For this purpose, you can use the code below to generate an **actiontab** element, which lets you specify the action to be executed when the tab is clicked. The action can be a URL or a JavaScript function.

The challenge is the default behavior for a Kendo TabStrip tab is to display the .k-content block associated with that tab, while hiding the .k-content block associated with any other tab. That is not what we want to do in this use case. 

## Solution
As with most challenges we can overcome this one with a little JavaScript. 

The HTMLTabStrip provides several properties to execute code associated with tab events. They are:

- **OnInitialize** - execute code before the TabStrip is created. 
- **OnLoad** - execute code when the TabStrip is first loaded.
- **OnActivateTab** - execute code every time a tab is activated, including the when first rendered. 

The one we what for our purpose is **OnLoad**.  When the OnLoad code executes there are several key variables in scope:

- **tabStripId**: this will be the HS generated ID of the TabStrip
- **element**: the jQuery reference to the TabStrip DOM element, i.e. ```jQuery('#' + tabStripId);```
- **tabStripName** = the name you provided in the Name property of the TabStrip.

Create a function similar to the one below in your skin, where will we pass in the element jQuery object.

{% include codeheader.html %}
```js
function addActionTab(element) {
    const bing = 'https://www.bing.com';
    let tab = jQuery(`<li class="k-item hs-actiontab">
                    <div class="k-link">Open Bing Search</div>
                  </li>`);
    element.find('ul.k-tabstrip-items').append(tab);
    tab.on('click', (e) => {
        window.open(bing, "_blank");
        return false;
    })
}
```

Nothing overly complex here.  We create a jQuery element with styling simpler to normal tab elements. We then append it to the list of k-tabstrip-items. And finally set an eventListener to the click event of the new tab. It is in the click event we execute the code we want. 

Important: return false in this event listener so that the tab is not actually activated and thereby showing an empty content container. 

To execute this code add this function to the onLoad property of the TabStrip. 
```xml
<HTML5TabStrip name="testtab" onload="addActionTab(element);">
```

All that is left is some cosmetic details. I will usually want this special action tab to be separated from the normal content tabs, so this css will move that one tab element flush right in the container.
```css
.k-tabstrip .k-tabstrip-items li.k-item.hs-actiontab {
    margin-left:auto;
}
```

## Example Screen Shot
![sample tab](/assets/images/sample-inject-tab.png)

## More advanced functionality
Suppose we didn't just want to open Bing but pass it some value we had on this particular page.  For example if this tab was being rendered on a client page we might have **clientname** being provided as an intake property.  In this case, we can retrieve the value of client name and add that to the bing url.

{% include codeheader.html %}
```js
function addActionTab(element) {
    let bing = 'https://www.bing.com/search?q=';
    let tab = jQuery(`<li class="k-item hs-actiontab">
                    <div class="k-link">Open Bing Search</div>
                  </li>`);
    element.find('ul.k-tabstrip-items').append(tab);
    tab.on('click', (e) => {
        let elem = e.currentTarget; // this will be the DOM Element
        let clientName = HSGetProperty(elem, 'clientname');
        bing += encodeURIComponent(clientName);
        
        // or, if you prefer terse code. 
        // bing += encodeURIComponent(HSGetProperty(e.currentTarget, 'clientname'))

        window.open(bing, "_blank");
        return false;
    })
}
```