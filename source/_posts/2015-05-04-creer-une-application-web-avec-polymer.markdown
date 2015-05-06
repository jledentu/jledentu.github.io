---
layout: post
title: "Créer une application web avec Polymer"
date: 2015-05-06 14:40:00 +0200
comments: true
categories:
---
{% img "pull-right img-responsive" /images/polymer-logo.png %}
Dans [l'article précédent](/blog/2015/04/01/a-la-decouverte-des-web-components/), j'introduisais la notion de *web components*. Nous allons voir maintenant comment créer rapidement une petite application web (*Single Page App*) en assemblant des composants [Polymer](https://www.polymer-project.org/), la bibliothèque de Google qui permet de créer des *web components* facilement, et fournit de base des éléments très utiles.

 *__Note__ : j'ai utilisé ici la version 0.5 de Polymer. La prochaine version majeure (0.8) introduira un certain nombre de changements, mais les concepts présentés ici resteront valables.*

<!--more-->

## Installation de Polymer

Polymer peut être récupéré [avec Git à partir de Github](https://www.polymer-project.org/resources/tooling-strategy.html#git), ou bien [en téléchargeant le ZIP](https://www.polymer-project.org/docs/start/getting-the-code.html#using-zip), ou encore en utilisant le gestionnaire de dépendances Bower. C’est cette dernière méthode qui est officiellement recommandée, et que je vais vous présenter.

[Installez Bower](http://bower.io/#install-bower). Si Bower n’est pas encore configuré dans votre projet, exécutez cette commande à la racine de votre projet pour créer un fichier `bower.json`&nbsp;:

```
bower init
```

Puis installez Polymer :

```
bower install --save Polymer/polymer#^0.5
```

L’installation de Polymer installe également la dépendance `webcomponents.js`, un *polyfill* qui étend le support des *web components* aux navigateurs qui ne les supportent pas nativement.

Pour installer les core-elements&nbsp;:

```
bower install --save Polymer/core-elements
```

## Structure de l’application

Pour créer le layout de l’application, on peut utiliser un élément `core-scaffold`&nbsp;:

```html index.html
<!DOCTYPE html>
<html>
	<head>
		<title>Polymer</title>

		<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
		<meta name="mobile-web-app-capable" content="yes">
		<meta name="apple-mobile-web-app-capable" content="yes">

		<script src="/bower_components/webcomponentsjs/webcomponents.min.js"></script>

		<link rel="import" href="/bower_components/core-scaffold/core-scaffold.html">
		<link rel="import" href="/bower_components/core-header-panel/core-header-panel.html">
		<link rel="import" href="/bower_components/core-menu/core-menu.html">
		<link rel="import" href="/bower_components/core-item/core-item.html">
		<link rel="import" href="/bower_components/core-icons/social-icons.html">
	</head>
	<body>
		<core-scaffold>
			<core-header-panel navigation flex mode="seamed">
				<core-menu>
					<core-item icon="polymer" label="Projects"></core-item>
					<core-item icon="social:people" label="People"></core-item>
				</core-menu>
			</core-header-panel>
			<div tool>Polymer</div>
			<div>Contenu</div>
		</core-scaffold>
	</body>
</html>
```

Avec peu de code, on obtient ceci&nbsp;:

{% img "center-block img-responsive img-bordered" /images/polymer-app-1.png %}

Pour l’affichage du contenu, on va utiliser l’élément `core-animated-pages`, qui permet de switcher entre plusieurs pages de façon animée&nbsp;:

```html index.html
<link rel="import" href="/bower_components/core-animated-pages/core-animated-pages.html">
<link rel="import" href="/bower_components/core-animated-pages/transitions/slide-from-right.html">
```

```html index.html
<div layout horizontal center-center fit>
<!-- Pages -->
	<core-animated-pages selected="0" transitions="slide-from-right" fit>
		<section fit layout vertical center-center>
			<div>
				<h1>Projects</h1>
			</div>
		</section>
		<section fit layout vertical center-center>
			<div>
				<h1>People</h1>
			</div>
		</section>
	</core-animated-pages>
</div>
```

Notez les attributs `layout`, `vertical`, `center-center`, `fit`. Ce sont des attributs apportés par Polymer pour la [mise en page](https://www.polymer-project.org/docs/polymer/layout-attrs.html). Il est possible de positionner, centrer, aligner des éléments HTML sans écrire une seule ligne de CSS&nbsp;!

{% img "center-block img-responsive img-bordered" /images/polymer-app-2.png %}

## Utiliser du Data Binding

Bon c’est bien beau tout ça, mais notre contenu n’est pas du tout dynamique (l’attribut `selected` reste à `0` ce qui affiche la première page). Pour cela, on va utiliser du *two-way data binding*, mécanisme apporté par Polymer aux templates HTML, qui permet de lier la valeur d’un attribut ou d’un élément de texte à une variable du modèle de template. Pour s’en servir en-dehors d’un composant Polymer, on ajoute un template avec l’attribut `is="auto-binding"`&nbsp;:

```html index.html
<body>
	<template is="auto-binding">
		<core-scaffold>
		<!-- ... -->
		</core-scaffold>
	</template>
</body>
```

Puis, on ajoute un binding `{% raw %}{{page}}{% endraw %}` sur l’attribut `selected` du menu&nbsp;:

{% raw %}
```html index.html
<core-menu selected="{{page}}">
	<core-item icon="polymer" label="Projects"></core-item>
	<core-item icon="social:people" label="People"></core-item>
</core-menu>
```

Et sur l’attribut `selected` du `core-animated-pages` :

```html index.html
<core-animated-pages selected="{{page}}" transitions="slide-from-right" fit>
	<section fit layout vertical center-center>
		<div>
			<h1>Projects</h1>
		</div>
	</section>
	<section fit layout vertical center-center>
		<div>
			<h1>People</h1>
		</div>
	</section>
</core-animated-pages>
```

Le menu est désormais relié aux pages. En cliquant sur l’entrée "People", la page "People" s’affiche, etc.

## Requêtes Ajax

Le composant `core-ajax` permet de faire des requêtes Ajax. Utilisons-le pour obtenir la liste des projets Polymer dans Github&nbsp;:

```html index.html
<link rel="import" href="/bower_components/core-ajax/core-ajax.html">
```

```html index.html
<body>
<core-ajax
	id="ajaxProjects"
	auto
	url="https://api.github.com/orgs/Polymer/repos"
	handleAs="json"></core-ajax>
<template is="auto-binding">
<!-- ... -->
```

Ici, le paramètre `auto` indique que la requête XHR est faite automatiquement quand l’attribut `url` est modifié. Un événement `core-response` permet de traiter la réponse (ici en JSON). On ajoute une *callback* pour ajouter un paramètre `projects` au modèle du template, qu’on va pouvoir utiliser comme *data binding* :

```html index.html
<script>
document.addEventListener('polymer-ready', function() {
	var ajaxProjects = document.getElementById('ajaxProjects');
	ajaxProjects.addEventListener('core-response',
		function(e) {
			document.querySelector('template').model = {
				projects: e.detail.response
			};
		}
	);
});
</script>
```

On utilise maintenant un template itératif de Polymer pour afficher le contenu de `projects`&nbsp;:

```html index.html
<core-animated-pages selected="{{page}}" transitions="slide-from-right" fit>
	<section fit layout vertical center-center>
		<div>
			<h1>Projects</h1>
			<template repeat="{{p in projects}}">
			<div>{{p.name}}</div>
			</template>
		</div>
	</section>
```
{% endraw %}

L’attribut `repeat` du template permet de générer un fragment de DOM pour chaque élément du tableau `projects`.
Et voici le résultat&nbsp;:

{% img "center-block img-responsive img-bordered" /images/polymer-app-3.png %}

## Créer un nouveau composant

Nous allons maintenant voir comment créer notre propre composant Polymer. Ici, on veut générer une carte pour afficher les détails d’un projet Github.

### Structure du composant

Dans un fichier `project-card/project-card.html`, créons la structure du composant&nbsp;:
{% raw %}
```html project-card/project-card.html
<link rel="import" href="/bower_components/polymer/polymer.html">

<polymer-element name="project-card" noscript>
	<template>
		<h2>
			<a class="project-name">Nom du projet</a>
		</h2>
		<p class="project-desc">description</p>
	</template>
</polymer-element>
```

Cette syntaxe permet de créer et enregistrer un nouvel élément `project-card`, avec un template qui sera cloné et ajouté au Shadow DOM. L’attribut `noscript` indique qu’il n’y a pas de JavaScript associé, et l’élément est ajouté au registre automatiquement.

### Ajouter des attributs

On désire ajouter des attributs pour rendre notre composant paramétrable. On peut utiliser pour cela l’attribut `attributes` :

```html project-card/project-card.html
<polymer-element name="project-card" noscript attributes="name url description">
```

Faisons le lien entre nos attributs et le contenu de notre composant en utilisant des *data bindings*&nbsp;:

```html project-card/project-card.html
<polymer-element name="project-card" noscript attributes="name url description">
	<template>
		<h2>
			<a class="project-name" href="{{url}}">{{name}}</a>
		</h2>
		<p class="project-desc">{{description}}</p>
	</template>
</polymer-element>
```

Utilisons ce composant pour afficher les projets Github de Polymer dans notre `index.html`&nbsp;:

```html index.html
<link rel="import" href="/project-card/project-card.html">
```

```html index.html
<section fit layout vertical center-center>
	<div fit>
		<h1>Projects</h1>
		<template repeat="{{p in projects}}">
		<project-card name="{{p.name}}" url="{{p.html_url}}" description="{{p.description}}"></project-card>
		</template>
	</div>
</section>
```
{% endraw %}

Voilà ce que ça donne&nbsp;:

{% img "center-block img-responsive img-bordered" /images/polymer-app-4.png %}

### Un peu de style

On peut modifier l’aspect du composant en ajoutant une balise `style` dans son template&nbsp;:
{% raw %}
```html project-card/project-card.html
<polymer-element name="project-card" noscript attributes="name url description">
	<template>
		<style>
		:host {
			display: block;
			border: solid 1px #ddd;
			margin-bottom: 10px;
			padding: 10px;
			background-color: white;
		}

		.project-name {
			text-decoration: none;
			color: black;
		}

		.project-desc {
			color: #888;
		}
		</style>
		<h2>
			<a class="project-name" href="{{url}}">{{name}}</a>
		</h2>
		<p class="project-desc">{{description}}</p>
	</template>
</polymer-element>
```

Et voici ce qu’on obtient dans l’application&nbsp;:

{% endraw %}
{% img "center-block img-responsive img-bordered" /images/polymer-app-5.png %}
{% raw %}

### Scripter le composant

Pour scripter le composant et programmer son fonctionnement, on peut ajouter une balise `script` dans la définition du composant (sans oublier de retirer l’attribut `noscript`)&nbsp;:

```html
<link rel="import" href="/bower_components/polymer/polymer.html">

<polymer-element name="project-card" attributes="name url description">
	<template>
		<style>
		:host {
			display: block;
			border: solid 1px #ddd;
			margin: 10px;
			padding: 10px;
			background-color: white;
		}

		.project-name {
			text-decoration: none;
			color: {{isPolymer ? 'red' : 'black'}};
		}

		.project-desc {
			color: #888;
		}
		</style>
		<h2>
			<a class="project-name" href="{{url}}">{{name}}</a>
		</h2>
		<p class="project-desc">{{description}}</p>
	</template>
	<script>
	Polymer('project-card', {
		isPolymer: false,

		created: function() {},
		domReady: function() {},
		ready: function() {},
		attached: function() {},
		detached: function() {},
		attributeChanged: function(attr, oldVal, newVal) {},

		nameChanged: function(oldValue, newValue) {
			if (newValue === 'polymer') {
				this.isPolymer = true;
			}
		}
	});
	</script>
</polymer-element>
```

On peut y trouver les *callbacks* du cycle de vie du composant :

* `created`&nbsp;: appelé à la création de l’élément&nbsp;;
* `domReady`&nbsp;: appelé quand les enfants et les voisins de cet élément dans le DOM existent&nbsp;;
* `ready`&nbsp;: appelé quand ce composant Polymer est prêt&nbsp;;
* `attached`&nbsp;: appelé quand cet élément a été inséré dans le DOM&nbsp;;
* `detached`&nbsp;: appelé quand cet élément a été retiré du DOM&nbsp;;
* `attributeChanged`&nbsp;: appelé quand un attribut a été ajouté, retiré, modifié.

Ici, on crée une *callback* sur l’attribut `name` pour modifier une propriété du composant `isPolymer`, utilisée dans une expression de *data binding* pour mettre en valeur le projet Polymer. Résultat&nbsp;:

{% endraw %}
{% img "center-block img-responsive img-bordered" /images/polymer-app-6.png %}

Bien sûr, le script présenté ici n'est pas très intéressant, mais c'est juste un exemple. En réalité, on scripterait les interactions avec le composant, la mise à jour de son état, etc.

### Tester le composant

Polymer propose un outil pour tester des composants&nbsp;: web-component-tester.

Installez-le avec la commande suivante&nbsp;:

```
npm install -g web-component-tester
```

Et&nbsp;:

```
bower install --save-dep web-component-tester
```

Créez un fichier `test/project-card-tests.html`. Ce fichier va contenir des suites de tests, soit des ensembles d'assertions pour tester l'état et le comportement du composant. Ici, on teste juste la valeur par défaut de l'attribut `isPolymer`&nbsp;:

```html test/project-card-tests.html
<!DOCTYPE html>
<html>
<head>
  <script src="../bower_components/webcomponentsjs/webcomponents.min.js"></script>
  <script src="../bower_components/web-component-tester/browser.js"></script>

  <link rel="import" href="../project-card/project-card.html">
</head>
<body>

<project-card></project-card>

<script>
  var pCard = document.querySelector('project-card');

  suite('<project-card>', function() {

    test('default attribute values', function() {
      assert(pCard.isPolymer === false, 'isPolymer is false');
    });
  });
</script>

</body>
</html>
```

On peut créer un fichier `test/index.html` qui va exécuter les tests des composants de notre application, grâce à la méthode `WCT.loadSuites`&nbsp;:

```html test/index.html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <script src="../bower_components/web-component-tester/browser.js"></script>
  <link rel="import" href="../project-card-tests.html">
</head>
<body>
  <script>
    WCT.loadSuites([
      'project-card-tests.html'
    ]);
  </script>
</body>
</html>
```

Et voilà ce que ça donne si on affiche la page de test&nbsp;:

{% img "center-block img-responsive img-bordered" /images/polymer-app-tests1.png %}

On peut également exécuter la commande `wct` à la racine de l'application, qui va exécuter les tests sur les différents navigateurs du système grâce à Selenium&nbsp;:

```plain
wct
```

```plain
Starting Selenium server for local browsers
Selenium server running on port 52803
Web server running on port 52812 and serving from /Users/jeremie/prog/polymer-app
chrome 42                Beginning tests via http://localhost:52812/components/polymer-app/generated-index.html?cli_browser_id=0
chrome 42                Tests passed
firefox 37               Beginning tests via http://localhost:52812/components/polymer-app/generated-index.html?cli_browser_id=1
firefox 37               Tests passed
chrome 42 (1/0/0)                       firefox 37 (1/0/0)
```

## Conclusion

On a vu ici comment créer une application web rapidement en assemblant des composants Polymer, fournis ou créés *from scratch*. Pour aller plus loin, nous aurions pu&nbsp;:

* faire communiquer différents composants avec des signaux (`core-signals`) ;
* utiliser du routage, grâce au composant [`app-router`](https://github.com/erikringsmuth/app-router) ;
* utiliser Vulcanize pour [concaténer des composants](https://www.polymer-project.org/0.5/articles/concatenating-web-components.html) et réduire le nombre de requêtes...

Vous pouvez jeter un coup d’œil au code source de cette application d'exemple sur [Github](https://github.com/jledentu/polymer-app) (j’ai généré le projet en utilisant [Yeoman](http://yeoman.io/) et le générateur [Polymer](https://github.com/yeoman/generator-polymer), ce que je vous conseille de faire également).

*Merci à [Wassim Chegham](https://twitter.com/manekinekko) pour sa relecture.*
