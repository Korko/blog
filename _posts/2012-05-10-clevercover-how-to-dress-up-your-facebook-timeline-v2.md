---
title: 'CleverCover &#8211; How to dress up your Facebook Timeline (V2)'
date: 2012-05-10 10:56:01
layout: post
---
<h3>He's Alive, ALIVE!</h3>

<p>Finally, my new version of CleverCover is available for everyone : <a href="http://www.korko.fr/clevercover/" title="CleverCover" target="_blank">CleverCover</a>!</p>

<p>For those who don't know what CleverCover is, you can take a look at the first version here : <a href="http://blog.korko.fr/2012/03/26/clevercover-how-to-dress-up-your-facebook-timeline/" title="CleverCover – How to dress up your Facebook Timeline"></a><br />
It was not very userfriendly, not very modular etc. This time, it looks great. Here's some screen shots in order to explain how it works :</p>

<h2>Behavior</h2>

<p>Here's the new welcome screen (select the social network you want your cover for) :<br />
<a href="http://blog.korko.fr/wp-content/uploads/2012/05/CleverCover-Prepare-Chromium_020.png"><img src="http://blog.korko.fr/wp-content/uploads/2012/05/CleverCover-Prepare-Chromium_020-300x188.png" alt="" title="CleverCover v2 - 1 - Index" width="300" height="188" class="alignnone size-medium wp-image-312" /></a></p>

<p>And the selection of the image :<br />
<a href="http://blog.korko.fr/wp-content/uploads/2012/05/CleverCover-Prepare-Chromium_021.png"><img src="http://blog.korko.fr/wp-content/uploads/2012/05/CleverCover-Prepare-Chromium_021-300x188.png" alt="" title="CleverCover v2 - 2 - Image Choice" width="300" height="188" class="alignnone size-medium wp-image-313" /></a></p>

<p>Now that every thing is ok, click on "Generate your Cover" and Voila!<br />
<a href="http://blog.korko.fr/wp-content/uploads/2012/05/CleverCover-Chromium_023.png"><img src="http://blog.korko.fr/wp-content/uploads/2012/05/CleverCover-Chromium_023-300x188.png" alt="" title="CleverCover v2 - 3 - Cover" width="300" height="188" class="alignnone size-medium wp-image-310" /></a></p>

<p>In this page, you can resize the picture with the black scroll bar just below the cover, you can also flip your picture from left to right with the little check box, and do not forget to move by dragging the picture!</p>

<p>When finished, click the "Save" button in the top right corner and Abracadabra, here are your pictures!<br /><a href="http://blog.korko.fr/wp-content/uploads/2012/05/CleverCover-Chromium_024.png"><img src="http://blog.korko.fr/wp-content/uploads/2012/05/CleverCover-Chromium_024-300x188.png" alt="" title="CleverCoverv2 - 4 - Save" width="300" height="188" class="alignnone size-medium wp-image-311" /></a></p>

<p>You can now save both images (left is cover and right is avatar) and put them in your favorite social network.</p>

<p>Here's a list of what I intend to improve next in this script :
<ul>
	<li>Add one script for each social network that will automatically put both images in your profile without any action for you except the "Save" button"</li>

	<li>Allow to rotate the picture</li>

	<li>Allow to use 2 images instead of just one</li>

	<li>Allow to over resize the picture (actually you can only resize from the minimum of the required width/height to the exact size of the picture, not over)</li>

</ul>

<p>If you have ideas to improve it, do not hesitate to leave a comment bellow!</p>

<h2>Code</h2>

<p>Now let's talk about code!</p>

