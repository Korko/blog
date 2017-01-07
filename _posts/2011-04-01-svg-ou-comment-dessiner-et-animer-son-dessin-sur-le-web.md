---
title: SVG ou Comment dessiner et animer son dessin sur le web
date: 2011-04-01 01:44:37
layout: post
---
Bonjou’ à vous amis codeurs,

Pour peu que vous ayez essayé un jour de faire un dessin sur internet, vous avez dû vous demander "mais comment vais-je faire ?". Autant pour une image fixe c'est assez "simple" on va dire : on fait l'image en hors ligne, on l'upload sur internet et on l'affiche avec une balise `&lt;img src="http://monurl/image.png" /&gt;`. Autant quant il s'agit d'une image dynamique, la galère commence.<!--more-->

Une première approche consiste à générer l'image coté serveur. Pour ce faire, il existe en PHP la librairie GD qui permet de manipuler des images et de pouvoir la modifier à volonté. C'est ce qui est utilisé pour générer les images de signature dynamique comme par exemple via xfire :
<img src="http://miniprofile.xfire.com/bg/sh/type/0/username.png?test=11" />

L'avantage de cette approche est que pour l'utilisateur final, ce n'est qu'une image comme une autre à afficher rien de plus. Cela implique qu'il n'est pas besoin de procéder à des calculs du coté client (navigateur internet). Le problème majeur est l'absence de dynamisme une fois l'image générée. Prenons un exemple : imaginons que nous devions afficher une balle qui se déplace dans un cadre : un gif suffit. Ajoutons à cela que le cadre doit être redimensionnable. Cas plus complexe. Pour ce faire, nous allons probablement utiliser une image png représentant la balle et l'animer via du Javascript. Si maintenant, nous désirons pouvoir changer la couleur de la balle à volonté (ou sa taille). A moins d'avoir une image pour chaque taille ou de vouloir faire du redimensionnement d'image en Javascript, ça devient impossible.

Il y a certaines choses qui ne se font pas, pour tout le reste il y a SVG. J'entends déjà des voix se lever pour me proposer de faire ça en Flash mais justement, le point ici est de se passer de cette technologie qui est très lourde et nécessite un ajout au navigateur (même si la majorité des internautes l'ont installé mais il n'est pas question ici de faire un étalage du pourquoi je veux me passer de Flash).

Mais alors à quoi ressemble le SVG ? Comment l'utiliser ? Allons-y pas à pas. Le SVG va majoritairement se présenter sous la forme d'un fichier texte avec l'extension .svg qui contiendra du XML sous une forme particulière. La différence avec une image "classique" tient ici du fait qu'on ne va pas représenter pixel par pixel notre image mais élément par élément. Je m'explique : si vous voulez dessiner un trait, en jpg, vous allez définir une suite de pixels qui seront par exemple noir. De ce fait, toute votre image sera blanche avec juste disons 20 pixels noirs. En SVG, vous allez non pas définir les pixels mais la droite elle même. L'intérêt étant de pouvoir manipuler directement l'élément l'agrandir, le rétrécir etc sans jamais avoir de pixelisation. Pour les connaisseurs, le SVG est le format de sauvegarde du dessin vectoriel et votre logiciel favoris (qui est probablement Inkscape) manipule du SVG.

Entrons dans le vif du sujet : comment écrire un SVG. D'une manière similaire à un XML basique, voici la structure de base :
	<?xml version="1.0" encoding="UTF-8" standalone="no"?>
	<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.0//EN" "http://www.w3.org/TR/2001/REC-SVG-20010904/DTD/svg10.dtd">
	<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
	</svg>

Ainsi, entre les balises &lt;svg&gt; et &lt;/svg&gt;, vous allez pouvoir définir ce qui composera votre image. Reprenons l'exemple de notre sphère (positionnée ici en 10:10 et de rayon 5) :
	<?xml version="1.0" encoding="UTF-8" standalone="no"?>
	<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.0//EN" "http://www.w3.org/TR/2001/REC-SVG-20010904/DTD/svg10.dtd">
	<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
	    <ellipse cx="10" cy="10" rx="5" ry="5" />
	</svg>

