---
title: 'Javascript: Programmation Orientée Objets'
date: 2011-03-29 00:39:12
layout: post
---
Bonjou' à vous amis codeurs,

Aujourd'hui j'aimerai vous parler d'un langage globalement assez peu apprécié et trop souvent mal codé : le Javascript. Comme beaucoup de langages de scripts, il est complexe de part sa maniabilité et le nombre impressionnant de manières de coder une même chose.

Je suppose que la majorité d'entre vous programment le javascript de manière linéaire à l'aide de fonctions mais savez-vous qu'il est possible de programmer des concepts objets ? Certains ont peut-être essayé les mots clefs public/private/class etc et pourtant rien n'y fait. En effet, la structure Objet de javascript est particulière. Je vais essayer de vous expliquer certaines structures et leur utilité à chacune.<!--more-->

Commençons par des structures simples en javascript
<pre lang="javascript" colla="+">
var foo = "bar"; // Variable simple
var foobar = ["array"]; // Element Array (tableau)
var bar = {"foo": "bar"}; // Objet
alert(bar.foo); // Affiche "bar"
alert(bar["foo"]); // Affiche aussi "bar"
</pre>

Comme vous pouvez le voir à la dernière ligne, l'utilisation des crochets défini un objet. Il est difficile de définir en tant que tel si il s'agit d'un objet dynamique ou statique puisqu'il ne dispose ici que d'une valeur. Maintenant utilisons cette structure mais en ajoutant cette fois-ci des fonctions.
<pre lang="javascript" colla="+">
var bar = {
    "foo": "bar",
    "foobar": function() {
        alert(bar.foo);
    }
};
bar.foobar(); // Affiche "bar"
</pre>

On peut maintenant dire qu'il s'agit d'un objet statique mais comme vous l'aurez remarqué, tout ici est publique.
Le problème est que pour avoir des données privées, il va falloir changer radicalement de structure.

<pre lang="javascript" colla="+">
var bar = (function() {
    var foo = "foobar";
    var foobar = function() {
        alert(foo);
    };

    return {
        "foobar": foobar
    };
})();
bar.foobar(); // Affiche "foobar"
alert(bar.foo); // Affiche "undefined" car la variable n'est pas accessible
</pre>

Il s'agit ici toujours d'un objet statique mais en revanche, nous pouvons définir des valeurs privées et publiques ce qui est très pratique.

Maintenant, si jamais on voulait avoir des instances pour pouvoir stocker des données, il faut encore changer de structure.
<pre lang="javascript" colla="+">
var foo = function(total) {
    var bar = 4;
    this.foobar = function() {
        alert(total);
    }
};
var foobar = new foo(3);
foobar.foobar(); // Affiche 3
alert(foobar.bar); // Affiche undefined (non accessible)
</pre>

Vous commencez à voir la structure des objets javascript ? Autant dans certains langages comme Java, tout est objet, ici presque tout est fonction.

Voici maintenant deux exemples de codes complets avec une structure différente.
Tout d'abord une instance d'Utilisateur (User)
<pre lang="javascript" colla="-" file="user.js">
var User = function(userInfos) {
	var bind = {};

	// Will return the value or undefined
    this.getInfo = function(key) {
		return userInfos[key];
	};

	// Will update the user values and throw events if needed
	this.updateValues = function(newValues) {
		// Foreach newValues
		for (key in newValues) {
			// If new value
			if (userInfos[key] !== newValues[key]) {
				// If there's callbacks
				if (bind[key]) {
					// Foreach callbacks
					for (index in bind[key]) {
						bind[key][index](key, newValues[key], userInfos[key]);
					}
				}
				// Update values
				userInfos[key] = newValues[key];
			}
		}
	};

	// Attach a callback to a key update
	this.bind = function(key, callback) {
		if (bind[key]) {
			bind[key].push(callback);
		} else {
			bind[key] = [callback];
		}
	};
};

