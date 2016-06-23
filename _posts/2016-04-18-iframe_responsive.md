---
layout: post
title: "How to keep an iFrame ratio in a responsive context?"
description:
tags: [responsive, responsive, mobile, front-end, javascript, css]
categories: [responsive, front-end]
---

During the development of our new learning path technology, we faced some issues with responsiveness.
One of them was to make a responsive iFrame containing a video.

The main challenge was to make the iFrame responsive by keeping its ratio, so we avoid the black bands around the video at resize.

## The easy dirty way ##

Our first approach was to put the iFrame `.crossPlayer-iframe` inside of a DIV `.crossplayer-wrapper`, and set width and height of our iFrame to 100% (so it fits its container)

Now we have to give a size to our container. Remember: we want a responsive behavior.
So let's set to `.crossplayer-wrapper` a width of 100%, and an auto height so it will be sized with our iFrame content.

##### HTML: #####

    <div class="crossPlayer-wrapper">
        <iframe class="crossPlayer-iframe">
            ...
        </iframe>
    </div>

##### CSS: #####

    .crossPlayer-wrapper {
      width: 100%;
      height: auto;
    }

    .crossPlayer-iframe {
      width: 100%;
      height: 100%;
    }

Hoho... By doing this, the height of `.crossplayer-wrapper` is not modified once the iFrame content is loaded...

Okay, let's add a bit of javascript to compute the height with a 16/9 of the width!

> Wait... I will have compute the height every time the window is resized if I want to keep the video ratio?!

See the problem? Let's think to a better way!

## The responsive way ##

Back to the start. We have our `.crossPlayer-wrapper` and our `.crossPlayer-iframe`.

What we need to make `.crossPlayer-iframe` responsive and keep its ratio, it's to have `.crossPlayer-wrapper` perfectly sized at the loading and to keep its ratio on resize without any javascript.

The solution is to set a placeholder in `.crossPlayer-wrapper` that will have the same ratio properties

Hey! We have a native HTML element which does the job: **an image!** Let's try with it!

##### HTML: #####

    <div class="crossPlayer-wrapper">
        <img class="crossPlayer-placeholder" src="data:image/gif;base64,R0lGODlhEAAJAIAAAP///wAAACH5BAEAAAAALAAAAAAQAAkAAAIKhI+py+0Po5yUFQA7" />
        <iframe class="crossPlayer-iframe">
            ...
        </iframe>
    </div>
> Please note that the img src attribute contains the data of a transparent image with a 16/9 ratio.

##### CSS: #####

    .crossPlayer-wrapper {
      position: relative;
    }

    .crossPlayer-placeholder {
      display: block;
      width: 100%;
      height: auto;
    }

    .crossPlayer-iframe {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    }

Here we are! The `.crossPlayer-wrapper` will fit the image size (which keeps its ratio) and the iFrame, set in `position: absolute;` is placed hover the image and take all the space given by `.crossPlayer-wrapper`.

Moreover, we don't use any particular JS or CSS trick, so this solution works even with old navigators such as IE8