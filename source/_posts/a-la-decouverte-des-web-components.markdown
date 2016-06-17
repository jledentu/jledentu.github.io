---
layout: post
title: "À la découverte des Web Components"
date: 2015-04-01 11:26:59 +0100
comments: true
categories:
published: true
---

{% img "pull-right img-responsive" /images/webcomponents-logo.png %}
Vous avez peut-être déjà entendu parler des *Web Components*, qui font le buzz parmi les développeurs web, dont certains estiment qu’ils vont bouleverser la façon de développer des applications web. L’idée est de permettre aux développeurs d’applications web d’enrichir le HTML avec des éléments personnalisés, programmables, réutilisables, en s’appuyant sur des technologies standardisées.

Imaginons par exemple qu’on veuille afficher une barre de progression. Avec un web component, ça donnerait ça :

```html
<progress-bar value="10"></progress-bar>
```

À la clé, on a donc un code HTML qui traduit bien mieux sa sémantique, et l’ajout de widgets graphiques complexes se fait plus simplement que par l’utilisation de bibliothèques ou frameworks JavaScript. Le HTML redevient un langage déclaratif accessible à tous et pas seulement aux développeurs – ce que le HTML était, à l’origine.

<!--more-->

## Les technologies à l’œuvre

Les web components s’appuient sur quatre technologies en voie de standardisation par le W3C :

* Les Custom Elements ;
* Les Imports HTML ;
* Les Templates ;
* Le Shadow DOM.

### Custom Elements

