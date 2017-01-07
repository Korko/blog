---
title: Focus dans un textarea
date: 2012-03-30 14:16:46
layout: post
---
Bonjour tout le monde !

Cette fois-ci, j'ai eu un problème avec un textarea. Je voulais être capable de mettre le focus à un endroit précis et ensuite, pouvoir ajouter du contenu à cet endroit (là où est mon curseur de texte).<!--more-->

Ça parait simple dit comme ça mais en fait pas du tout. Voyons le code (jQuery) :
<pre lang="javascript" colla="+">
$.fn.extend({
	focusOnPos: function(pos) {
		// "this" dans une méthode jQuery est un tableau d'éléments
		$.each(this, function(i, obj) { // this === obj
			if (this.createTextRange) { // IE
				var range = this.createTextRange();
				range.move("character", pos); // Déplacement jusqu'à "pos" et selection
				range.select();
			} else if (this.setSelectionRange) { // FF/Chrome
				this.focus();
				this.setSelectionRange(pos, pos); // On selectionne de "pos" à... "pos"
			} else { // Par Defaut
				this.focus(); // Secours : on fait simplement un focus à la fin
			}
		});
	},
	insertOnFocus: function(content) {
		$.each(this, function(i, obj) {
			if (document.selection) { // IE
				sel = document.selection.createRange();
				sel.text = content; // On remplace ce qui était sélectionné par le nouveau contenu... Etrange comme comportement
			} else if (this.selectionStart !== undefined) { // FF/Chrome
				var pos = this.selectionStart;
				this.value = this.value.substring(0, pos) + content + this.value.substring(pos); // On coupe avant, après et on insert au milieu
			} else { // Default
				this.value += content; // Encore une fois, secours : on ajoute à la fin
			}
		});
	}
});
</pre>

Maintenant que nos méthodes sont définies, faisons un petit test avec une simple fonction JS
<pre lang="javascript" colla="+">
function showTextarea() {
	var rawContent = 'a § b'; // Chaîne simple avec un marqueur spécifique (ici §)
	var rawCursorPosition = rawContent.indexOf('§'); // Trouver la position du marqueur (future position du curseur)
	rawContent = rawContent.replace('§', ''); // Supprimer le marqueur maintenant qu'on a la position

	if ($('#textarea').is(':visible')) {
		$('#textarea').hide();
	} else {
		$('#textarea').val(rawContent).show().focusOnPos(rawCursorPosition);

		// Juste pour voir le changement
		setTimeout(function() {
			$('#textarea').insertOnFocus('c');
		}, 2000);
	}
}
</pre>

Et un HTML tout simple
<pre lang="html" colla="+">
<a href="#" style="display: block;" onclick="showTextarea(); return false;">Show/Hide</a>
<textarea id="textarea" style="display: none"></textarea>
</pre>

Testons : <a href="http://www.korko.fr/sources/examples/focus-in-a-textarea.html">Essayer</a>