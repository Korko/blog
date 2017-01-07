---
title: 'CleverCover &#8211; How to dress up your Facebook Timeline'
date: 2012-03-26 14:17:57
layout: post
---
<blockquote style="background-color: #eee; border: 2px solid #000;text-align: center;">A new version of CleverCover is out, be sure to check the <a href="http://blog.korko.fr/2012/05/10/clevercover-how-to-dress-up-your-facebook-timeline-v2/" title="CleverCover – How to dress up your Facebook Timeline (V2)">article</a>!</blockquote>

<h3>No, I did not forget you!</h3>
<p style="text-align: justify;">Hi everybody! Even if some people think so, I did not forget you!
I have to admit, I had ups and downs and lots of projects that do not progress.</p>
<p style="text-align: justify;">Happily for you, I had time to make a little script to dress up you Facebook profile with the new design : the Timeline.
For those who don't know what it is : <a href="https://www.facebook.com/about/timeline" title="Facebook - About Timeline">https://www.facebook.com/about/timeline</a>.</p>
<p style="text-align: justify;">As you can see, there's a big picture in the background (called Cover) and your profile picture.</p>

<h3>Clever Cover</h3>
<p style="text-align: justify;">My script will allow you to cut a big image in two blocs that will perfectly fit.</p>
<p style="text-align: justify;">Here is an example with my profile : </p>
<a href="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0058.png"><img class="alignnone size-medium wp-image-170" title="Clever Cover : Result example" src="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0058-300x124.png" alt="" width="300" height="124" /></a><!--more-->

<h3>Where to find it? How to use it?</h3>
<p style="text-align: justify;">Well, first, where to find this tool? <a title="Clever Cover" href="http://www.korko.fr/clevercover/">Clever Cover</a></p>
<p style="text-align: justify;">Next, to use it, it's simple :
<ol>
<li>First, enter the url of the image you want to use in the text field on the top left<br /><a href="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0150.png"><img src="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0150-150x150.png" alt="" title="2012-01-27_0150" width="150" height="150" class="alignnone size-thumbnail wp-image-183" /></a></li>
<li>Click on "Generate". This step may take a while (~5/10s), this depends on your image.</li>
<li>You can resize the picture by using the little horizontal scroll bar :<br /><a href="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0201.png"><img src="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0201-150x150.png" alt="" title="Clever Cover : Step 2" width="150" height="150" class="alignnone size-thumbnail wp-image-184" /></a></li>
<li>You can move the image as if you were moving a file (drag/drop)</li>
<li>Once the size and the place done, click on "Extract"</li>
<li>You now only have to save the pictures displayed. (right click => "Save as") and to use them in your Facebook profile :<br /><a href="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0203.png"><img src="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0203-150x150.png" alt="" title="Clever Cover : Step 3" width="150" height="150" class="alignnone size-thumbnail wp-image-185" /></a></li>
</ol>

<h3>A little bit of code</h3>
<p style="text-align: justify;">After all, we are here for that so don't deprive ourselves.</p>
<p style="text-align: justify;">This little script allowed me to learn to use Canvas in HTML5. It was only developed for Chrome, but I suppose it must work for Firefox too. I will put here the code but in a more commented version than the online one in order to be more precise.</p>

<p style="text-align: justify;">Let's begin with the site heart : the Javascript. 99% of the logic is in here so you may fasten your seat belt.</p>
<pre lang="javascript" colla="+">
/**
 * This script can generate from a global picture two parts in order to be used as cover and profile picture in Facebook.
 * ----------------------------------------------------------------------------
 * "THE BEER-WARE LICENSE" (Revision 42):
 * <jeremy.lemesle@korko.fr> wrote this file. As long as you retain this notice you
 * can do whatever you want with this stuff. If we meet some day, and you think
 * this stuff is worth it, you can buy me a beer in return. Jeremy Lemesle
 * ----------------------------------------------------------------------------
 */

/**
 * Here we find a lot of Helpers so methods without real interest
 * except that they shortcut the coding time
 */

// Redefine of <myfunction>.bind, very useful to keep the scope and the <myelement>.addEventListener not defined in IE.
// Totally useless as this script is not working in IE and so essential.
if(!Function.prototype.bind)Function.prototype.bind=function(binding){return function(){this.apply(binding,arguments);};};
if(!HTMLElement.prototype.addEventListener)HTMLElement.prototype.addEventListener=function(eventType,listener){this.attachEvent("on"+eventType,function(e){e=e||window.event;listener(e);});};

// A little helper to create an HTML element fast with a parameters list and the classic $ for getElementById
function c(t,o){var d=document.createElement(t);o=o||{};for(var k in o)if(k === 'attributes')for(var a in o[k])d.setAttribute(a,o[k][a]);else d[k]=o[k];return d;}
function $(i){return document.getElementById(i);}

// Useful for the callbacks in order to do (callback || noop)() instead of a painful if(callback) { callback(); }
function noop(){}

