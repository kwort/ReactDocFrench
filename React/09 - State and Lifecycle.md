# State and Lifecycle

## Introduction

Considérons l'exemple de l'horloge. Dans Rendering Elements, nous n'avons appris qu'une seule façon de mettre à jour l'interface utilisateur. Nous appelons ReactDOM.render() pour changer la sortie rendue:

```javascript
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

Dans cette section, nous allons apprendre comment rendre le composant Clock vraiment réutilisable et encapsulé. Il va configurer son propre minuteur et se mettre à jour toutes les secondes. Nous pouvons commencer par encapsuler l'apparence de l'horloge:

```javascript
function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Clock date={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

Cependant, il manque une exigence cruciale: le fait que l'horloge configure une minuterie et met à jour l'interface utilisateur chaque seconde devrait être un détail d'implémentation de l'horloge. Idéalement, nous voulons écrire cela une fois et avoir la mise à jour de l'horloge elle-même:

```javascript
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

Pour implémenter cela, nous devons ajouter "state" au composant Clock.
Le state est similaire aux props, mais il est privé et entièrement contrôlé par le composant.

Nous avons mentionné précédemment que les composants définis en tant que classes ont des fonctionnalités supplémentaires. L'état local est exactement cela: une fonctionnalité disponible uniquement pour les classes.

## Converting a Function to a Class

Vous pouvez convertir un composant fonctionnel comme Clock en cinq classes:

- Créez une classe ES6, avec le même nom, qui étend React.Component.
- Ajoutez-y une méthode vide appelée render().
- Déplacez le corps de la fonction dans la méthode render().
- Remplacez les props par this.props dans le corps render().
- Supprime la déclaration de fonction vide restante.

```javascript
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

L'horloge est maintenant définie comme une classe plutôt qu'une fonction.

La méthode de rendu sera appelée chaque fois qu'une mise à jour se produit, mais tant que nous restituons **Clock** dans le même noeud DOM, seule une instance de la classe Clock sera utilisée. Cela nous permet d'utiliser des fonctionnalités supplémentaires telles que le ***local state*** et les ***lifecycle hooks***.

## Adding Local State to a Class

Nous déplacerons la date des props par le state en trois étapes:

Remplacez this.props.date par this.state.date dans la méthode render():

```javascript
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

Ajouter la classe constructeur qui assigne le state initial à this.state :

```javascript
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

Notez que la les props passe maintenant par le constructeur :

```javascript
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }
```

Les composants de classe doivent toujours appeler le constructeur de base avec des props.
Supprimez les props de date de l'élément **Clock**:

```javascript
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);

```

Nous ajouterons plus tard le code de temporisateur au composant lui-même.

Le résultat ressemble à ceci:

```javascript
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

Ensuite, nous ferons en sorte que l'horloge configure son propre minuteur et se mette à jour toutes les secondes.

## Adding Lifecycle Methods to a Class

Dans les applications comportant de nombreux composants, il est très important de libérer les ressources prises par les composants lorsqu'ils sont détruits.
Nous voulons mettre en place une minuterie à chaque fois que l'horloge est rendue au DOM pour la première fois. C'est ce qu'on appelle "mounting" dans React.
Nous voulons également effacer cette minuterie à chaque fois que le DOM produit par l'horloge est supprimé. C'est ce qu'on appelle "unmounting" dans React.
Nous pouvons déclarer des méthodes spéciales sur la classe de composant pour exécuter du code quand un composant mounts et unmounts:

```javascript
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
      // mounnt
  }

  componentWillUnmount() {
      // unMounnt
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

Ces méthodes sont appelées “lifecycle hooks”.

Le hook componentDidMoun () s'exécute après que la sortie du composant a été rendue au DOM. C'est un bon endroit pour configurer une minuterie:

```javascript
  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }
```

Note how we save the timer ID right on this.

While this.props is set up by React itself and this.state has a special meaning, you are free to add additional fields to the class manually if you need to store something that doesn’t participate in the data flow (like a timer ID).

We will tear down the timer in the componentWillUnmount() lifecycle hook:

---

Notez comment nous enregistrons l'identifiant de la minuterie sur ce point.

Bien que this.props soit configuré par React lui-même et que this.state ait une signification particulière, vous êtes libre d'ajouter manuellement des champs supplémentaires à la classe si vous avez besoin de stocker quelque chose qui ne participe pas au flux de données (comme un timer ID).

Nous allons supprimer le minuteur dans le lifecycle hook componentWillUnmount():

```javascript
  componentWillUnmount() {
    clearInterval(this.timerID);
  }
```

Enfin, nous allons implémenter une méthode appelée tick() que le composant Clock va lancer toutes les secondes.
Il utilisera this.setState() pour planifier les mises à jour de l'état local du composant:

```javascript
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

Maintenant, l'horloge tourne toutes les secondes. Récapitulons rapidement ce qui se passe et l'ordre dans lequel les méthodes sont appelées:

- Lorsque **Clock** est transmis à ReactDOM.render(), React appelle le constructeur du composant Clock. Comme Clock doit afficher l'heure actuelle, il initialise this.state avec un objet incluant l'heure actuelle. Nous mettrons plus tard à jour cet état.

- React appelle ensuite la méthode render() du composant Clock. C'est ainsi que React apprend ce qui devrait être affiché sur l'écran. Réactualiser puis met à jour le DOM pour correspondre à la sortie de rendu de l'horloge.

- Lorsque la sortie Clock est insérée dans le DOM, React appelle le hook lifecycle componentDidMount(). A l'intérieur, le composant Clock demande au navigateur de configurer un temporisateur pour appeler la méthode tick() du composant une fois par seconde.

- Chaque seconde, le navigateur appelle la méthode tick(). À l'intérieur, le composant Horloge planifie une mise à jour de l'interface utilisateur en appelant setState() avec un objet contenant l'heure actuelle. Grâce à l'appel setState(), React sait que le state a changé et appelle à nouveau la méthode render() pour savoir ce qui devrait être à l'écran. Cette fois, this.state.date dans la méthode render() sera différent, et donc la sortie de rendu inclura l'heure mise à jour. React met à jour le DOM en conséquence.

- Si le composant Clock est jamais supprimé du DOM, React appelle le hook de cycle de vie componentWillUnmount() afin que le temporisateur soit arrêté.

## Using State Correctly

Il y a trois choses à savoir sur setState ().

### Do Not Modify State Directly

Par exemple, cela ne ré-affichera pas un composant:

```javascript
// Wrong
this.state.comment = 'Hello';
Instead, use setState():
```

mais celui-ci le fera :

```javascript
// Correct
this.setState({comment: 'Hello'});
```

Le seul endroit où vous pouvez assigner this.state est dans le constructeur.

### State Updates May Be Asynchronous

React peut traiter plusieurs appels setState() en une seule mise à jour pour les performances.
Puisque this.props et this.state peuvent être mis à jour de manière asynchrone, vous ne devez pas compter sur leurs valeurs pour calculer l'état suivant.
Par exemple, ce code peut échouer à mettre à jour le compteur:

```javascript
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

Utilisez une seconde forme de setState() qui accepte une fonction plutôt qu'un objet. Cette fonction recevra le state précédent en tant que premier argument, et les props au moment où la mise à jour est appliquée en tant que second argument:

```javascript
// Correct
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));
```

Nous avons utilisé une arrow function ci-dessus, mais cela fonctionne également avec des fonctions régulières:

```javascript
// Correct
this.setState(function(prevState, props) {
  return {
    counter: prevState.counter + props.increment
  };
});
```

### State Updates are Merged

Lorsque vous appelez setState(), React fusionne l'objet que vous fournissez dans l'état actuel.
Par exemple, votre état peut contenir plusieurs variables indépendantes:

```javascript
  constructor(props) {
    super(props);
    this.state = {
      posts: [],
      comments: []
    };
  }
```

Vous pouvez ensuite les mettre à jour indépendamment avec des appels setState() séparés:

```javascript
  componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts
      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments
      });
    });
  }
```

La fusion est peu profonde, donc this.setState ({comments}) laisse intacts this.state.posts, mais remplace complètement this.state.comments.

## The Data Flows Down

Ni les composants parents ni les composants enfant ne peuvent savoir si un certain composant est à état ou sans état, et ils ne devraient pas se soucier de savoir s'il est défini en tant que fonction ou classe. C'est pourquoi l'état est souvent appelé local ou encapsulé. Il n'est accessible à aucun composant autre que celui qui le possède et le définit. Un composant peut choisir de passer son état en tant que props à ses composants enfants:

```javascript
<h2>It is {this.state.date.toLocaleTimeString()}.</h2>
```

Cela fonctionne également pour les composants définis par l'utilisateur:

```javascript
<FormattedDate date={this.state.date} />
```

Le composant FormattedDate recevrait la date dans ses props et ne saurait pas s'il venait de du state de l'horloge, des props de l'horloge, ou était tapé à la main:

```javascript
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```

Ceci est communément appelé flux de données "top-down" ou "unidirectionnel". N'importe quel état est toujours détenu par un composant spécifique, et toute donnée ou interface utilisateur dérivée de cet état ne peut affecter que les composants "au-dessous" d'eux dans l'arborescence.

Pour montrer que tous les composants sont vraiment isolés, nous pouvons créer un composant App qui rend trois ***Clock*** :

```javascript
function App() {
  return (
    <div>
      <Clock />
      <Clock />
      <Clock />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

Chaque horloge met en place son propre minuteur et se met à jour indépendamment.

Dans les applications React, le fait qu'un composant soit dynamique ou sans état est considéré comme un détail d'implémentation du composant susceptible de changer avec le temps. Vous pouvez utiliser des composants sans state à l'intérieur de composants avec state, et vice versa.