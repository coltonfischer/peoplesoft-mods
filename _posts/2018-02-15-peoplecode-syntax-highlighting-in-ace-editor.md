---
id: 1280
title: PeopleCode Syntax Highlighting in Ace Editor
date: 2018-02-15T18:48:18+00:00
guid: http://www.peoplesoftmods.com/?p=1280
permalink: /tips-and-tricks/peoplecode-syntax-highlighting-in-ace-editor/
tags:
  - PeopleCode
  - JavaScript
  - Metadata API
categories:
  - Tips and Tricks
  - Utilities
---

Ace is an embeddable code editor written in JavaScript. The [Ace Editor](https://ace.c9.io/) allows for practical software development in online tools such as the [Cloud9 IDE](https://c9.io). 
With tools like Cloud9, you can develop software from any one of your internet-connected devices. PeopleSoft development is a bit different than developing software in other languages, however, as App 
Designer is needed to edit PeopleCode programs. I would rather not have to always rely on a client-based application to edit PeopleCode. This is the reason that I embarked on writing a JavaScript-based 
PeopleCode editor powered by Ace. The Ace Editor provides many desirable features that can be found in most modern editors and it also allows for language-specific syntax highlighting. Today I would 
like to share the PeopleCode syntax mode that I created for the Ace Editor.

Usage of the PeopleCode syntax highlighter in the Ace Editor is really simple. A few lines of JavaScript are needed to target an HTML element containing the PeopleCode to highlight:

```js
var editor = ace.edit("editor");
var JavaScriptMode = ace.require("ace/mode/people_code").Mode;
editor.session.setMode(new JavaScriptMode());
editor.setOption("showPrintMargin", false);
editor.getSession().setTabSize(3);
```

This will apply the syntax highlighting rules to the PeopleCode string within the HTML element when the page is rendered. You can check out [this JS Fiddle](https://jsfiddle.net/coltonfischer/fr5h63ha/) 
to see the highlighter in action. The syntax highlighter is not perfect, but it is definitely a step up from no highlighting at all. If you are interested in the implementation of the highlighting rules, then you can 
view the [source code on GitHub](https://github.com/coltonfischer/PeopleCode-Ace-Editor).

While PeopleCode syntax highlighting with the Ace Editor is helpful for displaying read-only PeopleCode snippets online, this functionality really becomes useful when used in an online PeopleCode editor 
that is capable of making real time code updates to the application. I did a [post last year](/tips-and-tricks/online-peoplecode-editor-project/) discussion the idea of 
exposing the %Metadata Application Package as an API of sorts to support the backend of an online PeopleCode editor. While this was a solid proof of concept, the demonstrated editor in that post was 
undoubtedly hideous. Since then I have added a slew of changes with the most important one being the incorporation of the Ace Editor. Here is a screen shot of the progress that has been made to the 
online PeopleCode editor project:

[1]: /assets/images/2018/02/Online_PeopleCode_Editor.png
[![Online PeopleCode Editor][1]][1]

As you can probably tell, the Ace Editor with PeopleCode syntax highlighting is a step in the right direction in making a usable online PeopleCode editor. I hope to to share and document the new 
features and functionality of the online PeopleCode editor project in a future post.