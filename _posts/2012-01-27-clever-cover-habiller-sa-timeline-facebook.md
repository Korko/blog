---
title: 'Clever Cover &#8211; Habiller sa Timeline Facebook'
date: 2012-01-27 03:07:09
layout: post
---
<h3>Non, je ne vous ai pas oublié !</h3>
<p style="text-align: justify;">Bonjou' à tous ! Bien que certains pouvaient le penser, je ne vous ai pas oublié !
Il faut bien l'avouer, j'ai eu des périodes de haut et de bas niveau programmation et nombreux sont les projets qui n'avancent pas.</p>
<p style="text-align: justify;">Heureusement pour vous, j'ai eu le temps de faire un petit script pour habiller son profil Facebook avec le nouveau design bientôt imposé pour tous : la Timeline.
Pour ceux qui ne savent pas ce que c'est : <a href="https://www.facebook.com/about/timeline" title="Facebook - A propos de la Timeline">https://www.facebook.com/about/timeline</a>.</p>
<p style="text-align: justify;">Comme vous pouvez le remarquer, il y a une grande image de fond (appelée Couverture) et votre image de profil.</p>

<h3>Clever Cover</h3>
<p style="text-align: justify;">Mon script va vous permettre de découper une grande image en deux blocs qui s'imbriqueront parfaitement.</p>
<p style="text-align: justify;">Voici un exemple avec mon propre profil : </p>
<a href="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0058.png"><img class="alignnone size-medium wp-image-170" title="Clever Cover : Exemple de résultat" src="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0058-300x124.png" alt="" width="300" height="124" /></a><!--more-->

<h3>Où le trouver ? Comment l'utiliser ?</h3>
<p style="text-align: justify;">Alors, tout d'abord, où trouver cet outil ? <a title="Clever Cover" href="http://www.korko.fr/clevercover/">Clever Cover</a></p>
<p style="text-align: justify;">Ensuite, pour l'utiliser c'est simple :
<ol>
<li>Tout d'abord, entrez l'adresse de l'image que vous voulez utiliser dans le champ de texte en haut à gauche<br /><a href="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0150.png"><img src="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0150-150x150.png" alt="" title="2012-01-27_0150" width="150" height="150" class="alignnone size-thumbnail wp-image-183" /></a></li>
<li>Cliquez sur "Generate". Cette étape peut être longue (~5/10s), cela dépend de votre image.</li>
<li>Vous pouvez agrandir l'image en utilisant cette petite scrollbar horizontale :<br /><a href="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0201.png"><img src="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0201-150x150.png" alt="" title="Clever Cover : Etape 2" width="150" height="150" class="alignnone size-thumbnail wp-image-184" /></a></li>
<li>Vous pouvez déplacer l'image comme si vous déplaciez un fichier (drag/drop)</li>
<li>Une fois la taille et l'emplacement terminés, cliquez sur "Extract"</li>
<li>Il ne vous reste plus qu'à enregistrer les images maintenant affichées (clic droit => "Enregistrer sous") et à les utiliser sur votre profil Facebook :<br /><a href="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0203.png"><img src="http://blog.korko.fr/wp-content/uploads/2012/01/2012-01-27_0203-150x150.png" alt="" title="Clever Cover : Etape 3" width="150" height="150" class="alignnone size-thumbnail wp-image-185" /></a></li>
</ol>

<h3>Un peu de code</h3>
<p style="text-align: justify;">Après tout, nous sommes là pour ça alors ne nous en privons pas.</p>
<p style="text-align: justify;">Ce petit script m'a permit d'apprendre à utiliser les Canvas en HTML5. Il n'a été développé que pour Chrome, mais je suppose qu'il doit fonctionner sous Firefox aussi. Je vais mettre ici le code mais en version plus commentée que celle en ligne afin de mieux entrer dans les détails.</p>

<p style="text-align: justify;">Commençons par le coeur même du site : le Javascript. 99% de la logique est dedans alors accrochez vous</p>
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
 * Ici on trouve tout un tas de Helpers donc des méthodes sans véritable intérêt
 * Si ce n'est qu'elles raccourcissent le temps de développement
 */

// Redéfinition du <myfunction>.bind, très utile pour conserver le scope et du <myelement>.addEventListener non défini sous IE.
// Inutiles puisque le script n'est pas fonctionnel sous IE donc indispensables.
if(!Function.prototype.bind)Function.prototype.bind=function(binding){return function(){this.apply(binding,arguments);};};
if(!HTMLElement.prototype.addEventListener)HTMLElement.prototype.addEventListener=function(eventType,listener){this.attachEvent("on"+eventType,function(e){e=e||window.event;listener(e);});};

// Un petit helper pour créer un élément HTML rapidement avec une liste de paramètres et le classique $ pour getElementById
function c(t,o){var d=document.createElement(t);o=o||{};for(var k in o)if(k === 'attributes')for(var a in o[k])d.setAttribute(a,o[k][a]);else d[k]=o[k];return d;}
function $(i){return document.getElementById(i);}

// Utile pour les callback pour faire simplement (callback || noop)() plutôt qu'un pénible if(callback) { callback(); }
function noop(){}

// Charger une image depuis son URL et appeler un callback une fois celle-ci totalement chargée (avec metadata)
function loadImg(u,c){var i=new Image();i.src=u;$('hidden').appendChild(i);i.onload=c;return i;}

// Un arrondi à n au lieu de 1 et un petit min/max pour s'assurer des bornes d'une valeur
function r(v,s){s=s||1;return Math.ceil(v/s)*s;}
function mm(mi,v,ma){return Math.max(mi,Math.min(v,ma));}

