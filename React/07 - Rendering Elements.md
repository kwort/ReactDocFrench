# Rendering Elements

## Introduction

Les éléments sont les plus petits blocs de construction des applications React.

Un élément décrit ce que vous voulez voir à l'écran:

```javascript
const element = <h1>Hello, world</h1>;
```

Contrairement aux éléments DOM des navigateurs, les éléments React sont des objets simples et peu coûteux à créer. React DOM prend soin de mettre à jour le DOM pour qu'il corresponde aux éléments React. On pourrait confondre des éléments avec un concept plus connu de «composants». Nous présenterons les composants dans la section suivante. Les éléments sont ce que les composants sont «faits de», et nous vous encourageons à lire cette section avant de sauter le pas.

## Rendering an Element into the DOM

Disons qu'il y a un *div* quelque part dans votre fichier HTML:

```html
<div id="root"> </ div>
```

Nous appelons cela un noeud DOM «root» car tout ce qui est à l'intérieur sera géré par React DOM. Les applications construites avec simplement React ont généralement un seul noeud DOM racine. Si vous intégrez React dans une application existante, vous pouvez avoir autant de nœuds DOM racine isolés que vous le souhaitez.

Pour rendre un élément React dans un noeud DOM racine, passez les deux à ReactDOM.render():

```javascript
const element = <h1>Hello, world</ h1>;
ReactDOM.render (element, document.getElementById ('root'));
```

Il affiche "Hello, world" sur la page.

## Updating the Rendered Element

Les éléments React sont immuables. Une fois que vous créez un élément, vous ne pouvez pas modifier ses enfants ou ses attributs.
Avec nos connaissances à ce jour, la seule façon de mettre à jour l'interface utilisateur est de créer un nouvel élément et de le passer à ReactDOM.render().

Considérez cet exemple d'horloge :

```javascript
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(element, document.getElementById('root'));
}

setInterval(tick, 1000);
```

Il appelle ReactDOM.render() chaque seconde à partir callback setInterval ().

En pratique, la plupart des applications React n'appellent qu'une seule fois ReactDOM.render()). Dans les sections suivantes, nous allons apprendre comment un tel code est encapsulé dans des composants avec state.

## React Only Updates What’s Necessary

React DOM compare l'élément et ses enfants avec le précédent, et n'applique que les mises à jour DOM nécessaires pour amener le DOM à l'état souhaité.
Vous pouvez vérifier en inspectant le dernier exemple avec les outils du navigateur:

Même si nous créons un élément décrivant toute l'arborescence de l'interface utilisateur à chaque tick, seul le nœud de texte dont le contenu a été modifié est mis à jour par le DOM REACT. Dans notre expérience, penser à la façon dont l'interface utilisateur devrait regarder un moment donné plutôt que comment le changer au fil du temps élimine toute une classe de bugs.