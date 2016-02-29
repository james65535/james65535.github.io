---
layout: article
title: Navigating Front End Wilderness
modified:
categories: Development
excerpt: 
tags: [Javascript, JSX, React, Babel, IDE]
image:
  feature: 
  teaser: "bar.jpg"
  thumb:
---

Recently I've set myself a task of developing a web based game in order to get myself up to speed on modern development patterns and technologies... something I haven't done since I worked with PHP over a decade ago.

For my application I wanted to work in a MVC pattern and to carve off the view layer as client side.  For this task I selected <a href="https://facebook.github.io/react/">React</a>.

This is a challenge for me as the minimal amount of Javascript I know has been used to create Gluecode for automation tasks.  I decided to get a little bit of help by working with online video tutorials to help guide me but ran into some trouble through the combination of my lack of knowledge and the speed of innovation with Javascript libraries and frameworks.  The most problems I had was with <a href="https://facebook.github.io/react/docs/jsx-in-depth.html">JSX</a>.

## JSX
Challenges:

* Lack of understanding about JSX in general
* Lack of understanding about transpiling
* Changes to the React framework since the creation of the video tutorials

This manifested it's self when I decided to take the small working app I had create in <a href="https://plnkr.co/">Plunker</a> and bring it into my <a href="https://www.jetbrains.com/idea/">IDE</a>.  All of a sudden when I browsed to the HTML file locally from Firefox I was getting none of the expected elements to display on the page.  I went into a mad fury researching the above elements pages, creating more tabs in my browser than stars in the sky, and by the end it was a quick solution.

The crux of the problem was simple.  Somewhere between the time the creation of the video tutorials and now, React had moved away from the JSX Transform tools and adopted <a href="https://babeljs.io/">Babel</a>.  Details on this move can be found <a href="https://facebook.github.io/react/blog/2015/06/12/deprecating-jstransform-and-react-tools.html">here</a>.

Turns out all I had to do was place a `<script></script>` reference in my HTML to link in Babel, example:

```<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.23/browser.min.js"></script>```

I feel like a fish out of water with this Web Front End work!
