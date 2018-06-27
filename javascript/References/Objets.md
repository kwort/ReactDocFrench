# Objets globaux standards (par catégorie)

## Propriétés - valeurs
Les propriétés globales renvoient une valeur simple, elles ne possèdent aucune propriété ou méthode :

### Infinity
La propriété globale Infinity est une valeur numérique représentant l'infini.

### NaN
La propriété globale NaN est une valeur utilisée pour représenter une quantité qui n'est pas un nombre (Not a Number en anglais).

### undefined
La propriété globale undefined représente la valeur undefined. Cette valeur est l'un des types primitifs de JavaScript.

### null
La valeur null est un littéral JavaScript représentant la nullité au sens où aucune valeur pour l'objet n'est présente. C'est une des valeurs primitives de JavaScript.

## Propriétés - fonctions
Les fonctions globales, appelées globalement (et non par rapport à un objet), renvoient directement leur résultat à l'objet appelant.

### eval()
La fonction eval() permet d'évaluer du code JavaScript représenté sous forme d'une chaîne de caractères.

### uneval()
La fonction uneval() renvoie une chaîne de caractères représentant le code source d'un objet.

### isFinite()
La fonction globale isFinite() détermine si la valeur passée en argument est un nombre fini. Si nécessaire, le paramètre est d'abord converti en nombre.

### isNaN()
La fonction isNaN() permet de déterminer si une valeur est NaN. On notera que cette fonction utilise des règles de conversion différentes de Number.isNaN(), défini avec ECMAScript 2015 (ES6).

### parseFloat()
La fonction parseFloat() permet de transformer une chaîne de caractères en un nombre flottant après avoir analysée celle-ci (parsing).

### parseInt()
La fonction parseInt() analyse une chaîne de caractère fournie en argument et renvoie un entier exprimé dans une base donnée.

### decodeURI()
La méthode decodeURI() permet de décoder un Uniform Resource Identifier (URI) créé par la méthode encodeURI ou une méthode similaire.

### decodeURIComponent()
La fonction decodeURIComponent() permet de décoder un composant d'un Uniform Resource Identifier (URI) précédemment créé par encodeURIComponent ou par une méthode similaire.

### encodeURI()
La fonction encodeURI() encode un Uniform Resource Identifier (URI) en remplaçant chaque exemplaire de certains caractères par une, deux, trois ou quatre séquences d'échappement représentant le caractère encodé en UTF-8 (les quatre séquences d'échappement ne seront utilisées que si le caractère est composé de deux caractères « surrogate »).

### encodeURIComponent()
La fonction encodeURIComponent() permet d'encoder un composant d'un Uniform Resource Identifier (URI) en remplaçant chaque exemplaire de certains caractères par une, deux, trois ou quatres séquences d'échappement UTF-8 correspondantes (quatre séquences seront utilisées uniquement lorsque les caractères à encoder sont composés de deux caractères « surrogate »).

### escape()
La fonction escape() permet de renvoyer une nouvelle chaîne de caractères dont certains caractères ont été remplacés par leur séquence d'échappement hexadécimale. Cette méthode est obsolète et il est donc conseillé d'utiliser encodeURI ou encodeURIComponent à la place.

### unescape() 
La fonction dépréciée unescape() calcule une nouvelle chaîne de caractères et remplace les séquences d'échappement hexadécimales par les caractères qu'elles représentent. Les séquences d'échappement peuvent provenir de la fonction escape. Cette méthode est obsolète, c'est pourquoi il est conseillé d'utiliser decodeURI ou decodeURIComponent à la place.

## Objets fondamentaux
Ces objets sont les objets fondamentaux de JavaScript. Parmi ces objets, on retrouve les objets génériques, les fonctions et les erreurs.

### Object
Le constructeur Object crée un wrapper d'objet.

### Function
Le constructeur Function crée un nouvel objet Function. En JavaScript, chaque fonction est un objet Function.

Appeler ce constructeur permet de créer des fonctions dynamiquement mais cette méthode souffre de défauts équivalents que eval en termes de sécurité et de performance. Toutefois, à la différence d'eval, le constructeur Function permet d'exécuter du code dans la portée globale.

### Boolean
L'objet Boolean est un objet permettant de représenter une valeur booléenne.

### Symbol
Un symbole est un type de données unique et inchangeable qui peut être utilisé pour représenter des identifiants pour des propriétés d'un objet. L'objet Symbol est un conteneur objet implicite pour le type de données primitif symbole.

### Error
Le constructeur Error crée un objet d'erreur. Des instances d'objets Error sont déclenchées lorsque des erreurs d'exécution surviennent. L'objet Error peut aussi être utilisé comme objet de base pour des exceptions définies par l'utilisateur. Voir ci-dessous pour les types d'erreur natifs standard.