/**
 * Voici LA classe qui gère tout l'affichage de la couverture et de l'image
 * A l'origine les deux ne faisaient parti que d'un seul et unique <canvas> mais malheureusement
 * Il n'est pas possible d'extraire seulement une parti d'un canvas sous forme d'image
 * Alors voici 2 canvas différents : un pour la couverture et l'autre pour l'image.
 */
var Cover = function() {

	var COVER_WIDTH = 850, // Ces chiffres sont issus d'une analyse CSS de Facebook
		COVER_HEIGHT = 313,
		FULL_HEIGHT = 355,
		img = null,
		x = 0,
		y = 0,
		ratio = 0;

	// Encore deux petites fonctions Helper pour récupérer un canvas et son contexte
	function cv(c){return $('canvas_'+c);}
	function ctx(c){return cv(c).getContext('2d');}

	this.draw = function() {
		if (img === null) return;

		drawCover();
		drawPicture();
	};

	// On récupère un rectangle avec comme point d'origine (x,y) (ce sont des valeurs négatives d'où le -x/-y)
	// Ce rectangle a à l'origine une largeur de COVER_WIDTH/ratio et une hauteur de COVER_HEIGHT/ratio
	// Ce ratio comme vous le verrez plus bas est un pourcentage de l'image de base.
	// On copie ce rectangle dans le canvas en position (0,0) (normal) et avec comme taille, celles du canvas (toujours normal)
	function drawCover() {
		ctx('cover').drawImage(img, Math.max(0, -x), Math.max(0, -y), COVER_WIDTH/ratio, COVER_HEIGHT/ratio, 0, 0, COVER_WIDTH, COVER_HEIGHT);
	}

	// Cete fois-ci, nous avons des positions d'origine qui sont données pour les tailles du canvas.
	// De ce fait, il faut les corroborer avec le ratio. Donc la position du carré dans l'image de base est (32/ratio - x, 230/ratio - y)
	// Toujours ce x/y qui permettent de déplacer l'image à l'intérieur du canvas
	// La taille affichée chez Facebook est 125px au carré donc adaptons encore une fois au ratio.
	// Position dans le canvas d'arrivé et tailles ne sont pas à expliquer
	function drawPicture() {
		ctx('picture').drawImage(img, 32/ratio - x, 230/ratio - y, 125/ratio, 125/ratio, 0, 0, 125, 125);
	}

	// Changeons l'image du canvas
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

	// Changeons le ratio d'affichage. Donnons donc une valeur entre 0 et 100
	this.changeRatio = function(newRatio) {
		newRatio = mm(0, newRatio, 100);

		// Au final, cette valeur [0,100] doit être transformer.
		// En effet, la plus petite valeur est celle qui affiche l'image dans toute sa largeur (exactement celle de la couverture)
		var min = COVER_WIDTH / img.naturalWidth * 100;
		ratio = (min + newRatio * (100 - min)/100)/100; // ratio est compris dans [min, 100]

		// Si par hasard l'image est toute décalée, le fait de réduire doit la déplacer
		if (COVER_WIDTH/ratio - x > img.naturalWidth) {
			x += COVER_WIDTH/ratio - x - img.naturalWidth;
		}
		if (FULL_HEIGHT/ratio - y > img.naturalHeight) {
			y += FULL_HEIGHT/ratio - y - img.naturalHeight;
		}
		this.draw();
	};

	this.save = function() {
		// L'image de la couverture que nous pourrons sauvegarder est en fait une chaîne sous forme base64
		// e.g. data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAALQAAAC0CAYAAAA9zQYyAAAgA...
		// Fonctionnalité permise par un canvas si l'origine de l'image est sûre (même domaine) d'où le fichier php "img.php" plus bas
		$('output_cover').setAttribute('src', cv('cover').toDataURL());

		// Autant la couverture c'est simple mais chez Facebook on fait pas les choses simples.
		// L'image du profile est affichée en 125x125 dans la Timeline mais il faut qu'elle soit au moins de 180x180 donc...
		// Redimensionnons pour exporter l'image du résultat
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
		// Initialisons le Drag/Drop des canvas (dans la div d'id canvas)
		drag.init('canvas', function(data) {
			// Si on essaye de déplacer les éléments dans "canvas", déplaçons au final l'image de fond des canvas
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

		// Petite astuce pour permettre l'enregistrement : passer par un script php qui réaffiche l'image comme si elle était sur le même serveur
		// Sinon il n'est pas possible de faire un toDataURL sur un canvas qui affiche une image externe
		cover.setImage('img.php?src='+$('url').value, function() {
			$('loading').style.display = 'none';

			// Initialisons la barre de scroll
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

		// Chrome utilise event.x/event.y
		// Firefox utilise event.clientX/event.clientY
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

Je ne pense pas que le HTML soit vraiment intéressant. En revanche, le script PHP qui affiche l'image distante comme si elle était ici...
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
<p style="text-align: justify;">Je n'en avais pas encore parlé ici mais la majorité de mes codes sont sous "licence" Beerware. Traduisez que vous en faites ce que vous voulez. Il n'y a pas de limitation. Mais si mon boulot vous plait et que vous voulez me le faire savoir, n'hésitez pas à me payer un pinte la prochaine fois qu'on se verra.</p>

<h3>Sources</h3>
<p style="text-align: justify;">Pour télécharger les sources complètes : <a href='http://blog.korko.fr/wp-content/uploads/2012/01/clevercover.zip'>clevercover</a></p>