// Load a picture from its URL and call a function when it's loaded (with metadata)
function loadImg(u,c){var i=new Image();i.src=u;$('hidden').appendChild(i);i.onload=c;return i;}

// A round to n instead of 1 and a little min/max to assert bounds of a value
function r(v,s){s=s||1;return Math.ceil(v/s)*s;}
function mm(mi,v,ma){return Math.max(mi,Math.min(v,ma));}

/**
 * Here is THE class that manage all the display for the cover and the picture
 * At the beginning, both were in the same <canvas> but sadly
 * it's not possible to extract only a part of a canvas as an image
 * so here is 2 different canvas : one for the cover and one for the picture.
 */
var Cover = function() {

	var COVER_WIDTH = 850, // This numbers come from a CSS analyze of Facebook
		COVER_HEIGHT = 313,
		FULL_HEIGHT = 355,
		img = null,
		x = 0,
		y = 0,
		ratio = 0;

	// Again, two little Helper functions to get a canvas and its context
	function cv(c){return $('canvas_'+c);}
	function ctx(c){return cv(c).getContext('2d');}

	this.draw = function() {
		if (img === null) return;

		drawCover();
		drawPicture();
	};

	// We get the rectangle with origin (x,y) (negative values so -x,-y)
	// This rectangle has COVER_WIDTH/ratio as with and COVER_HEIGHT/ratio as height
	// This ratio as you will see is a percent of the raw image.
	// We copy this rectangle in the canvas in (0,0) and with sizes, the one from the canvas.
	function drawCover() {
		ctx('cover').drawImage(img, Math.max(0, -x), Math.max(0, -y), COVER_WIDTH/ratio, COVER_HEIGHT/ratio, 0, 0, COVER_WIDTH, COVER_HEIGHT);
	}

	// This time, we have origin positions that are given for the canvas sizes.
	// That imply, we have to mix them with the ratio. So, the square position in the raw image is (32/ratio - x, 230/ratio - y)
	// Again this x/y that allow us to move the image inside the canvas
	// The size displayed in Facebook is 125px square so change once again the ratio.
	// Position inside the arrival canvas and size are not to explain.
	function drawPicture() {
		ctx('picture').drawImage(img, 32/ratio - x, 230/ratio - y, 125/ratio, 125/ratio, 0, 0, 125, 125);
	}

	// Change the canvas picture
	this.setImage = function(url, callback) {
		img = loadImg(url, function() {
			if (img.naturalWidth < COVER_WIDTH || img.naturalHeight < FULL_HEIGHT) {
				alert('This picture is too small. Need at least '+COVER_WIDTH+'x'+FULL_HEIGHT+'pixels.');
			} else {
				x = 0;
				y = 0;

				this.changeRatio(0);
				this.draw();
			}

			(callback || noop)();
		}.bind(this));
	};

	// Change the display ratio. Give a value between 0 and 100.
	this.changeRatio = function(newRatio) {
		newRatio = mm(0, newRatio, 100);

		// In the end, this value [0,100] have to be transformed.
		// In fact, the shortest value is the one that display the whole picture (exactly the one of the cover)
		var min = COVER_WIDTH / img.naturalWidth * 100;
		ratio = (min + newRatio * (100 - min)/100)/100; // ratio est compris dans [min, 100]

		// If the image is moved, reducing have to imply moving.
		if (COVER_WIDTH/ratio - x > img.naturalWidth) {
			x += COVER_WIDTH/ratio - x - img.naturalWidth;
		}
		if (FULL_HEIGHT/ratio - y > img.naturalHeight) {
			y += FULL_HEIGHT/ratio - y - img.naturalHeight;
		}
		this.draw();
	};

	this.save = function() {
		// Cover picture is in fact a base64 string
		// e.g. data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAALQAAAC0CAYAAAA9zQYyAAAgA...
		// Allowed by the canvas if the source of the picture is safe (same domain) hence the php file "img.php" next.
		$('output_cover').setAttribute('src', cv('cover').toDataURL());

		// As much as the cover is simple, at Facebook, we don't do things the easy way.
		// The picture profile is displayed in 125x125 in the Timeline but it has to be at least 180x180 so...
		// Resize before export
		var savePicture = c('canvas', {
			attributes: {
				width: 180,
				height: 180
			}
		});

		savePicture.getContext('2d').drawImage(img, 32/ratio - x, 230/ratio - y, 125/ratio, 125/ratio, 0, 0, 180, 180);
		$('output_picture').setAttribute('src', savePicture.toDataURL());
		$('output').style.display = 'block';
	};

	(function init() {
		// Initialize the Drag/Drop of the canvas (in the "canvas" div)
		drag.init('canvas', function(data) {
			// If we try to move elements in "canvas", let's move in the end the canvas' background image
			x = mm(COVER_WIDTH/ratio - img.naturalWidth, x + data.dx, 0);
			y = mm(FULL_HEIGHT/ratio - img.naturalHeight, y + data.dy, 0);
			this.draw();
		}.bind(this));
	}.bind(this))();
};