<p>First, the more important : The cover!</p>
<h3>JavaScript - ResizableCanvas class</h3>
<pre lang="javascript" colla="-">
// Javascript class to implement to add a new part of image
// Required : width/height/id
var ResizableCanvas = (function() {
	var poolEventId = 0;

	return function(params) {

		/*********************
		 *	 Variables	 *
		 ********************/
		this._id = params.id;
		this._width = params.width;
		this._height = params.height;
		this._extractWidth = params.extractWidth || params.width;
		this._extractHeight = params.extractHeight || params.height;

		this._links = {};
		this._fullWidth = params.width;
		this._fullHeight = params.height;
		this._left = 0;
		this._top = 0;
		this._right = 0;
		this._bottom = 0;

		this._img = null;
		this._x = 0;
		this._y = 0;
		this._ratio = 100;
		this._minRatio = 0;
		this._opacity = 1;
		this._flipFactor = 1;

		this._events = {};

		/*********************
		 *	  Methods	  *
		 ********************/

		// draw the canvas
		this.draw = function(propagate) {
			var _draw = function() {
				this._fixPosition();

				var context = ctx(this._id);
				context.clearRect(0, 0, this._fullWidth, this._fullHeight);
				context.save();
				context.globalAlpha = this._opacity;
				context.scale(this._ratio, this._ratio);

				if(this._flipFactor === -1) {
					context.translate(this._fullWidth/this._ratio, 0);
					context.scale(-1, 1);
				}
				context.translate(this._flipFactor*(-this._x-this._left)/this._ratio, (-this._y-this._top)/this._ratio);
				context.drawImage(this._img, 0, 0);

				context.restore();
			};

			if (propagate) {
				this._propagate(_draw, [], true);
			} else {
				_draw.apply(this);
			}
		};

		// init when dom's ready
		this._prepare = function() {
			$(this._id).setAttribute('width', this._width);
			$(this._id).setAttribute('height', this._height);
			$(this._id).style.width = this._width + 'px';
			$(this._id).style.height = this._weight + 'px';
		};
		jQuery(this._prepare.bind(this));

		// link to an other ResizableCanvas
		this.link = function(canvas, params) {
			if(this._links[canvas._id]) {
				return;
			}

			var shared = ["left", "top"];
			for(i in shared) {
				if(params[shared[i]] < 0) {
					canvas['_' + shared[i]] -= params[shared[i]];
				} else {
					this['_' + shared[i]] += params[shared[i]] || 0;
				}
			}

			this._links[canvas._id] = canvas;
			canvas._links[this._id] = this;

			// TODO: adapt to multiple links
			this._fullWidth = Math.max(this._width + Math.max(0, canvas._left + canvas._width - this._width), canvas._width + Math.max(0, this._left + this._width - canvas._width));
			this._fullHeight = Math.max(this._height + Math.max(0, canvas._top + canvas._height - this._height), canvas._height + Math.max(0, this._top + this._height - canvas._height));
			this._propagate({
				_fullWidth : this._fullWidth,
				_fullHeight : this._fullHeight
			});
			jQuery(this._prepare.bind(this));
		};

		// propagate a function or params firectly to linked RezisableCanvas
		this._propagate = function(event, params, execute) {
			var values = [];
			if(!event.id) {
				event.id = ++poolEventId;
			} else if(this._events[event.id]) {
				return values;
				// Already done and forwarded
			} else {
				execute = true;
			}

			if(execute) {
				if(jQuery.isFunction(event)) {
					values.push(event.apply(this, params));
				} else {
					// Treat the event
					for(var index in event) {
						this[index] = event[index];
					}
				}
			}

			this._events[event.id] = event;

			// Forward
			for(var canvasId in this._links) {
				jQuery.merge(values, this._links[canvasId]._propagate(event, params));
			}
			return values;
		};

		// change canvas picture
		this.setImage = function(url, callback) {
			this._img = loadImg(url, function() {
				var success = true;
				if(this._img.naturalWidth < this._fullWidth || this._img.naturalHeight < this._fullHeight) {
					alert('This picture is too small. Need at least ' + this._fullWidth + 'x' + this._fullHeight + 'pixels (got ' + this._img.naturalWidth + 'x' + this._img.naturalHeight+').');
					success = false;
				} else {
					this._minRatio = Math.max(this._fullWidth / this._img.naturalWidth, this._fullHeight / this._img.naturalHeight) * 100;

					this._propagate({
						_img : this._img,
						_minRatio : this._minRatio
					});

					this.changeRatio(this._ratio);
				}(callback || noop)(success);
			}.bind(this), true);
		};

		// Move the picture
		this.move = function(data) {
			this._propagate(function() {
				this._x -= data.dx;
				this._y -= data.dy;
				this.draw(false);
			}, [], true);
		};

		// Correct position while moving/resizing (if at bottom-right corner, resize have to move the picture to the left)
		this._fixPosition = function() {
			this._x = mm(0, this._x, this._flipFactor * (this._img.naturalWidth - this._fullWidth / this._ratio) * this._ratio);
			this._y = mm(0, this._y, (this._img.naturalHeight - this._fullHeight / this._ratio) * this._ratio);
		};

		// get current ratio [0,100]
		this.getRatio = function() {
			return (this._ratio * 100 - this._minRatio) / (100 - this._minRatio) * 100;
		};

		// change current ratio
		this.changeRatio = function(newRatio) {
			newRatio = mm(0, newRatio, 100);

			this._propagate(function() {
				// Au final, cette valeur [0,100] doit être transformer.
				// En effet, la plus petite valeur est celle qui affiche l'image dans toute sa largeur (exactement celle de la couverture)
				this._ratio = this._minRatio + newRatio * (100 - this._minRatio) / 100;
				// ratio est compris dans [min, 100]
				this._ratio /= 100;

				this.draw();
			}, [], true);
		};

		// save the picture to a specific div (toId) and/or return HTML
		this.save = function(toId) {
			return this._propagate(function() {
				var saveCanvas = $(this._id);

				if(this._extractWidth !== this._width || this._extractHeight !== this._height) {
					saveCanvas = c('canvas', {
						attributes : {
							width : this._extractWidth,
							height : this._extractHeight
						}
					});

					var context = saveCanvas.getContext('2d');
					var ratio = this._ratio / (this._width / this._extractWidth);
					context.scale(ratio, ratio);
					context.drawImage(this._img, (-this._x-this._left)/this._ratio, (-this._y-this._top)/this._ratio);
				}

				// L'image de la couverture que nous pourrons sauvegarder est en fait une chaîne sous forme base64
				// e.g. data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAALQAAAC0CAYAAAA9zQYyAAAgA...
				// Fonctionnalité permise par un canvas si l'origine de l'image est sûre (même domaine) d'où le fichier php "img.php" plus bas
				var node = toId ? $(toId) : c('div');
				if(node.nodeName !== 'IMG') {
					var parent = node;
					node = c('img');
					node.style.height = '100px';
					parent.appendChild(node);
				}
				node.setAttribute('src', saveCanvas.toDataURL());

				return node.outerHTML;
			}, [], true);
		};

		// flip image left to right
		this.flip = function() {
			this._propagate(function() {
				this._flipFactor = this._flipFactor === -1 ? 1 : -1;
				this.draw();
			}, [], true);
		};
	};
})();
</pre>

