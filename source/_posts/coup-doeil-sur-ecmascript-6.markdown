---
layout: post
title: "Coup d'œil sur ECMAScript 6"
date: 2015-02-20 20:12:37 +0100
comments: true
categories: 
---
JavaScript est une implémentation de ECMAScript, le langage de script standardisé par la norme ECMA-262. La dernière édition publiée de cette norme est la 5.1, qui est [supportée](http://kangax.github.io/compat-table/es5/) par tous les navigateurs web modernes. L'édition 6, baptisée *Harmony*, est en cours de préparation. Elle ambitionne d'améliorer notablement le langage et de le rendre plus simple pour écrire des applications complexes. D'ors et déjà, les moteurs Javascript implémentent déjà certaines fonctionnalités (une table de compatibilité se trouve [ici](http://kangax.github.io/compat-table/es6/)).

Découvrons maintenant quelques-unes des nouveautés apportées par ES6 (liste non exhaustive)...

<!--more-->

## Classes et héritage

Les classes ES6 proposent une syntaxe plus simple et plus légère (un "sucre syntaxique") pour créer des "classes" :

```js
class Polygon {
  constructor(height, width) { // Constructeur
    this.name = 'Polygone';
    this.height = height;
    this.width = width;
  }

  sayMyName() { // Méthode
    console.log('Coucou, je suis un ', this.name + '.');
  }
}
```

On instancie un objet classiquement :

```js

console.log(typeof Polygon === 'function'); // true

var p = new Polygon(5, 5);
p.sayMyName(); // Coucou, je suis un Polygone.
```

Pour créer une classe qui hérite d'une autre classe :

```js
class Square extends Polygon {
  constructor(side) {
    super(side, side); // Appelle la méthode de la classe mère
    this.name = 'Carré';
  }

  get area() {
    return this.height * this.width;
  }
}

var s = new Square(5);
s.sayMyName(); // Coucou, je suis un Carré.
console.log(s.area); // 25
```

## let

Vous connaissez le mot clé `var`, qui permet de définir une variable globale, ou locale à une fonction. Le nouveau mot-clé `let` permet de définir une variable locale à un bloc :

```js
if (true) {
  var x = 1;
  let y = 1;
  console.log(x); // 1
  console.log(y); // 1
}

console.log(x); // 1
console.log(y); // ReferenceError
```

Contrairement à `var`, il n'y a pas de *hoisting* avec `let`, c'est-à-dire qu'une variable locale à un bloc doit être déclarée avant son utilisation. Dans le cas contraire, on obtient une `ReferenceError` :
```js
function foo() {
  bar = 2; // ReferenceError
  console.log(bar);
  let bar;
}

function foo2() {
  bar = 2;
  console.log(bar); // 2
  var bar;
}
```

En cas de redéclaration d'une variable déclarée avec `let`, on obtient une `TypeError` :
```js
if (x) {
  let bar;
  let bar; // TypeError
} 
```

## const

`const` permet de déclarer une variable, globale ou locale à une fonction, dont la valeur ne peut pas être modifiée après sa déclaration/initialisation (la variable est initialisée en même temps qu'elle est déclarée).

```js
function foo() {
  const bar = 2;
  bar = 3; // Erreur
}
```

## Itérateurs et générateurs

ES6 introduit des **itérateurs**, notion qu'on trouve dans d'autres langages de programmation. Un itérateur (*iterator*) est un objet qui dispose d'une méthode `next()`, permettant d'exécuter l'itération suivante, qui retourne un objet contenant deux propriétés : `value` et `done`  (qui indique si la séquence itérative est terminée).

Un itérateur peut être construit à l'aide d'un **générateur**, une fonction dont l'exécution peut être mise en pause grâce à une expression `yield`. Un itérateur construit par un générateur dispose aussi d'une méthode `next()` qui permet de poursuivre l'exécution de la fonction, et retourne un objet contenant deux propriétés : `value` (valeur retournée par l'expression `yield` suivante lors de l'exécution) et `done` (qui indique si l'exécution de la méthode est terminée).

```js
function* generator() {
  yield 0;
  yield 3;
  yield 10;
}

var gen = generator();

console.log(gen.next()); // {value: 0, done: false}
console.log(gen.next()); // {value: 3, done: false}
console.log(gen.next()); // {value: 10, done: false}
console.log(gen.next()); // {value: undefined, done: true}
```

Il est possible d'itérer sur un itérateur avec une boucle `for..of`, type de boucle introduit par ES6 :

```js
for (var i of generator()) {
  console.log(i);
}

// 0
// 3
// 10
```

Un objet **itérable** est un objet qui contient une méthode interne qui retourne un itérateur. Une boucle `for..of` permet d'itérer sur un tel objet :

```js
const iterable = {
  *[Symbol.iterator]() {
    yield 1;
    yield 3;
    yield 10;
  }
}

for (let x of iterable) {
  console.log(x);
}
// 1
// 3
// 10
```

## Promises

De nombreux développeurs JavaScript utilisent déjà des bibliothèques comme RSVP.js, qui implémentent ces fameuses promesses (*promises*). Ce sont des objets qui représentent le résultat d'une opération asynchrone, et elles feront bientôt nativement partie du langage.

```js
var async = function() {
  return new Promise(function(res, rej) {
    setTimeout(res, 1000);
  });
};

var after = function() {
  console.log('C\'est terminé !');
};

// Après 1 seconde... C'est terminé !
async().then(after);
```

## Arrow functions

Les *arrow functions* sont des expressions de fonctions anonymes définies par une syntaxe plus simple.

En ECMAScript 5 :

```js
var square = function(x) {
  return x * x;
};

console.log(square(5)); // 25
```

La même chose avec une *arrow function* en ES6 :

```js
var square = x => x * x;

console.log(square(5)); // 25
```

## Paramètres par défaut

ES6 permet de spécifier des valeurs par défaut pour les paramètres d'une fonction :

```js
function logMessage(msg='Message par défaut') {
  console.log(msg);
}

logMessage();
logMessage('Un autre message');
```

## Rest parameters

Les *rest parameters* permettent de définir un nombre variable de paramètres dans une fonction. C'est bien sûr déjà possible en itérant sur le contenu de la variable `arguments` :

```js
let bar = {
  drinks: []
};

bar.add = function() {
  var drinks = [].slice.call(arguments);
  drinks.forEach(function (drink) {
    bar.drinks.push(drink);
  });
};

bar.add('whisky');
bar.add('martini', 'vodka');
bar.add('rhum', 'pastis', 'water');

console.log(bar.drinks); // whisky,martini,vodka,rhum,pastis,water
```

Mais la syntaxe de *rest parameters* permet de l'écrire de façon un peu plus légère :
```js
let bar = {
  drinks: []
};

bar.add = function(...drinks) {
  drinks.forEach(function (drink) {
    bar.drinks.push(drink);
  });
};

bar.add('whisky');
bar.add('martini', 'vodka');
bar.add('rhum', 'pastis', 'water');

console.log(bar.drinks); // whisky,martini,vodka,rhum,pastis,water
```

Ici, le paramètre `drinks` est un tableau.

## Spread

L'opérateur *Spread* permet de passer en paramètre d'une fonction un tableau de paramètres :

```js
function add(a, b) {
    return a + b;
}

let nums = [5, 4];

console.log(add(...nums)); // 9
```

## Template strings

Les *template strings* apportent une nouvelle syntaxe pour définir et manipuler des chaînes de caractères (notez les back-ticks \`) :

```js
let string = `Coucou`;
```

Ces *template strings* permettent de faire de l'interpolation de chaînes, et d'embarquer des expressions (*embedded expressions*) :

```js
let cocktail = {name: 'mojito'};
let tpl = `Un ${cocktail.name}, je vous prie.`;

console.log(tpl); // Un mojito, je vous prie.
console.log(`J'en prendrai même ${1+1}.`); // J'en prendrai même 2
```
Ou encore écrire des chaînes de caractères sur plusieurs lignes :

```js
let string = `Voici
une chaîne sur deux lignes`; 
```

## Nouvelles structures de données

La spécification ECMAScript 6 ajoute plusieurs structures de données au langage : `Set`, `WeakSet`, `Map`, `WeakMap`.

### Set

Un `Set` est un ensemble de données sans doublon.

```js
let s = new Set([1, 2, 3, 3, 4]);

s.add('test');
s.delete(2);

console.log(s.size); // 4
console.log(s.has(1)); // true
```

Il est possible d'itérer sur un `Set` avec une boucle `for..of`, dans l'ordre dans lequel les éléments ont été ajoutés :

```js
for (let value of s) {
  console.log(value);
}

// 1
// 3
// 4
// test
```

### WeakSet

Un `WeakSet` est un `Set` qui contient uniquement des objets, ou plutôt des références faibles sur des objets, c'est-à-dire que ces références n'empêchent pas le nettoyage des objets par le Garbage Collector si aucune autre référence n'est présente.

```js
let obj1 = {name: 'obj1'};
let obj2 = {name: 'obj2'};

let s = new WeakSet();

s.add(obj1);
s.delete(obj2);

console.log(s.has(obj1)); // true
console.log(s.has(obj2)); // false
```

### Map

Une `Map` est un tableau associatif (association clés-valeurs).

```js
let myMap = new Map();

let key1 = 'cle1';
let key2 = 2;
let key3 = {};

myMap.set(key1, 'Value 1');
myMap.set(key2, 'Value 2');
myMap.set(key3, 'Value 3');

console.log(myMap.size); // 3

console.log(myMap.get(key1)); // Value 1
console.log(myMap.get(key2)); // Value 2
console.log(myMap.get(key3)); // Value 3
```

### WeakMap

Une `WeakMap` est une `Map` dont les clés sont uniquement des objets, ou plutôt des références faibles sur des objets, c'est-à-dire que ces références n'empêchent pas le nettoyage des objets par le Garbage Collector si aucune autre référence n'est présente.

Contrairement à une `Map`, une `WeakMap` n'est pas énumérable et n'a pas de propriété `size`.

```js
let obj1 = {name: 'myObject'};

let m = new WeakMap();
m.set(obj1, 'value1');

console.log(m.has(obj1)); // true
console.log(m.get(obj1)); // value1

m.delete(obj1);

console.log(m.has(obj1)); // false
console.log(m.get(obj1)); // undefined
```

## Conclusion

Comme vous pouvez le voir, ECMAScript 6 apporte un gros lot de nouveautés destiné à faciliter l'écriture du langage, en reprenant des possibilités présentes dans d'autres langages de programmation. Pour savoir si vous pouvez utiliser ces nouveautés, consultez [cette table de compatibilité](http://kangax.github.io/compat-table/es6/). En attendant que tous les moteurs Javascript soient compatibles, notez l'existence de la bibliothèque [Traceur](https://github.com/google/traceur-compiler), qui permet de transformer du code ES5.1 en code ES6 (attention, toutes les fonctionnalités ES6 ne sont pas compatibles). Enfin, [ES6 Fiddle](http://www.es6fiddle.net/) permet de tester des bouts de code ES6.
