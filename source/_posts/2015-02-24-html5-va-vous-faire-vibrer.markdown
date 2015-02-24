---
layout: post
title: "HTML5 va vous faire vibrer, avec l'API Vibration"
date: 2015-02-24 23:48:52 +0100
comments: true
categories: 
---
Le W3C a publié l'**API Vibration** en [Recommandation](http://www.w3.org/TR/vibration/). Cette API, qui s'adresse particulièrement aux périphériques mobiles (smartphones et tablettes) permet d'émettre un *feedback* à l'utilisateur en utilisant le vibreur du périphérique. Imaginez une vibration émise lorsque l'utilisateur clique sur un bouton. Ou dans un jeu HTML5, en se faisant toucher par un ennemi !

Son utilisation est très simple. L'objet `window.navigator` dispose d'une méthode `vibrate(pattern)`, avec un paramètre *pattern* qui peut être :

* un nombre désignant la durée de la vibration (en millisecondes) ;
* un array contenant alternativement des vibrations et des pauses (en millisecondes).

<!-- more -->

Par exemple, pour faire vibrer l'appareil pendant une seconde :
```js
navigator.vibrate(1000);
```

Qu'on peut écrire aussi :
```js
navigator.vibrate([1000]);
```

Autre exemple, pour faire vibrer l'appareil pendant une seconde, puis patienter pendant une seconde, puis refaire vibrer pendant une demi-seconde :
```js
navigator.vibrate([1000, 1000, 500]);
```

Notez qu'un appel à `vibrate` remplace le pattern de vibration précédent. Pour annuler une vibration en cours :
```js
navigator.vibrate(0);
```

Cette API est désormais un standard, et est actuellement disponible dans Firefox, Chrome, Opera et Android Browser. N'hésitez donc pas à l'utiliser pour enrichir l'interactivité de vos applications HTML5.
