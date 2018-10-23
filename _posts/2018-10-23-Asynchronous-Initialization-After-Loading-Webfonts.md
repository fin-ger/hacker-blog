---
title: Asynchronous Initialization After Loading Webfonts
published: true
---

Yesterday I was facing the problem of poorly rendered custom fonts in [xterm.js](https://xtermjs.org). The fonts get loaded via the `@font-face` CSS directive. As this is asynchronous it *can* happen that the terminal gets initialized before the font is fully loaded and available for use.

![xterm broken fonts](assets/2018-10-23-xterm-broken.png)

The above image shows the wrong spacing of characters which is a result of the incomplete font during initialization of the terminal.

I found the [xterm-webfont](https://github.com/CoderPad/xterm-webfont) plugin which handles this issue very well. The plugin uses the `FontFaceObserver` which can monitor the availability of custom fonts for websites.

This is actually quite a handy library which is useful for a whole bunch of other issues with fonts in webpages. It is definitely a library worth looking at!

After including the `xterm.js` plugin the terminal gets initialized after all fonts that are needed for operation are available. It just works :D

![xterm fixed](assets/2018-10-23-xterm-fixed.png)
