---
title: 'jQuery&rsquo;s document.ready function ($(<fn>)) fired too early'
date: 2012-04-04 22:35:36
layout: post
---
In a current development, i had a bug with the jQuery document.ready function which was called before the css was applyed. This resulted in an unwanted behavior of $(<element>).show() was not working because jQuery thinked it was already displayed. But then the css came and hide the div so finally it was never displayed (or only after hard refresh). The best solution was to put the css declaration BEFORE the javascript's one. Easy but hard to find...