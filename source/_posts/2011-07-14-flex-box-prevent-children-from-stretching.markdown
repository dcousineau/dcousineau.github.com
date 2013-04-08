---
author: admin
comments: true
date: 2011-07-14 15:47:57
layout: post
slug: flex-box-prevent-children-from-stretching
title: 'Flex Box: Prevent Children From Stretching'
wordpress_id: 497
categories:
- Development
tags:
- columns
- css3
- flex box
- moz-box-flex
- webkit-box-flex
---

So, [flex boxes](http://www.the-haystack.com/2010/01/23/css3-flexbox-part-1/) are a glorious addition to CSS3 to make advanced layouts rapidly. No more crazy floats and nested divs and weird percentage values for columns! Just setup your orientation to horizontal and toss a `box-flex: 1` in for kicks.

However, just a tip: In the course of working with flex boxes, you will find that if you given a parent container a flex value, the children can still stretch it out. For example, a long paragraph will push its parent wider and wider to accommodate unless you give it a static width (which kinda defeats the purpose of using flex boxes really).

A nifty trick, however, is if you give an element with a defined `box-flex` a `width: 0px;`, the sizing algorithm will ignore said element's children when sizing the element.

Here's an example of automatic vertical columns:


```html

<div id="wall">
    <div id="col1" class="column top">
        
        <div class="entry">
            Lorem ipsum dolor sit amet, consectetur adipiscing elit. Aliquam eget tristique velit. In ut ligula nibh, a pulvinar lorem. Nam sed elit eget tellus vestibulum
        </div>
        
    </div>
    <div id="col2" class="column bottom">
        
        <div class="entry">
            Lorem ipsum dolor sit amet, consectetur adipiscing elit. Aliquam eget tristique velit. In ut ligula nibh, a pulvinar lorem. Nam sed elit eget tellus vestibulum
        </div>
        
    </div>
    <div id="col3" class="column top"></div>
    <div id="col4" class="column bottom"></div>
</div>

```



```css

#wall {
    display: -webkit-box;
    -webkit-box-orient: horizontal;
    
    position: absolute !important;
    top:0; 
    right:0; 
    bottom:0; 
    left:0;
    
    overflow: hidden;
}

#wall .column {
    display: -webkit-box;
    -webkit-box-sizing: border-box;
    
    -webkit-box-orient: vertical;
    
    -webkit-box-flex: 1;
    width:0px;
    
    border-right: 1px solid #333;
}

```



