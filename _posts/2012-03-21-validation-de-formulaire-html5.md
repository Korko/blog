---
title: Validation de Formulaire (HTML5)
date: 2012-03-21 00:30:48
layout: post
---
Bonjour à tous,

Un petit article rapide à propos d'un sujet que je trouve formidable : la validation des formulaires via HTML5.
En effet, HTML5 apporte de nouveaux moyens de s'assurer que ce que l'utilisateur envoie comme données est valide. Après, le principe de toujours se protéger de ce que l'utilisateur donne est toujours valable, HTML5 apporte juste une couche de validation qui permet de simplifier la vie de l'utilisateur.<!--more-->

<h3>Required</h3>
Premier attribut qui permet de valider que le champ est bien rempli
<code>&lt;input type="text" required="required" /&gt;</code>
<img src="http://content.screencast.com/users/Korko/folders/Jing/media/26a9c3f1-7d34-4f44-9c16-f1b4a665fa2f/2012-03-20_2144.png" alt="Input required" />

Il est possible de spécifier l'attribut "title" pour afficher un message plus sympathique aux utilisateurs :
<img src="http://content.screencast.com/users/Korko/folders/Jing/media/b2a4d543-3d1b-43f9-bdb4-7a78099c9d0e/2012-03-20_2153.png" alt="Input required with title" />

Firefox permet de spécifier le message d'erreur (ici celui de Chrome) via l'attribut "x-moz-errormessage" mais aucun parallèle n'existe chez ses concurrents à l'heure actuelle.

<h3>Pattern</h3>
Autre attribut très intéressant qui permet de définir l'expression régulière (type POSIX) pour valider le contenu d'un champ si il est rempli. Du coup, il faut en général combiner pattern et required.
<code>&lt;input type="text" title="Ceci est un titre" pattern="^\d+$" /&gt;</code>
<img src="http://content.screencast.com/users/Korko/folders/Jing/media/8c22559c-3cde-43b6-8055-8f34a0f234da/2012-03-20_2258.png" alt="Input pattern" />

<h3>Custom Validity</h3>
Il existe une chose encore qui est très sympathique : le CustomValidity :
<code><pre>
&lt;input id="input" title="Ceci est un titre" type="text" /&gt;
&lt;script type="text/javascript"&gt;
    var input = document.getElementById("input");
    input.addEventListener("input", function() {
        if (input.value < 3) {
            input.setCustomValidity("Minimum is 4");
        }
    });
&lt;/script&gt;
</pre></code>
<img src="http://content.screencast.com/users/Korko/folders/Jing/media/492025bd-b467-4d73-adf0-03a6709ea89a/2012-03-20_2305.png" alt="Input custom validity" />

<h3>Types d'input</h3>
HTML5 arrive aussi avec des nouveaux types d'input qui contiennent donc des validations personnalisées comme
&lt;input type="email" /&gt;
<img src="http://content.screencast.com/users/Korko/folders/Jing/media/d0240ff9-154a-416a-bf39-f2a40354a2a0/2012-03-20_2310.png" alt="Input email" />

Il y en a d'autres comme url, ou encore certains non encore disponibles sur Chrome comme date, datetime, search, tel, color etc, qui verrons probablement arriver des calendriers, des palettes de couleur, etc, allez savoir.

En attendant, il y a d'autres input spéciaux comme
&lt;input type="number" min="0" max="10" step="3" /&gt;
<img src="http://content.screencast.com/users/Korko/folders/Jing/media/6f43b3e7-0c5c-4396-964a-e92284817516/2012-03-20_2315.png" alt="Input number" />

Où on peut définir des valeurs min/max et même un step (c'est à dire ici que l'on passe de 3 en 3).

Bon codage à tous !

<div style="font-size: 10px;line-height:10px">
Sources de cet article :
http://stephenwalther.com/blog/archive/2012/03/13/html5-form-validation.aspx
http://www.html5rocks.com/en/tutorials/forms/html5forms/
</div>