window.onload = function() {
	var cover = new Cover();

	$('action_save').addEventListener('click', function() {
		cover.save();
	});
	$('action_generate').addEventListener('click', function() {
		$('loading').style.display = 'inline';

		// Little trick to allow the save : pass by a php script that display the picture as if it's in the same server
		// Else it's not possible to make a toDataURL on a canvas that displays an external picture.
		cover.setImage('img.php?src='+$('url').value, function() {
			$('loading').style.display = 'none';

			// Initialize the scroll bar
			Scroller.bind('ratio', 100, 1, function(x) {
				cover.changeRatio(x);
			});
		});
	});
};

/**
 * Make an element draggable
 * @param string id Id of the div
 * @param fn callback Callback called each time element is dragged with 4 params:
 *		{x: <current mouse x>, y: <current mouse y>, dx: <x delta since last call>, dy: <y delta since last call>}
 */
var drag = (function() {
	var draggables = {};
	var dragged = null;
	var mousePos = null;

	document.addEventListener('mouseup', function() {
		dragged = null;
		mousePos = null;
	});
	document.addEventListener('mousemove', function(event) {
		if (!dragged) return;

		// Chrome use event.x/event.y
		// Firefox use event.clientX/event.clientY
		var newPos = {
			x: event.x || event.clientX,
			y: event.y || event.clientY
		};

		if (mousePos) {
			(draggables[dragged] || noop)({
				x: newPos.x,
				y: newPos.y,
				dx: newPos.x - mousePos.x,
				dy: newPos.y - mousePos.y
			});
		}
		mousePos = newPos;
	});

	return {
		init: function(id, callback) {
			draggables[id] = callback;

			$(id).addEventListener('mousedown', function() {
				dragged = id;
			});
		},
		del: function(id) {
			delete draggables[id];
			$(id).removeEventListener('mousedown');
		}
	};
})();

/**
 * Generate a scroller
 */
function Scroller() {
	var SCROLLER_WIDTH = 5;

	this.init = function(id, max, step, callback) {
		var maxWidth = max * SCROLLER_WIDTH/step; // Every SCROLLER_WIDTH, we gain 1 step until max

		$(id).className = $(id).className+' scroller';
		$(id).style.width = maxWidth + 'px';
		$(id).style.display = 'block';

		var scroller = c('div', {id: id+'_scroller'});
		$(id).appendChild(scroller);

		drag.init(id+'_scroller', function(data) {
			var old = parseInt(scroller.style.left, 10) || 0;

			var value = mm(0, old + data.dx, maxWidth);
			scroller.style.left = Math.min(value, maxWidth - SCROLLER_WIDTH) + 'px';
			callback(r(value/SCROLLER_WIDTH, step));
		}.bind(this));
	};

	this.unbind = function(id) {
		drag.del(id+'_scroller');
		$(id).removeChild($(id+'_scroller'));
	};
}
Scroller.bind = (function() {
	var binded = {};

	return function(id) {
		if (binded[id]) {
			binded[id].unbind(id);
		}
		binded[id] = new Scroller();
		binded[id].init.apply(binded[id], arguments);
	};
})();
</pre>

I don't think HTML is very interesting. On the other hand, the PHP script which display the remote picture as if it's local...
<pre lang="php" colla="+">
<?php

ob_start("ob_gzhandler");

function loadImage ($file) {
    $data = getimagesize($file);
    switch($data["mime"]){
        case "image/jpeg":
            $im = imagecreatefromjpeg($file);
            break;
        case "image/png":
            $im = imagecreatefrompng($file);
            break;
        default:
            throw new Exception('Img Type not managed');
    }
    return $im;
}

if (!isset($_GET['src'])) {
    die('USAGE: img.php?src=&lt;imageURL&gt;');
}

$src = $_GET['src'];

try {
    $image = loadImage($src);
} catch(Exception $e) {
    die('Unable to load this image');
}

$w = imagesx($image);
$h = imagesy($image);

$im2 = ImageCreateTrueColor($w, $h);
imagecopyResampled ($im2, $image, 0, 0, 0, 0, $w, $h, $w, $h);

$time = @filemtime($src);
if ($time === null) {
	$time = time();
}

header('Content-type: image/png');
header("Content-Disposition: inline; filename=".basename($src).".png");
header('Last-Modified: ' . gmdate('D, d M Y H:i:s', $time) . ' GMT');
header("Cache-Control: public");
header("Pragma: public");
imagepng($im2);
imagedestroy($im2);
imagedestroy($image);
</pre>

<h3>Beerware</h3>
<p style="text-align: justify;">I had never told of that here before but the bigger part of my codes are under Beerware "licence". In shortcut, you can do whatever you want with it. There's no limitation. But if my work please you and you want me to know it, don't hesitate to pay me a bear next time we met.</p>

<h3>Sources</h3>
<p style="text-align: justify;">In order to download the whole sources : <a href='http://blog.korko.fr/wp-content/uploads/2012/01/clevercover.zip'>clevercover</a></p>