### EvalError
L'objet EvalError indique une erreur concernant la fonction globale eval(). Cette exception n'est plus levée par JavaScript mais l'objet EvalError est conservé pour des raisons de compatibilité.

### InternalError
L'objet InternalError indique qu'une erreur liée au moteur JavaScript s'est produite. Par exemple "InternalError : Niveau de récursion trop important".

### RangeError
L'objet RangeError permet d'indiquer une erreur lorsqu'une valeur fournie n'appartient pas à l'intervalle autorisé.

### ReferenceError
L'objet ReferenceError représente une erreur qui se produit lorsqu'il est fait référence à une variable qui n'existe pas.

### StopIteration
L'objet StopIteration est une exception levée lorsque l'on cherche à accéder au prochain élément d'un itérateur épuisé et implémentant le protocole itérateur historique.

### SyntaxError
L'objet SyntaxError représente une erreur qui se produit lors de l'interprétation d'un code dont la syntaxe est invalide.

### TypeError
L'objet TypeError représente une erreur qui intervient lorsque la valeur n'est pas du type attendu.

### URIError
L'objet URIError représente une erreur renvoyée lorsqu'une fonction de manipulation d'URI a été utilisée de façon inappropriée.

## Nombres et dates
Ces objets permettent de manipuler les nombres, dates et calculs mathématiques.

### Number
L'objet Number est une enveloppe objet (wrapper) autour du type primitif numérique. Autrement dit, il est utilisé pour manipuler les nombres comme des objets. Pour créer un objet Number, on utilise le constructeur Number().

### Math
L'objet Math est un objet natif dont les méthodes et propriétés permettent l'utilisation de constantes et fonctions mathématiques. Cet objet n'est pas une fonction.

### Date
Ce constructeur permet de créer des instances Date qui représentent un moment précis dans le temps. Les objets Date se basent sur une valeur de temps qui est le nombre de millisecondes depuis 1er janvier 1970 minuit UTC.

## Manipulation de textes
Ces objets permettent de manipuler des chaînes de caractères.

### String
L'objet global String est un constructeur de chaînes de caractères.

### RegExp
Le constructeur RegExp crée un objet expression rationnelle pour la reconnaissance d'un modèle dans un texte.

## Collections indexées
Ces objets sont des collections ordonnées par un index. Cela inclut les tableaux (typés) et les objets semblables aux tableaux.

### Array
L'objet global Array est utilisé pour créer des tableaux. Les tableaux sont des objets de haut-niveau (en termes de complexité homme-machine) semblables à des listes.

### Int8Array
Le tableau typé Int8Array permet de représenter un tableau d'entiers signés (en complément à deux) représentés sur 8 bits. Les éléments du tableau sont initialisés à 0. Une fois le tableau construit, il est possible de faire référence aux éléments en utilisant les méthodes de l'objet ou en utilisant la notation usuelle de parcours d'un tableau (la syntaxe utilisant les crochets).

### Uint8Array
### Uint8ClampedArray
### Int16Array
### Uint16Array
### Int32Array
### Uint32Array
### Float32Array
### Float64Array

## Collections avec clefs
Ces objets représentent des collections d'objets avec clefs. Ils contiennent des éléments itérables, dans leur ordre d'insertion.

### Map
### Set
### WeakMap
### WeakSet

## Collections vectorielles
Les types de données vectoriels SIMD sont des objets dont les données sont organisées de façon linéaire :

### SIMD 
### SIMD.Float32x4 
### SIMD.Float64x2 
### SIMD.Int8x16 
### SIMD.Int16x8 
### SIMD.Int32x4 
### SIMD.Uint8x16 
### SIMD.Uint16x8 
### SIMD.Uint32x4 
### SIMD.Bool8x16 
### SIMD.Bool16x8 
### SIMD.Bool32x4 
### SIMD.Bool64x2 

## Données structurées
Ces objets permettent de représenter et de manipuler des tampons de données (buffers) et des données utilisant la notation JSON (JavaScript Object Notation).

### ArrayBuffer
### SharedArrayBuffer 
### Atomics 
### DataView
### JSON

## Objets de contrôle d'abstraction

### Promise
### Generator
### GeneratorFunction
### AsyncFunction

## Introspection

### Reflect
### Proxy

## Internationalisation
Ces objets ont été ajoutés à ECMAScript pour des traitements dépendants de particularités linguistiques. Ils possèdent leur propre spécification.

### Intl
### Intl.Collator
### Intl.DateTimeFormat
### Intl.NumberFormat

## WebAssembly

### WebAssembly
### WebAssembly.Module
### WebAssembly.Instance
### WebAssembly.Memory
### WebAssembly.Table
### WebAssembly.CompileError
### WebAssembly.LinkError
### WebAssembly.RuntimeError

## Autres

### arguments