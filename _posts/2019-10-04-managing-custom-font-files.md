---
id: 1349
title: Managing Custom Font Files
date: 2019-10-04T06:00:01+00:00
guid: http://www.peoplesoftmods.com/?p=1349
permalink: /psadmin/managing-custom-font-files/
tags:
  - Web Server
  - CSS
  - Cache
categories:
  - PeopleSoft Administration
---

Design requirements often require PeopleSoft developers to use custom web fonts when styling PeopleSoft
applications.  An easy way to make use of a custom font is to link to a third-party server that hosts the
font files.


```css
@font-face {
  font-family: 'Mansalva';
  font-style: normal;
  src: url(https://fonts.gstatic.com/s/mansalva/v1/aWB4m0aacbtDfvq5NKliKY8.woff2) format('woff2');
}

.ps-label {
 font-family: 'Mansalva', cursive;
}
```


However, this method causes an undesirable dependency on the third-party server and as well as might not
be an option with local firewall constraints.

### An Alternative

Another option to access custom fonts from PeopleSoft application CSS is to host the custom web font on the
PeopleSoft web server.  This requires the web font file(s) to be placed in an accessible directory (typically
the `fonts` directory) on the server to be consumed from the application CSS.


```css
@font-face {
  font-family: 'Mansalva';
  font-style: normal;
  src: url(../fonts/aWB4m0aacbtDfvq5NKliKY8.woff2) format('woff2');
}

.ps-label {
 font-family: 'Mansalva', cursive;
}
```


This option works well, but managing custom font files on the web server can be problematic from a lifecycle
management perspective as the fonts are not managed by PeopleTools.  The self-hosting of font files requires a
different migration workflow versus the standard App Designer migration.  This variation of object management
causes more moving parts and higher chances for mistakes to occur when performing migrations and installations
of the custom application code.

### A Solution

A technique to avoid having to manage custom font files on the web server is to store the font files in Image
objects within App Designer.  These Image objects can be referenced in the application CSS with the use of
the `%Image` meta-function.


```css
@font-face {
  font-family: 'Mansalva';
  font-style: normal;
  src: url(%Image(PSM_MY_CUSTOM_FONT)) format('woff2');
}

.ps-label {
 font-family: 'Mansalva', cursive;
}
```


The `%Image` meta-function will resolve to a URL of the font file stored in the web server cache directory:

[0]: /assets/images/2019/10/CachedFontFile.png
[![Cached Font File][0]][0]

This technique allows for the custom font file to be served from the web server without having to manually
manage the file on the server.  Additionally, this technique allows App Designer to manage/migrate the custom
font to ease lifecycle management.