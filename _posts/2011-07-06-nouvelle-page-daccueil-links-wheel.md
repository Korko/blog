---
title: 'Nouvelle page d&rsquo;accueil &#8211; Links Wheel'
date: 2011-07-06 01:49:20
layout: post
---
Bonjou’ à vous amis codeurs,

Comme certains ont pu s'en rendre compte en arrivant sur le site, la page d'accueil à changé pour présenter différentes informations me concernant (mail/profile LinkedIn/CV etc). Pour ce faire, je me suis amusé à coder ma propre "Links Wheel" en javascript. Pour ceux que ça intéresse, je vais essayer d'expliquer un peu le fonctionnement et les choix faits. <!--more-->
<h1>Le code HTML / JSON</h1>
Je voulais tout d'abord avoir un code d'initialisation le plus petit possible.
<pre lang="javascript" colla="+">
var wheelRadius = 250, thumbSize = 100, fullSize = 200;
new linksWheel([
    { name: 'Link1', link: 'http://link1/', img: 'image.png'},
    { name: 'Link2', link: 'http://link2/', img: 'image.png'}
], wheelRadius, thumbSize, fullSize).attach('home');
</pre>
Notez qu'il ne faut surtout pas de "," après le dernier élément. IE interpréterait cela comme un autre élément undefined cette fois.
Du coup au lieu d'avoir Array.length = 2 ici, on aurait 3 pour IE. Très vicieux !

<h1>Javascript - Etape 1 : L'objet</h1>
Partant de ce concept, j'ai pu commencer à créer mon objet JS. Pour ceux qui ne comprendraient pas la structure, vous pouvez aller lire mon article <a href="http://blog.korko.fr/2011/03/29/javascript-programmation-orientee-objets/" title="Javascript: Programmation Orientée Objets">Javascript: Programmation Orientée Objets</a>
<pre lang="javascript" colla="+">
var linksWheel = function(links, r, thumbSize, fullSize) {
        // Constants
        var prefix = 'korko.linkCircle';
        var mainMargin = Math.round(thumbSize/2);
        var angle = 2*Math.PI / links.length;

        // Inner vars
        var content;

        /**
         * Method to attach the circle to a specific HTMLElement (append)
         * @access public
         */
        this.attach = function (divId) {
                document.getElementById(divId).appendChild(content);
        };

        /**
         * Constructor of this object
         * @access private
         */
        (function constructor(links) {
                // Create container
                content = document.createElement('div');
                content.id = prefix;
                content.style.width = 2 * r;
                content.style.height = 2 * r;
                content.style.marginLeft = mainMargin + 'px';
                content.style.marginTop = mainMargin + 'px';

                var i = 0;
                for (link in links) {
                        // TODO: generate link
                        i++;
                }
        })(links);
};
</pre>