var callback = function(key, newValue, oldValue) {
	alert(key+" "+newValue+" "+oldValue);
};

var toto = new User({'id': 3, 'name': "toto", 'nickname': "toto"});
toto.bind("name", callback);
toto.bind("nickname", callback);
toto.updateValues({'id': 4, 'name': "tata", "nickname": "toto"});
// Will alert('name toto tata')
</pre>

<pre lang="javascript" colla="-" file="cookies.js">
var Cookies = (function() {
	var cookies = {};

	// Init
	if (document.cookie !== "") {
		var cookiesSplit = document.cookie.split(";");
		var key, value;
		for(i=0;i<cookiesSplit.length;i++) {
			key = cookiesSplit[i].substr(0, cookiesSplit[i].indexOf("="));
			value = cookiesSplit[i].substr(cookiesSplit[i].indexOf("=")+1);
			cookies[key] = unescape(value);
		}
	}

	// Will return the value or undefined
	var get = function(key) {
		return cookies[key];
	};

	var set = function(key, value) {
		cookies[key] = value;
		save();
	};

	var save = function() {
		var values = [];
		for(key in cookies) {
			values.push(key+"="+escape(cookies[key]));
		}
		document.cookie = values.join(";");
	};

	return {
		get: get,
		set: set
	};
})();
alert(document.cookie); // Will display "" (on the first call)
Cookies.set("test", "test2");
alert(document.cookie); // Will display "test=test2"
</pre>

Après ces deux exemples, je vais conclure sur une petite spécialité à javascript : le concept de prototype. Elle vous servira à bien des choses comme l'héritage ou même pour alléger vos objets dynamiques (instances via new). En effet, lorsque comme présent ici avec la "classe" User, toutes vos méthodes sont définies à l'intérieur de la "classe", il faut savoir que chaque objet instancié sera une pure copie et donc que chaque objet aura sa copie de la méthode. De ce fait, rien ne m'empêche de faire ceci :
<pre lang="javascript" colla="+">
var foo = function() {
    this.bar = function() {
        alert(2);
    };
};
var foobar = new foo();
foobar.bar(); // Affiche 2
foobar.bar = function() {
    alert(3);
};
foobar.bar(); // Affiche 3
var foobar2 = new foo();
foobar2.bar(); // Affiche 2
</pre>

Voici un petit exemple de prototype:
<pre lang="javascript" colla="+">
var A = function() {
};
A.prototype.echo = function() {
	alert("Hello World!");
};
var instanceA = new A();
instanceA.echo(); // Affiche "Hello World!"

var B = function() {
};
var instanceB2 = new B();
//instanceB.echo(); // Fail: instanceB.echo is not a function
B.prototype.echo = A.prototype.echo;
instanceB.echo(); // Affiche "Hello World!"
instanceB2.echo = function() {
	alert("pwet");
};
instanceB2.echo(); // Affiche "pwet"
instanceB.echo(); // Affiche toujours "Hello World!"
</pre>

Vous pouvez voir que même après la création de l'objet, le fait de passer par le prototype répand la modification à toutes les instances ! Par ailleurs, il est toujours possible de redéfinir la fonction lors d'une instance, il n'y a pas de protection pour cela malheureusement. Cependant, cela ne touche qu'une seule et unique instance.
Il est donc possible de créer une fonction extends qui prendrait deux classes et ferait hériter virtuellement le premier en fils du second.

Bien évidemment, il existe plusieurs Framework javascript qui vous permettent de faciliter tout ceci comme par exemple l'excellent Prototype (http://www.prototypejs.org/) dont vous comprenez maintenant l'appellation. jQuery s'en sort bien aussi mais est plus "machine à café" que prototype qui reste très tourné vers le bas niveau javascript (comme une méthode Class.create par exemple).

Je vous proposerais un article plus tard concernant un domaine très difficile en javascript : le scope. C'est à dire l'environnement dans chaque fonction ainsi que les variables qui y sont définies.