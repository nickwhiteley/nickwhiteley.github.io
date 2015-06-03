---
layout: post
title:  "Client side preview uploaded video files"
date:   2015-06-02 14:08:57
categories: Javascript HTML5
---

When someone uploads a video file it is nice to show them a thubmnail to they can be sure it's the right file being sent.  The usual way to do this is to send the file to the server and the server creates a new image based on the video.   It is possible to do this entirely on the client with an HTML5 compatible browser.  

It's possible but should you do this?  Probably not.  Processor usage goes through the roof when trying to do this and because the Javascript is running on the main UI thread the page becomes unresponsive. Still, it's an interesting excercise.

In order to make this work I am going to use three features introduced in HTML5.

The FileReader class can load files from disk.

The video tag can display them on screen.

The canvas tag can draw an image based on another tag.

Bring it all together and this is what you've got:

{% highlight javascript %}
function previewFile() {
    var thumbVidSrc = document.getElementById('thumbVidSrc');
    var thumbVid = document.getElementById('thumbVid');

    var thumbVidCanvas = document.getElementById('thumbVidCanvas');
    var file    = document.getElementById('videoFile').files[0];
    var reader  = new FileReader();

    reader.onloadend = function () {
        thumbVid.src = reader.result;
    };
    thumbVid.oncanplay = function () {
        thumbVidCanvas.getContext('2d').drawImage(thumbVid, 0, 0, thumbVid.width, thumbVid.height);
    };

    if (file) {
        reader.readAsDataURL(file);
    } else {
        thumbVid.src = "";
    }
}
document.getElementById('videoFile').onchange = previewFile;
{% endhighlight %}

{% highlight html %}
<style>.hide { display: none; }</style>
<input type="file" name="video" id="videoFile">
<canvas id="thumbVidCanvas" width="70" height="40" class="uploadContainer"></canvas>
<video id="thumbVid" src="" class="hide" width="70" height="40" controls=""></video>
{% endhighlight %}


