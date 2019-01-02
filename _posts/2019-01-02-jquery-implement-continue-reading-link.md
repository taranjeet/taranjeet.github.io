---
title: "Jquery &#58; Implement Continue reading link"
layout: post
date: 2019-01-02 18:00
tag:
- jquery
- javascript
category: blog
author: taranjeet
description: This post is about implementing a continue reading link
---

This post is about implementing a continue reading link. It will use jquery, but it can be done using javascript as well.

"Continue reading" link is needed when we need a div of fixed size and the height of the content is more than height of the div. To implement, this we can have two div, one in which the content is there(here with id `fixed-height`) and the other which will continue reading link and is currently hidden (here with id `fixed-height-footer`). When the page is loaded, get the height of the content div, and see if its more than threshold value(here `50px`).

The html and css code to create a div can be written as

```html
<div class="card">
    <div class="card-body" id="fixed-height">
        <p>
            This div contains so many paragraphs that the width of the content becomes more than the width of the div. This is going to be a very long paragraph.
            .....................
            .....................
        </p>
    </div>
    <div class="card-footer" id="fixed-height-footer" style="display:none">
    </div>
</div>
```

```css
<style type="text/css">
.fixed-height-div {
    max-height:450px;
    overflow:hidden;
}
</style>
```

The javascript code can be written as

```js
<script type='text/javascript'>
$(document).ready(function(){

    var fixedDiv = $('#fixed-height');
    var fixedDivHeight = $(fixedDiv).height();

    if (fixedDivHeight > 450){
        $(fixedDiv).addClass('fixed-height-div');
        $('#fixed-height-footer')
        .html('<a class="view-fixed-height">Continue Reading</a>')
        .show();
    }

    $('.view-fixed-height').on('click', function(){
        $(fixedDiv).removeClass('fixed-height-div');
        $('#fixed-height-footer').hide();
    });

});
</script>
```

The complete html code will look like


```
<html lang="en">
<head>
    <style type="text/css">
        .fixed-height-div {
            max-height:50px;
            overflow:hidden;
        }
        </style>
</head>

<body>
    <div class="card">
        <div class="card-body" id="fixed-height">
            <p>
                This div contains so many paragraphs that the width of the content becomes more than the width of the div. This is going to be a very long paragraph.
                .....................
                .....................
            </p>
        </div>
        <div class="card-footer" id="fixed-height-footer" style="display:none">
        </div>
    </div>

    <script src="https://code.jquery.com/jquery-3.3.1.min.js" integrity="sha256-FgpCb/KJQlLNfOu91ta32o/NMZxltwRo8QtmkMRdAu8=" crossorigin="anonymous"></script>
    <script type='text/javascript'>
        $(document).ready(function(){

            var fixedDiv = $('#fixed-height');
            var fixedDivHeight = $(fixedDiv).height();

            if (fixedDivHeight > 50){
                $(fixedDiv).addClass('fixed-height-div');
                $('#fixed-height-footer')
                .html('<a class="view-fixed-height">Continue Reading</a>')
                .show();
            }

            $('.view-fixed-height').on('click', function(){
                $(fixedDiv).removeClass('fixed-height-div');
                $('#fixed-height-footer').hide();
            });

        });
</script>
</body>
</html>

```
### Summary

This post used jquery to implement continue reading functionality.
