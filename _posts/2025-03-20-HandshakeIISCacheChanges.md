---
title: "Fixing Handshake Cache Issues"
date: 2025-03-20
tags: CSS JavaScript IISCache
---
## Summary
It is a common problem where you change a style or script file on your handshake server, publish it to production, and users do not receive the updated code. This happens because the browser is not instructed to update the cache when the file has changed since it was last cached.

Below are the steps to resolve this issue.

## Required Changes in IIS

In the IIS Manager of the Handshake Server, add an entry to Output Caching for `.css` and `.js` files to 'cache until change.'

![Output Caching Screenshot](/assets/images/ss1.png)

## Handshake Theme Changes
The update above will cover all files in the `HandshakeWebServices` root. However, the `HandshakeThemes` application does not work with this change due to how the Handshake Server handles those particular files.

To address this, we need to move the CSS files from the `Themes` folder so that they are hosted in the `HandshakeWebServices` application.

1. Create a new folder named `css` under `\Program Files\Handshake\HandshakeWebServices`.
2. Copy the current active `Themes` folder to the new `css` folder.
3. In the active `Themes` folder, leave a single CSS file that imports from the new CSS location:
4. 
    {% include codeheader.html %}
    `C:\Program Files\Handshake\HandshakeThemes\Themes\<yourtheme>\redirect.css`
    ```css
    @import "https://<yourhsroot>.<yourdomain>.com/css/<corecss>.min.css";
    ```

5. Run the Handshake Configuration Utility to load this `redirect.css` file in the Themes settings.
   ![Theme Changes Screenshot](/assets/images/ss2.png)

6. In IIS, add a new application under `HS_Site` named `css`, point it to `./HandshakeWebServices/css`, and:
    - Add `Access-Control-Allow-Origin=*` to the HTTP Response Headers. This will allow font files to load from this folder across domains.
    - Set Authentication to allow anonymous access