<h3>JavaScript - CleverCover script</h3>
<pre lang="javascript" colla="-">
window.cleverCover = (function() {

	var params = {
		cover : {},
		avatar : {},
		link : {}
	};

	return {
		init : function(type, url_cover, url_avatar, callback) {
			var splited = !!url_avatar,
				slider_choice = 'cover',
				canvas = {};

			switch(type) {
				// Extracted data from Facebook/Google
				case 'google':
					params.cover = {
						width : 940,
						height : 180
					};
					params.avatar = {
						width : 250,
						height : 250
					};
					params.link = {
						left : 627,
						top : -39
					};
					break;

				case 'facebook':
					params.cover = {
						width : 850,
						height : 313
					};
					params.avatar = {
						width : 160,
						height : 160,
						extractWidth : 180,
						extractHeight : 180
					};
					params.link = {
						left : 31,
						top : 205
					};
					break;
			}

			canvas['cover'] = new ResizableCanvas(jQuery.extend({
				id : 'canvas_cover'
			}, params.cover));
			canvas['avatar'] = new ResizableCanvas(jQuery.extend({
				id : 'canvas_picture'
			}, params.avatar));

			// Hell yes, everything is almost ready for two distinct pictures
			if(!splited) {
				canvas['avatar'].link(canvas['cover'], params.link);
			}

			jQuery((function($) {
				return function() {
					$('#content').addClass(splited ? 'splited' : '');
					$('#content_inner').addClass(type);
					$('#save').click(function() {
						var content = canvas['cover'].save('popup_content').join('');
						if(splited) {
							content += canvas['avatar'].save('popup_content').join('');
						}
						popup.content('<p>Here\'s your avatar and your cover, just save them with right click and put them directly to your favorite social network.</p>' + content, true, 'cover_save');
					});
					$('input[name="cover_slider_choice"]').change(function() {
						slider_choice = $(this).val();
					});
				};
			})(jQuery));

			var manageSuccess = (function() {
				var globSuccess;
				return function(success) {
					if(!splited) {
						callback(success);
					} else {
						if(globSuccess !== undefined) {
							callback(globSuccess && success);
						} else {
							globSuccess = success;
						}
					}
				};
			})();

			var globSuccess;
			url_cover = 'img.php?src='+encodeURIComponent(url_cover);
			canvas['cover'].setImage(url_cover, function(success) {
				if(success) {
					jQuery('#canvas_cover').drag(function(data) {
						canvas['cover'].move(data);
					});
				}
				manageSuccess(success);
			});
			if(splited) {
				url_avatar = 'img.php?src='+encodeURIComponent(url_avatar);
				canvas['avatar'].setImage(url_avatar, function(success) {
					if(success) {
						jQuery('#canvas_picture').drag(function(data) {
							canvas['avatar'].move(data);
						}).hover(function() {
							canvas['avatar'].expand();
						}, function() {
							canvas['avatar'].reduce()
						});
					}
					manageSuccess(success);
				});
			}

			Scroller.bind('cover_slider', 100, 1, function(ratio) {
				canvas[slider_choice].changeRatio(ratio);
			});
			jQuery('#cover_flip input').change(function() {
				canvas[slider_choice].flip();
			});
		},
		help: function() {
			jQuery('#overlay').show();
		}
	};
})();
</pre>

