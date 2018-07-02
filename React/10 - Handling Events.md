# Handling Events

## Introduction

La gestion des événements avec les éléments React est très similaire à la gestion des événements sur les éléments DOM. Il y a quelques différences syntaxiques:

Les événements React sont nommés à l'aide de camelCase plutôt qu'en minuscules.
Avec JSX vous passez une fonction en tant que gestionnaire d'événements, plutôt qu'une chaîne.

For example, the HTML:

```javascript
<button onclick="activateLasers()">
  Activate Lasers
</button>
```

est légèrement différent dans React:

```javascript
<button onClick={activateLasers}>
  Activate Lasers
</button>
```

Une autre différence est que vous ne pouvez pas retourner false pour empêcher le comportement par défaut dans React. Vous devez appeler preventDefault explicitement. Par exemple, avec du code HTML brut, pour empêcher le comportement de lien par défaut d'ouvrir une nouvelle page, vous pouvez écrire:

```javascript
<a href="#" onclick="console.log('The link was clicked.'); return false">
  Click me
</a>
```

Dans React, cela pourrait être plutôt:

```javascript
function ActionLink() {
  function handleClick(e) {
    e.preventDefault();
    console.log('The link was clicked.');
  }

  return (
    <a href="#" onClick={handleClick}>
      Click me
    </a>
  );
}
```

Ici, e est un événement synthétique. React définit ces événements synthétiques en fonction de la spécification W3C, vous n'avez donc pas besoin de vous soucier de la compatibilité entre navigateurs.

Lorsque vous utilisez React, vous n'avez généralement pas besoin d'appeler addEventListener pour ajouter des listener à un élément DOM après sa création. Au lieu de cela, fournissez simplement un listener lorsque l'élément est rendu initialement.

Lorsque vous définissez un composant à l'aide d'une classe ES6, un modèle courant consiste à utiliser un gestionnaire d'événements comme méthode sur la classe. Par exemple, ce composant Toggle affiche un bouton qui permet à l'utilisateur de basculer entre les états "ON" et "OFF":

```javascript
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // This binding is necessary to make `this` work in the callback
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}

ReactDOM.render(
  <Toggle />,
  document.getElementById('root')
);
```

Vous devez faire attention à la signification de ceci dans les rappels JSX. En JavaScript, les méthodes de classe ne sont pas liées par défaut. Si vous oubliez de lier this.handleClick et de le passer à onClick, cela ne sera pas défini lorsque la fonction sera appelée.

Ce comportement n'est pas spécifique à React; C'est une partie de la façon dont les fonctions fonctionnent en JavaScript. Généralement, si vous faites référence à une méthode sans() après, comme onClick = {this.handleClick}, vous devez lier cette méthode.

Si l'appel de bind vous ennuie, il y a deux façons de contourner ce problème. Si vous utilisez la ***experimental public class fields syntax*** vous pouvez utiliser les champs de classe pour lier correctement les rappels:

```javascript
class LoggingButton extends React.Component {
  // This syntax ensures `this` is bound within handleClick.
  // Warning: this is *experimental* syntax.
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```

Cette syntaxe est activée par défaut dans Create React App.

Si vous n'utilisez pas la ***experimental public class fields syntax***, vous pouvez utiliser une arrow function dans le rappel:

```javascript
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // This syntax ensures `this` is bound within handleClick
    return (
      <button onClick={(e) => this.handleClick(e)}>
        Click me
      </button>
    );
  }
}
```

Le problème avec cette syntaxe est qu'un callback différent est créé chaque fois que le LoggingButton est rendu. Dans la plupssart des cas, c'est bien. Toutefois, si ce rappel est passé en tant que prop pour réduire les composants, ces composants peuvent effectuer un nouveau rendu. Nous recommandons généralement de lier dans le constructeur ou d'utiliser la ***experimental public class fields syntax***, pour éviter ce genre de problème de performance.

## Passing Arguments to Event Handlers

Dans une boucle, il est courant de vouloir passer un paramètre supplémentaire à un event handler. Par exemple, si id est l'ID de ligne, l'une des solutions suivantes fonctionnera:

```javascript
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

Les deux lignes ci-dessus sont équivalentes et utilisent respectivement les arrow functions et Function.prototype.bind.

Dans les deux cas, l'argument "e" représentant l'événement React sera passé en second argument après l'ID. Avec une fonction flèche, nous devons le passer explicitement, mais avec bind, tous les autres arguments sont automatiquement transférés.