# Introducing JSX

## Introduction

Considérez cette déclaration de variable:

```javascript
const element = <h1>Hello, world!</h1>;
```

Cette syntaxe n'est pas une string ou un du html mais du JSX. C'est une extension de syntaxe pour JavaScript. JSX produit des "éléments" React. Nous allons explorer les rendu au DOM dans la section suivante. Ci-dessous, vous trouverez les bases de JSX nécessaires pour vous aider à démarrer.

React utilise la logique que le rendu est intrinsèquement couplée à la logique UI et donc les deux sont inclus dans un modèle composant.

## Embedding Expressions in JSX

Dans l'exemple ci-dessous, nous déclarons une variable appelée nom, puis l'utilisons dans JSX en l'enveloppant dans des accolades :

```javascript
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

Vous pouvez mettre n'importe quelle expression JavaScript valide à l'intérieur des accolades dans JSX. Par exemple, 2 + 2, user.firstName ou formatName (utilisateur) sont toutes des expressions JavaScript valides.

Dans l'exemple ci-dessous, nous intégrons le résultat de l'appel d'une fonction JavaScript formatName (utilisateur) dans un élément **h1**.

```javascript
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

const element = (
  <h1>
    Hello, {formatName(user)}!
  </h1>
);

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

Nous divisons JSX sur plusieurs lignes pour plus de lisibilité. Bien que ce ne soit pas obligatoire, nous recommandons également de l'entourer de parenthèses pour éviter les pièges de l'insertion automatique de points-virgules.

## JSX est une expression trop

Après la compilation, les expressions JSX deviennent des appels de fonction JavaScript réguliers et évaluent les objets JavaScript. Cela signifie que vous pouvez utiliser JSX à l'intérieur des instructions if et for loops, l'assigner à des variables, l'accepter comme argument, et le renvoyer des fonctions:

```javascript
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
```

## Specifying Attributes with JSX

vous pouvez utiliser des guillemets pour spécifier des littéraux de chaîne en tant qu'attributs:

```javascript
const element = <div tabIndex="0"></div>;
```

Vous pouvez également utiliser des accolades pour incorporer une expression JavaScript dans un attribut:

```javascript
const element = <img src={user.avatarUrl}></img>;
```

Ne placez pas de guillemets autour des accolades lorsque vous intégrez une expression JavaScript dans un attribut. Vous devez utiliser des guillemets (pour les valeurs de chaîne) ou des accolades (pour les expressions), mais pas les deux dans le même attribut.

Comme JSX est plus proche du JavaScript que du HTML, React DOM utilise la convention de dénomination de la propriété camelCase au lieu des noms d'attributs HTML.
Par exemple, **class** devient **className** dans JSX et **tabindex** devient **tabIndex**.

## Specifying Children with JSX

Si une balise est vide, vous pouvez la fermer immédiatement avec />, comme XML

```javascript
const element = <img src={user.avatarUrl} />;
```

Les balises JSX peuvent contenir des enfants:

```javascript
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

## JSX Prevents Injection Attacks

Il est sûr d'intégrer l'entrée de l'utilisateur dans JSX:

```javascript
const title = response.potentiallyMaliciousInput;
// This is safe:
const element = <h1>{title}</h1>;
```

Par défaut, React DOM échappe toutes les valeurs incorporées dans JSX avant de les rendre. Ainsi, il garantit que vous ne pouvez jamais injecter quelque chose qui n'est pas explicitement écrit dans votre application. Tout est converti en chaîne avant d'être rendu. Cela permet d'éviter les attaques XSS (cross-site-scripting).

## JSX Represents Objects

Babel compile JSX vers les appels React.createElement(). Ces deux exemples sont identiques:

```javascript
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```

```javascript
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

React.createElement() effectue quelques vérifications pour vous aider à écrire du code sans bug, mais il crée essentiellement un objet comme celui-ci :

```javascript
// Note: this structure is simplified
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```

Ces objets sont appelés **React elements**. Vous pouvez les considérer comme des descriptions de ce que vous voulez voir à l'écran. React lit ces objets et les utilise pour construire le DOM et le tenir à jour. Nous explorerons le rendu des éléments React dans le DOM dans la section suivante.