<h3>JavaScript - Scroller (nothing changed since last version)</h3>
<pre lang="javascript" colla="-">
/**
 * Generate a scroller
 */

function Scroller() {
	var SCROLLER_WIDTH = 5;

	this.init = function(id, max, step, callback) {
		var maxWidth = max * SCROLLER_WIDTH / step;
		// Every SCROLLER_WIDTH, we gain 1 step until max

		$(id).className = $(id).className + ' scroller';
		$(id).style.width = maxWidth + 'px';
		$(id).style.display = 'inline-block';

		var scroller = c('div', {
			id : id + '_scroller',
			className : 'scroller_drag'
		});
		$(id).appendChild(scroller).style.left = maxWidth - SCROLLER_WIDTH + 'px';

		jQuery(scroller).drag(function(data) {
			var old = parseInt(scroller.style.left, 10) || 0;
			var value = mm(0, old + data.dx, maxWidth);
			scroller.style.left = Math.min(value, maxWidth - SCROLLER_WIDTH) + 'px';
			callback(r(value / SCROLLER_WIDTH, step));
		});
	};

	this.unbind = function(id) {
		jQuery(id + '_scroller').undrag();
		$(id).removeChild($(id + '_scroller'));
	};
}

Scroller.bind = (function() {
	var binded = {};

	return function(id) {
		if(binded[id]) {
			binded[id].unbind(id);
		}
		binded[id] = new Scroller();
		binded[id].init.apply(binded[id], arguments);
	};
})();
</pre>

