---
title: Focus in a textarea
date: 2012-03-30 14:09:32
layout: post
---
Hi everybody,

this time, i had a problem with a textarea. I wanted to be able to focus on a precise position and next, be able to add content exactly at this position.<!--more-->

It seems to be very simple but the solution is not. Let's see the code (jQuery) :
<pre lang="javascript" colla="+">
$.fn.extend({
	focusOnPos: function(pos) {
		// "this" in a jQuery method is an array of elements
		$.each(this, function(i, obj) { // this === obj
			if (this.createTextRange) { // IE
				var range = this.createTextRange();
				range.move("character", pos); // Move to pos and select (actually select nothing but that's the way...)
				range.select();
			} else if (this.setSelectionRange) { // FF/Chrome
				this.focus();
				this.setSelectionRange(pos, pos); // Empty range but again, that's the way
			} else { // Default
				this.focus(); // We don't know how to do so... focus at the end...
			}
		});
	},
	insertOnFocus: function(content) {
		$.each(this, function(i, obj) {
			if (document.selection) { // IE
				sel = document.selection.createRange();
				sel.text = content; // Replace the content selected by what we want. Strange behaviour...
			} else if (this.selectionStart !== undefined) { // FF/Chrome
				var pos = this.selectionStart;
				this.value = this.value.substring(0, pos) + content + this.value.substring(pos); // select before and after and insert between
			} else { // Default
				this.value += content; // Again, we don't know what to do so add at the end
			}
		});
	}
});
</pre>

Now that methods are defined, let's have a test with a simple JS function
<pre lang="javascript" colla="+">
function showTextarea() {
	var rawContent = 'a § b'; // Simple string with a specific marker (here "§")
	var rawCursorPosition = rawContent.indexOf('§'); // Find the position of the marker (future position of the cursor)
	rawContent = rawContent.replace('§', ''); // Now that we have the position, remove the marker

	if ($('#textarea').is(':visible')) {
		$('#textarea').hide();
	} else {
		$('#textarea').val(rawContent).show().focusOnPos(rawCursorPosition);

		// Just in order to see the change
		setTimeout(function() {
			$('#textarea').insertOnFocus('c');
		}, 2000);
	}
}
</pre>

And a simple HTML
<pre lang="html" colla="+">
<a href="#" style="display: block;" onclick="showTextarea(); return false;">Show/Hide</a>
<textarea id="textarea" style="display: none"></textarea>
</pre>

Let's have an example : <a href="http://www.korko.fr/sources/examples/focus-in-a-textarea.html">Try now</a>