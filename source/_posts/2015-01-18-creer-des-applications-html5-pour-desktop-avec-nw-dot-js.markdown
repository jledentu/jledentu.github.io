---
layout: post
title: "Créer des applications HTML5 pour desktop avec NW.js"
date: 2015-01-18 14:18:55 +0100
comments: true
categories: 
---

Vous êtes développeur web et vous aimeriez créer facilement une application native pour Windows, Linux ou MacOS X ? C'est possible ! Plusieurs projets ont vu le jour pour permettre de développer des applications desktop en utilisant des technologies web. [NW.js](http://nwjs.io/) (anciennement Node-Webkit) est l'un d'eux.

NW.js a été créé au sein d'Intel par Roger Wang et publié sous licence MIT. C'est une application native basée sur Chromium et Node.js, qui permet de lire des pages web. C'est donc un outil permettant de développer des applications pour desktop en utilisant des technologies web : HTML5, CSS3, Javascript et WebGL, mais aussi Node.js.

Vous pouvez trouver une liste d'applications développées avec NW.js [ici](https://github.com/nwjs/nw.js/wiki/List-of-apps-and-companies-using-node-webkit). Voyons maintenant comment développer une telle application...

<!--more-->

## Installation

NW.js est facile à installer. Téléchargez la dernière version correspondant à votre système [ici](https://github.com/nwjs/nw.js#downloads), puis vous n'avez qu'à extraire l'archive où vous voulez.

Le répertoire contient l'exécutable NW.js :

* `nw` sous Linux
* `nw.exe` sous Windows
* `Contents/MasOS/node-webkit` sous OS X

## Hello World!

Voyons maintenant comment créer une application NW.js toute simple. Créez un répertoire `nw-helloworld` et créez-y un fichier `index.html` :

```html index.html
<!DOCTYPE html>
<html>
  <head>
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
  </body>
</html>
```

Et créez maintenant un fichier manifest `package.json`, qui contient les paramètres de votre application NW.js :

```json package.json
{
  "name": "nw-helloworld",
  "main": "index.html"
}
```

Lancez votre application avec l'exécutable NW.js :

~~~
nw ./nw-helloworld
~~~

Vous devriez voir ceci :

{% img "center-block img-responsive" /images/nw-helloworld.png %}


<hr />

**_Note_** : sous certaines distributions linux, il est possible que vous rencontriez cette erreur : 

~~~
error while loading shared libraries: libudev.so.0: cannot open shared object file: No such file or directory
~~~

Cette erreur est causée par l'absence de la bibliothèque `libudev0` utilisée par `nw`. Pas de panique, quelques solutions sont proposées [ici](https://github.com/nwjs/nw.js/wiki/The-solution-of-lacking-libudev.so.0). Pour ma part, j'utilise la méthode la plus bourrine qui consiste à modifier l'exécutable :

~~~
sed -i 's/udev\.so\.0/udev.so.1/g' nw
~~~

<hr />

## Utiliser Node.js

Outre les API HTML5 que vous pouvez avoir l'habitude d'utiliser côté client, vous pouvez utiliser Node.js dans votre application comme vous le feriez côté serveur.

Nous pouvons par exemple, accéder au système de fichiers de la machine, grâce à l'API Node.js `fs`.

Créez un fichier `js/main.js` et incluez-le dans le body de votre `index.html` :

```html index.html
<!DOCTYPE html>
<html>
  <head>
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    <script src="js/main.js"></script>
  </body>
</html>
```

Dans ce fichier `main.js`, on utilise l'API `fs` pour lister les fichiers/répertoires présents dans le répertoire de l'application :

```javascript main.js
var fs = require('fs');

fs.readdir('.', function (err, files) {
  if (err) {
    // Erreur
    return;
  }

  var ul = document.createElement('ul');
  var li;

  for (var i = 0, len = files.length; i < len; i++) {
    li = document.createElement('li');
    li.innerHTML = files[i];
    ul.appendChild(li);
  }

  document.documentElement.appendChild(ul);
});
```

En rechargeant l'application, on obtient ceci :

{% img "center-block img-responsive" /images/nw-helloworld2.png %}

## Paramètres

Il est possible de paramétrer l'application grâce au fichier manifest `package.json`, dont vous pouvez trouver le format [ici](https://github.com/nwjs/nw.js/wiki/manifest-format).

Par exemple, le paramètre `window.toolbar` permet de spécifier la présence de la barre d'outils.

```json package.json
{
  "main": "index.html",
  "name": "nw-helloworld",
  "window": {
    "toolbar": false,
  }
}
```

## Packager son application

Générer son application NW.js est très simple ! Deux façons de le faire :

* Recopier les fichiers NW.js dans le répertoire de votre application (où se trouve le fichier `package.json`)
* Ou bien, zipper le contenu du répertoire de votre application dans une archive `package.nw` à placer près du binaire `nw`.

Dans les deux cas, il suffit de lancer l'exécutable `nw` pour démarrer l'appli.

## Conclusion

Comme on l'a vu, NW.js présente pas mal d'avantages :

* Vous pouvez utiliser les technologies web (et Node.js !) ;
* Générer son application et très simple (pas de compilation, pas de dépendances à installer), la distribuer aussi (il n'y a qu'à fournir un répertoire et l'utilisateur peut lancer l'exécutable) ;
* NW.js supporte Linux, MacOS X, Windows.

Parmi les points négatifs, on peut cependant noter :

* plusieurs versions de différence entre la dernière version de Chromium et celle supportée dans NW.js, donc attention si vous voulez utiliser des fonctionnalités HTML expérimentales ;
* le manque de contrôles (buttons, menus, etc.) natifs fait qu'il est difficile de développer une application intégrée au système.