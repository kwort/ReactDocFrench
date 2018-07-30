# Components and Props

## Introduction

Les composants vous permettent de diviser l'interface utilisateur en éléments indépendants et réutilisables, et de penser à chaque élément séparément. Cette page fournit une introduction à l'idée des composants.

Conceptuellement, les composants sont comme des fonctions JavaScript. Ils acceptent des entrées arbitraires (appelées "props") et renvoient des éléments React décrivant ce qui devrait apparaître à l'écran.

## Composants fonctionnels et de classe

La façon la plus simple de définir un composant est d'écrire une fonction JavaScript:

```javascript
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

Cette fonction est un composant React valide car elle accepte un seul argument d'objet "props" (qui correspond aux propriétés) avec des données et renvoie un élément React. Nous appelons ces composants "functional" car ils sont littéralement des fonctions JavaScript.

Vous pouvez également utiliser une classe ES6 pour définir un composant:

```javascript
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

Les deux composants ci-dessus sont équivalents du point de vue de React. Les classes ont des fonctionnalités supplémentaires dont nous parlerons dans les sections suivantes. Jusque-là, nous utiliserons des composants fonctionnels pour leur concision.

## Rendu d'un composant

Auparavant, nous ne rencontrions que des éléments React représentant des balises DOM:

```javascript
const element = <div />;
```

Cependant, les éléments peuvent également représenter des composants définis par l'utilisateur:

```javascript
const element = <Welcome name="Sara" />;
```

Lorsque React voit un élément représentant un composant défini par l'utilisateur, il transmet les attributs JSX à ce composant en tant qu'objet unique. Nous appelons cet objet "props". Par exemple, ce code affiche "Hello, Sara" sur la page:

```javascript
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

Récapitulons ce qui se passe dans cet exemple:

- Nous appelons ReactDOM.render() avec l'élément **Welcome name = "Sara"**.
- React appelle le composant Welcome avec {name: 'Sara'} comme props.
- Notre composant Welcome renvoie un élément **h1** Hello, Sara comme résultat.
- React DOM met à jour efficacement le DOM pour qu'il corresponde à **h1** Hello, Sara

Remarque: Commencez toujours les noms de composants avec une lettre majuscule. React traite les composants commençant par des minuscules comme des elements DOM.

## Composer des composants

Les composants peuvent se référer à d'autres composants dans leur sortie. Cela nous permet d'utiliser la même abstraction de composant pour n'importe quel niveau de détail. Un bouton, un formulaire, une boîte de dialogue, un écran: dans les applications React, tous ces éléments sont généralement exprimés en tant que composants.

Par exemple, nous pouvons créer un composant App qui rend plusieurs fois Welcome:

```javascript
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

En règle générale, les nouvelles applications React ont un seul composant App à la racine.

## Extraction de composants

N'ayez pas peur de diviser les composants en composants plus petits. Par exemple, considérez ce composant Commentaire:

```javascript
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

Il accepte l'auteur (un objet), le texte (une chaîne) et la date (une date) comme props, et décrit un commentaire sur un site Web de médias sociaux. Ce composant peut être difficile à modifier à cause de tous les imbrications, et il est également difficile de réutiliser des parties individuelles de celui-ci. En extrayons quelques composants.

Premièrement, nous allons extraire Avatar:

```javascript
function Avatar(props) {
  return (
    <img className="Avatar"
      src={props.user.avatarUrl}
      alt={props.user.name}
    />

  );
}
```

L'Avatar n'a pas besoin de savoir qu'il est rendu dans un commentaire. C'est pourquoi nous avons donné à son prop un nom plus générique: user plutôt que author. Nous recommandons de nommer les props du point de vue du composant plutôt que du contexte dans lequel il est utilisé. Nous pouvons maintenant simplifier un peu le commentaire:

```javascript
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <Avatar user={props.author} />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

Ensuite, nous allons extraire un composant UserInfo qui rend un Avatar à côté du nom de l'utilisateur:

```javascript
function UserInfo(props) {
  return (
    <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name">
        {props.user.name}
      </div>
    </div>
  );
}
```

Cela nous permet de simplifier encore plus le commentaire:

```javascript
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

L'extraction de composants peut sembler un travail fastidieux au début, mais avoir une palette de composants réutilisables est payant dans les applications plus volumineuses. En règle générale, si une partie de votre interface utilisateur est utilisée plusieurs fois (bouton, panneau, avatar) ou si elle est suffisamment complexe (App, FeedStory, Comment), elle peut être un composant réutilisable.

## Les props sont en lecture seule

Que vous déclariez un composant en tant que fonction ou classe, il ne doit jamais modifier ses propres props. Considérez cette fonction de somme:

```javascript
function sum(a, b) {
  return a + b;
}
```

De telles fonctions sont appelées "pure" car elles n'essaient pas de changer leurs entrées et retournent toujours le même résultat pour les mêmes entrées.

En revanche, cette fonction est impure car elle change sa propre entrée:

```javascript
function withdraw(account, amount) {
  account.total -= amount;
}
```

React est assez flexible mais il a une seule règle stricte:

Tous les composants React doivent agir comme des fonctions pures par rapport à leurs props.

Bien sûr, les interfaces utilisateur des applications sont dynamiques et évoluent avec le temps. Dans la section suivante, nous introduirons un nouveau concept de **state**. Le state est l'état permet aux composants React de modifier leur sortie au fil du temps en réponse aux actions de l'utilisateur, aux réponses réseau et à toute autre chose, sans enfreindre cette règle.