Je ne ferais pas une liste détaillée de toutes les possibilités de dessin par svg car elles sont très nombreuses et globalement peu de personnes vont réellement générer un dessin à la volée. Le svg va surtout permettre d'animer le dessin qui sera prédéfini. Sachez cependant qu'en plus des formes "basiques" comme rectangle, ellipse etc, SVG propose par exemple les courbes de Béziers. De nombreux articles plus techniques ou diverses documentations vous aideront si vous voulez en savoir plus dans le domaine.

Attardons-nous maintenant un peu plus sur l'intégration dans les HTML puisque c'est une des raisons de cet article. Tout d'abord, il est à noter que SVG peut transporter son propre code CSS/Javascript. Ce qui permet diverses gestions très intéressantes. Pour des raisons de séparation franche entre l'affichage et l'animation, mon SVG ne comportera pas de Javascript.
	<?xml version="1.0" encoding="ISO-8859-1" ?>
	<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd" >
	<html xmlns="http://www.w3.org/1999/xhtml">
	<head>
	    <title>Exemple de fichier SVG</title>
	    <link rel="stylesheet" type="text/css" href="svg_test.css" />
	    <script type="text/javascript" src="svg_test.js"></script>
	</head>
	<body>
	    <embed src="svg_test.svg" type="image/svg+xml" id="svg" />
	</body>
	</html>

Voilà ! Vous avez une sphère dessinée sur votre page web ! Magique non ? Bon et si maintenant on l'animait un peu ? Suite à quelques tests, je vous conseil d'utiliser Firefox pour lire ce fichier. En effet, Chrome semble avoir un bug quand vous essayez d'accéder à un fichier svg en local depuis une page html (Cross-Domain security...) alors que si vous le mettez sur votre hébergeur internet, tout passe bien (adresse en http:// et non file://).

Voici une classe que j'ai programmé pour manipuler le document SVG. Il suffit après d'appeler `SVGDocument.get(votreId)` et il vous retournera soit l'élément soit undefined.
	/**
	 * SVG Document Class
	 */
	var SVGDocument = (function(SVGId) {
		var svg;

		// Init the SVGDocument when document is loaded
		Event.bind('init', function() {
			var svgDom = document.getElementById(SVGId);

			if (svgDom) {
				svg = svgDom.getSVGDocument();
			}
		});

		function get(id) {
			if (!svg || !svg.getElementById) {
				return undefined;
			}
			return svg.getElementById(id);
		}

		return {
			get: get
		};
	})('svg');

<em>Si vous ne comprenez pas cette structure, vous pouvez toujours consulter l'article correspondant : <a href="http://korko.fr/2011/03/29/javascript-programmation-orientee-objets/">Javascript: Programmation Orientée Objets</a></em>

Une fois cela fait, laissez aller votre imagination et faites apparaître, disparaître, changer de couleur, se déplacer, etc tous les éléments de votre SVG à votre guise. Pour ma part, nous avions en projet de fin d'année durant mes études, fait un jeu dans l'esprit du Risk avec une carte SVG dont les pays changent de couleur en fonction de quel joueur les possède. Sur chaque pays, il y a une indication du nombre d'armées ainsi que des flèches qui indiquent les pays vers lesquels on peut se déplacer.

Parlons des choses qui fâchent : le support. Il aurait été trop beau que tout ceci fonctionne parfaitement pour tout le monde. Comme vous pouvez le voir sur ce graphique : http://www.codedread.com/svg-support.php, Internet Explorer fait de la résistance bien que dans sa version 9 il cède enfin du terrain et supporte SVG. Sans entrer dans les détails et d'une manière assez grossière, ce choix est très probablement dû au fait que Silverlight propose des fonctionnalités assez proche de SVG. A l'origine, Adobe proposait un plugin à IE qui permettait de lire le SVG mais le développement s'est arrêté et Adobe ne propose plus ce programme. En revanche, Google s'est lancé sur le terrain mais pour proposer une gestion presque native. Entendez par là que le but est de ne pas avoir à installer un plugin mais de pouvoir instantanément voir le SVG comme si le navigateur le comprenait. Pour ce faire, ils utilisent un fichier Flash (SWF) dans lequel Google (via Javascript) va générer l'image. Le nom de code de ce projet est <a href="http://code.google.com/p/svgweb/">SVGWeb</a>.

Sur-ce, amusez vous bien avez le SVG.