<h1>Javascript - Etape 2 : Le cercle de fond</h1>
Bon pour le moment ça fait pas grand chose, ça ajoute juste un div vide... pas de quoi fouetter un chat. Si maintenant, je voulais ajouter un cercle en fond, comment faire ? Impossible d'utiliser une image fixe, le but c'est d'être dynamique sur la taille. Impossible de le générer en Javascript, il faudrait utiliser le canvas HTML5 et j'ai pas ça sous la main (et ce n'est pas compatible avec beaucoup de browsers). La meilleure méthode qui me vient c'est la génération d'une image via PHP. Pour ce faire, j'ajoute dans le constructeur l'image de fond.
<pre lang="javascript" colla="+">
content.style.backgroundImage = 'url(home/circle.php?d='+2*r+')';
</pre>

Et je vais créer mon script PHP (qui utilise la librairie GD) :
<pre lang="php" colla="+">

<?php

$diameter = array_key_exists('d', $_GET) ? intval($_GET['d']) : 100;

$img = imagecreate($diameter, $diameter);
$white = imagecolorallocate($img, 255, 255, 255);
$black = imagecolorallocate($img, 0, 0, 0);
imagearc($img, round($diameter/2), round($diameter/2), $diameter-0.5, $diameter-0.5, 0, 360, $black);

header("Content-type: image/png");
imagepng($img);

imagedestroy($img);
</pre>

Un script bête et méchant en dehors du petit glitch $diameter-0.5 car j'ai remarqué que la génération du cercle faisant 1px de trop sur la droite et en bas.

<h1>Javascript - Etape 3 : Génération des liens</h1>

Avant de penser à positionner nos liens, générons les. Pour ce faire, ajoutons une méthode generateLink qui sera appelée dans la boucle des liens du constructeur.
<pre lang="javascript" colla="+">
        var generateLink = function(link) {
                // Create link
                var anchor = document.createElement('a');
                anchor.setAttribute('href', link.link);
                anchor.style.position = 'absolute';
                anchor.style.textAlign = 'center';
                anchor.style.width = thumbSize+'px';
                anchor.style.height = thumbSize+'px';

                // Create image (size will be compute after display)
                var img = document.createElement('img');
                img.setAttribute('src', link.img);
                img.style.display = 'block';
                img.setAttribute('title', link.name);

                // Add it
                anchor.appendChild(img);

                return anchor;
        };
</pre>

<h1>Javascript - Etape 4 : Position des liens</h1>

Attention, voilà la partie mathématique du dev : positionner les liens en coordonnées absolues autour du cercle.

Imaginons les 360° du cercle (2&pi; radians) et coupons le en N parts (nous avons donc N liens). Chaque lien aura pour coordonnées polaires (cf <a href="http://fr.wikipedia.org/wiki/Coordonn%C3%A9es_polaires" title="Wikipedia">Wikipedia</a>) : (r;&theta;) avec r le rayon du cercle et &theta; l'angle d'une part multipliée par la position du lien dans la liste. Le premier étant donc positionné au départ du cercle trigonométrique (&theta; = 0). Au final, HTML positionnant en coordonnées carthésiennes, avec i : position du lien dans la liste, on a :
<code>
x = r cos(i&theta;);
y = r sin(i&theta;);
</code>

On peut donc modifier notre méthode generateLink pour les positionner :
<pre lang="javascript" colla="+">
        var generateLink = function(link) {
                // Create link
                var anchor = document.createElement('a');
                anchor.setAttribute('href', link.link);
                anchor.style.position = 'absolute';
                anchor.style.textAlign = 'center';
                anchor.style.width = thumbSize+'px';
                anchor.style.height = thumbSize+'px';

                // Create image (size will be compute after display)
                var img = document.createElement('img');
                img.setAttribute('src', link.img);
                img.style.display = 'block';
                img.setAttribute('title', link.name);

                var x = Math.round(r * Math.cos(i * angle));
                var y = Math.round(r * Math.sin(i * angle));

                anchor.style.left = x + r + mainMargin + 'px';
                anchor.style.top = y + r + mainMargin + 'px';

                // Add it
                anchor.appendChild(img);

                return anchor;
        };
</pre>

Vous remarquerez que je positionne l'ancre (anchor) et pas l'image. Cette dernière sera automatiquement déplacée avec le lien. C'est mieux ainsi.

<h1>Javascript - Etape 5 : Centrage des images</h1>

Cependant, vous verrez que les liens sont placés de manière disgracieuse : leur côté haut gauche est bien situé sur le cercle mais l'image s'en trouve complètement excentrée. Connaissant la taille max de la zone, on peut facilement déplacer de thumbSize/2 mais pour une image non carrée, le centrage ne sera pas bon. Le problème c'est que pour plus de généricité, on ne peut pas savoir d'office la taille des images. Essayons de créer une méthode que l'on va appeler resizeAndCenter :
<pre lang="javascript" colla="+">
        var resizeAndCenter = function(img, maxSize, carrier) {
                if (!carrier) carrier = img;

                if (!img.naturalWidth) img.naturalWidth = img.width;
                if (!img.naturalHeight) img.naturalHeight = img.height;

                var ratio;
                if (img.naturalWidth > img.naturalHeight) {
                        img.style.width = maxSize + 'px';
                        ratio = maxSize / img.naturalWidth;
                        img.style.height = Math.round(ratio * img.naturalHeight) + 'px';
                } else {
                        img.style.height = maxSize + 'px';
                        ratio = maxSize / img.naturalHeight;
                        img.style.width = Math.round(ratio * img.naturalWidth) + 'px';
                }

                carrier.style.left = Math.round(parseInt(carrier.style.left) - (ratio * img.naturalWidth)/2) + 'px';
                carrier.style.top = Math.round(parseInt(carrier.style.top) - (ratio * img.naturalHeight)/2) + 'px';
        };
</pre>

On utilise ici une propriété du navigateur (Chrome/Firefox) naturalWidth et naturalHeight pour les images qui donnent la taille originale. L'intérêt est ici de redimensionner uniquement le côté le plus grand (pour garder une image de taille correcte) et en fonction de ces changements, déplacer le lien. Le ratio est important car on force une taille d'image selon un côté uniquement. Ce ne sera donc pas le même pour l'autre. Cependant, naturalWidth et naturalHeight ne sont pas définis dans IE. Nous allons donc profiter que cette méthode fait le redimensionnement plus tard pour prendre la taille (width/height) de l'élément HTML.
Vous remarquerez que j'introduis tout de suite la notion de "carrier" qui permettra de déplacer le lien tout en utilisant les attributs de l'image.
Par ailleurs, je défini aussi bien le width que le height de l'image car IE ne fera pas tout seul le calcul du ratio, il déformera l'image contrairement aux autres navigateurs.

Maintenant, nous allons modifier le generateLink en conséquence :
<pre lang="javascript" colla="+">
        var generateLink = function(link, i) {
                // Create link
                var anchor = document.createElement('a');
                anchor.setAttribute('href', link.link);
                anchor.style.position = 'absolute';
                anchor.style.textAlign = 'center';
                anchor.style.width = thumbSize+'px';
                anchor.style.height = thumbSize+'px';

                // Create image (size will be compute after display)
                var img = document.createElement('img');
                img.setAttribute('src', link.img);
                img.style.display = 'block';
                img.setAttribute('title', link.name);

                // Add it
                anchor.appendChild(img);

                // As we need to display it to know the real height,
                // add it really high and compute before display
                anchor.style.top = -10000 + 'px';
                img.onload = function() {
                        var x = Math.round(r * Math.cos(i * angle));
                        var y = Math.round(r * Math.sin(i * angle));

                        this.parentNode.style.left = x + r + mainMargin + 'px';
                        this.parentNode.style.top = y + r + mainMargin + 'px';

                        resizeAndCenter(this, thumbSize, this.parentNode);
                };

                return anchor;
        };
</pre>

Hélas, vous verrez que je positionne mon ancre en -10000 puis j'utilise l'évènement "onLoad" au lieu de directement placer mon image. La raison est simple : le navigateur ne nous donne les informations de taille que une fois l'élément affiché. Il faut donc l'afficher puis le déplacer.
Je fais remarquer au passage que "onload" ne fonctionne que pour les images. Il n'est pas valable pour des éléments comme "a".

<h1>Javascript - Etape 6 : Over</h1>

Dernière étape : lorsque l'on survole un lien, il faut que l'image soit affichée en gros en milieu du cercle (on a déjà le fullSize pour savoir la taille de l'image).

Pour ce faire, on ajoute à l'intérieur du generateLink :
<pre lang="javascript" colla="+">
                // On Mouse Over, display in full width the image
                anchor.onmouseover = function() {
                        var img = document.createElement('img');
                        img.id = prefix + 'over';
                        img.setAttribute('src', link.img);
                        img.style.display = 'block';
                        img.style.position = 'absolute';

                        // As we need to display it to know the real height,
                        // add it really high and compute before display
                        img.style.top = -10000 + 'px';
                        img.onload = function() {
                                this.style.top = r + mainMargin + 'px';
                                this.style.left = r + mainMargin + 'px';
                                resizeAndCenter(this, fullSize);
                        };

			// IE can put in cache img so never call onload.
			if (img.complete) {
				img.onload();
			}

                        content.appendChild(img);
                };

                // On Mouse Out, remove the full width image
                anchor.onmouseout = function() {
                        var img = document.getElementById(prefix + 'over');
                        if (img) content.removeChild(img);
                };
</pre>

Encore une fois l'astuce du -10000. Par ailleurs, on peut noter le img.complete qui est une petite astuce pour IE. Je me suis arraché les cheveux pas mal de temps pour comprendre pourquoi l'évènement onload n'était jamais lancé. En fait IE met en cache les images et de ce fait ne relance pas le onload. Pour fixer ça, il y a deux méthodes : la première c'est le img.complete qui vaut true si déjà en cache. La seconde est de définir le src <strong>APRES</strong> la définition du onload. De ce fait, IE la prend en compte et ne l'ignore plus.

<h1>Conclusion</h1>
Au final, un petit script marrant, quelques bonnes petites heures à programmer tout le fonctionnement. La version complète peut être vue ici : <a href="http://www.korko.fr/" title="Links Wheel">Website</a> et le code Javascript est là : <a href="http://www.korko.fr/home/circle.js">JS</a>.

Have fun !