<h3>JavaScript - My Toolbox</h3>
<pre lang="javascript" colla="-">
// Redefinition of <myfunction>.bind, very useful to keep the scope and <myelement>.addEventListener not defined in IE.
// Useless because browsers that don't define these methods will not manage canvas and so essential.
if(!Function.prototype.bind)
	Function.prototype.bind = function(binding) {
		return function() {
			this.apply(binding, arguments);
		};
	};
if(!HTMLElement.prototype.addEventListener)
	HTMLElement.prototype.addEventListener = function(eventType, listener) {
		this.attachEvent("on" + eventType, function(e) {
			e = e || window.event;
			listener(e);
		});
	};

// A little helper to create an HTML element fast with a parameters list and the classic $ for getElementById
// @params (type, params)
function c(t, o) {
	var d = document.createElement(t);
	o = o || {};
	for(var k in o)
	if(k === 'attributes')
		for(var a in o[k])
		d.setAttribute(a, o[k][a]);
	else
		d[k] = o[k];
	return d;
}

// @params (id)
function $(i) {
	return document.getElementById(i);
}

// Useful for the callbacks in order to do (callback || noop)() instead of a painful if(callback) { callback(); }
function noop() {
}

// Load a picture from its URL and call a function when it's loaded (with metadata)
// @params (url, callback)
function loadImg(u, cb) {
	var i = new Image();
	i.src = u;
	hide(i);
	i.onload = function(e) {
		cb(e);
		i.parentNode.removeChild(i);
	};
	return i;
}

// A round to n instead of 1, a little min/max to assert bounds of a value and a random [0,m]
// @params (value, round)
function r(v, s) {
	s = s || 1;
	return Math.ceil(v / s) * s;
}

// @params (min, value, max)
function mm(mi, v, ma) {
	if(ma>mi) return Math.max(mi, Math.min(v, ma));
	else return Math.max(ma, Math.min(v, mi));
}

// @params (max bound)
function rd(m) {
	return Math.round(Math.random() * m);
}

// Try to log an id for some times. If already locked, return false, else, lock and return true
// @params (lock id, lock time (in s))
var lock = (function() {
	var l = {};
	return function(i, t) {
		var n = new Date().getTime();
		if(l[i] && l[i] > n)
			return false;
		else
			l[i] = n + t * 1000;
		return true;
	};
})();

// File upload via iframe.
// @params (form, callback)
function fuajax(f, cb) {
	var d = 'fu' + rd(999);
	var e = c('iframe', {
		name : d
	});
	hide(e);
	f.setAttribute('target', d);
	e.onload = function() {
		cb(e.contentWindow.document.body.innerHTML);
		e.parentNode.removeChild(e);
	};
}

// Get canvas context
// @params (canvas id)
function ctx(id) {
	return $(id).getContext('2d');
}

window.URL_PATTERN = '^([^:/?#]+:)//[^/?#]+[^?#]*[^#]*(#.*)?';

// Get max value in an array
Array.prototype.max = function() {
	var max = this[0];
	var len = this.length;

	for(var i = 1; i < len; i++)
	if(this[i] > max)
		max = this[i];

	return max;
};

// Get min value in an array
Array.prototype.min = function() {
	var min = this[0];
	var len = this.length;

	for(var i = 1; i < len; i++)
	if(this[i] < min)
		min = this[i];

	return min;
};

// Hide an element in a hidden div
// @param (element)
function hide(element) {
	if(!$('hidden')) {
		var div = c('div', {
			attributes: {
				style: 'width: 0 !important; height: 0 !important; position: absolute !important; top: -10000px !important;',
				id: 'hidden'
			}
		});
		document.body.appendChild(div);
	}
	$('hidden').appendChild(element);
}
</pre>

<h3>Infos</h3>
Not sure the other files need interest, just a bunch of html/javascript/css. But if you want, everything is open source and Beerware (like the v1).
<a href="http://www.korko.fr/sources/clevercover.v2.tar.gz">Download sources</a>, <a href="https://github.com/Korko/CleverCover/">GitHub repository</a>.