La spécification [Custom Elements](http://w3c.github.io/webcomponents/spec/custom/) apporte la possibilité de définir de nouveaux éléments HTML/DOM, en utilisant la méthode `document.registerElement` :

```js
document.registerElement('progress-bar');
```

Note : vous pouvez donner le nom que vous voulez à un custom element, tant qu’il contient un “`-`” pour le distinguer des balises de base, et qu’il ne fait pas partie des noms réservés.

La méthode `registerElement` prend comme deuxième paramètre (optionnel) un objet décrivant le prototype de l’élément, qui va contenir des propriétés, des fonctions, et l’implémentation des *callbacks* suivants :

* `createdCallback`, appelé après que l’élément ait été créé et sa définition enregistrée ;
* `attachedCallback`, appelé après insertion de l’élément dans le DOM ;
* `detachedCallback`, appelé après que l’élément ait été retiré du DOM ;
* `attributeChangedCallback`, appelé lors du changement d’un attribut.

Par exemple :

```js
document.registerElement('progress-bar', {

	prototype: Object.create(HTMLElement.prototype, {
		createdCallback: {
			value: function() {
				this.innerHTML = '<b>' + this.getAttribute('value') + ' %</b>';
			}
		},
		attributeChangedCallback: {
			value: function(attrName, oldValue, newValue) {
				switch (attrName) {
					case 'value':
					this.innerHTML = '<b>' + newValue + ' %</b>';
					break;
				}
			},
		}
	})
});
```

Ce nouvel élément peut être instancié de façon impérative, ou déclaré dans le code HTML de la façon suivante :

```html
<progress-bar value="10"></progress-bar>
```

Et ça donne ça :

**10 %**

## De l’encapsulation avec le Shadow DOM

Le [Shadow DOM](http://www.w3.org/TR/shadow-dom/) est une structure DOM encapsulée dans un élément appelé shadow root. Ce shadow DOM va permettre de masquer la structure interne d’un web component. Ce concept est déjà utilisé pour certains éléments natifs, comme l’élément video par exemple. En inspectant un élément video avec les outils de développement de Chrome, on voit ceci :

{% img "center-block img-responsive" /images/shadowdom1.png %}

Mais en activant l’affichage du Shadow DOM (une option dans les outils de Chrome), on voit ceci :

{% img "center-block img-responsive" /images/shadowdom2.png %}

La méthode `element.createShadowRoot` permet d’ajouter un nœud *shadow root* à un élement du DOM. En reprenant notre exemple de custom element, on peut utiliser cette méthode pour mettre sa structure interne dans du shadow DOM :

```js
document.registerElement('progress-bar', {

	prototype: Object.create(HTMLElement.prototype, {
		createdCallback: {
			value: function() {
				var sr = this.createShadowRoot();
				sr.innerHTML = '<b>' + this.getAttribute('value') + ' %</b>';
			}
		},
		attributeChangedCallback: {
			value: function(attrName, oldValue, newValue) {
				switch (attrName) {
					case 'value':
					var sr = this.shadowRoot;
					sr.innerHTML = '<b>' + newValue + ' %</b>';
					break;
				}
			},
		}
	})
});
```

Les éléments présents dans du Shadow DOM ne sont pas retournés par des méthodes comme `getElementsByTagName`, `getElementById`, ou la propriété `children`, etc.

Par exemple, sur notre élément précédent :

```js
console.log(document.getElementsByTagName('b').length); // 0
console.log(document.getElementsByTagName('progress-bar')[0].children.length); // 0
```

Par contre, si on accède à la propriété `shadowRoot` de l'élément, là on peut récupérer le contenu :

```js
console.log(document.getElementsByTagName('progress-bar')[0].shadowRoot.children.length); // 1
```

Comme vous le voyez, le Shadow DOM sert bien son rôle d'encapsulation. L'intérêt ? Éviter des conflits entre composants, ou entre une bibliothèque et un composant.

La spécification Shadow DOM introduit aussi de nouveaux sélecteurs CSS :

* `:host`, qui permet de sélectionner l’élément host (l’élément qui contient le shadow root) ;
* `::shadow`, pseudo-élément qui permet de sélectionner un élément dans le Shadow DOM d’un élément ;
* `/deep/`, sélecteur qui permet de sélectionner des éléments, qu’ils soient dans le Shadow DOM ou non.

## Templates

L’API [Template](http://www.w3.org/TR/html5/scripting-1.html#the-template-element) HTML5 permet de simplifier l’écriture de la structure HTML interne d’un composant. Un template est un élément "parsé” au chargement de la page, mais qui n’est pas affiché, et peut être cloné plus tard pour être inséré dans le DOM.

```html
<template id="pbTemplate">
	<style>
		:host { display: block; height: 10px; border: solid 1px #ddd }
		.progress { height: 10px; background-color: orange; }
	</style>
	<div class="progress"></div>
</template>

<script>
document.registerElement('progress-bar', {

	prototype: Object.create(HTMLElement.prototype, {
		createdCallback: {
			value: function() {
				var t = document.getElementById('pbTemplate');
				var clone = t.content.cloneNode(true);
				this.createShadowRoot().appendChild(clone);

				this.shadowRoot.querySelector('.progress').style.width = this.getAttribute('value') + '%';
			}
		},
		attributeChangedCallback: {
			value: function(attrName, oldValue, newValue) {
				switch (attrName) {
					case 'value':
					this.shadowRoot.querySelector('.progress').style.width = newValue + '%';
					break;
				}
			},
		}
	})
});</script>
```

À ce stade, voici à quoi ressemble notre custom element :

{% img "center-block img-responsive" /images/webcomponents-progress-bar.png %}

Vous pouvez voir ce que ça donne [ici](http://jsfiddle.net/s2drg647/3/), en utilisant le navigateur Chrome.

## Réutilisation facile avec les Imports HTML

Nous avons passé en revue les technologies qui permettent de créer un web component. L’API HTML Imports permet de rendre les web components facilement réutilisables. Plutôt que d’inclure dans un fichier HTML les fichiers JavaScript, les feuilles de styles CSS, les images, etc. nécessaires à votre composant, que diriez-vous d’inclure un seul fichier HTML :

```html
<link rel="import" href="component.html">
```

Avec les imports HTML, c’est possible. Le fichier component.html contiendrait par exemple :

```html
<link rel="stylesheet" href="css/style.css">
<script src="js/script.js"></script>
<!-- etc. -->
```

Comme vous le voyez, réutiliser des composants existants, qu’ils aient été créés par vous-même, ou récupérés sur le web (par exemple sur [customelements.io](http://customelements.io/)) est très simple.

## Des polyfills pour utiliser les web components dès maintenant

Problème : à l’heure actuelle, seuls les navigateurs Chrome et Opera [supportent](http://caniuse.com/#search=components) par défaut les technologies évoquées ci-dessus. Mais rassurez-vous, un ensemble de *polyfills*, appelé [`webcomponents.js`](https://github.com/webcomponents/webcomponentsjs), permet à tous les navigateurs actuels (IE10+, Firefox, Chrome, Safari 7+, Chrome Android, Safari Mobile) de prendre en charge les technologies de web components. Le support de navigateurs plus anciens est possible à condition d’ajouter d’autres *polyfills*.

`webcomponents.js` peut être téléchargé manuellement à partir de [Github](https://github.com/WebComponents/webcomponentsjs), ou installé dans votre projet grâce à [Bower](http://bower.io/) :

```
bower install --save webcomponentsjs
```

Ou encore, si vous utilisez [npm](https://www.npmjs.com/) :

```
npm install --save webcomponents.js
```

Puis, il suffit d’insérer `webcomponents.js` dans votre document HTML :

```html
<script src="bower_components/webcomponentsjs/webcomponents.min.js"></script>
```

Note : pour maximiser la compatibilité des *polyfills* avec les autres bibliothèques JavaScript, il est préférable d’inclure `webcomponents.js` avant les autres scripts de la balise head du document.

## Utiliser un web component

Une fois les *polyfills* inclus, vous pouvez utiliser les APIs Custom Elements, Shadow DOM, HTML Imports… et donc utiliser des web components ! Par exemple, récupérez un composant (tel que [google-maps](https://github.com/GoogleWebComponents/google-map)), puis incluez-le de la façon suivante :

```html
<link rel="import" href="google-map/google-map.html">
```

Et insérez-le dans votre page :

```html
<google-map latitude="37.77493" longitude="-122.41942"></google-map>
```

## Bibliothèques

Il est tout à fait possible d’écrire un composant web en JavaScript *Vanilla* (sans bibliothèque tierce) en utilisant simplement les technologies exposées précédemment. Néanmoins, plusieurs bibliothèques ont été développées afin de faciliter l’écriture de composants.

### Polymer

[**Polymer**](http://www.polymer-project.org/) est un projet initié par Google, qui permet de créer des web components en proposant une syntaxe allégée. Polymer ajoute aussi des mécanismes tels que les *Data Bindings* ou l’unification des touch/mouse events (*gestures*).
Polymer propose également deux collections de composants :

* **Core elements**, des composants de base, graphiques ou non, pour les interactions, la mise en page, la création de squelettes d’applications web ;
* **Paper elements**, des éléments graphiques qui implémentent la charte Material Design de Google.

### X-Tag

[**X-Tag**](http://x-tags.org/) est une bibliothèque développée par Mozilla. Elle est plus légère que Polymer car elle n’intègre que la gestion des Custom Elements. La création de Shadow DOM ou la gestion de templates HTML sont optionnelles et doivent être ajoutées manuellement par le développeur.

Mozilla a également développé Brick, une collection d’éléments dont l’implémentation s’appuie sur X-Tag, utilisée notamment dans Firefox OS.

### Bosonic

[**Bosonic**](http://bosonic.github.io/) est un autre projet qui propose également un sucre syntaxique pour faciliter l’écriture de web components, en s’appuyant sur les custom elements, le Shadow DOM et les templates. C’est un compromis entre la richesse de Polymer et la légèreté de X-Tags. Bosonic supporte aussi des navigateurs moins récents comme IE 9.

## Conclusion

Si pour l’heure, le fait qu’ils ne soient pas supportés nativement dans tous les navigateurs (mais avec des polyfills) peut poser des problèmes de performances, les web components deviendront bientôt très répandus dans le développement web.

Dans le prochain article, j'expliquerai comment créer une web app avec Polymer.
