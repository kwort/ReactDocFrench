# Conditional rendering

## Introduction

Dans React, vous pouvez créer des composants distincts qui encapsulent le comportement dont vous avez besoin. Ensuite, vous ne pouvez en afficher que quelques-uns, en fonction de l'état de votre application.

Le rendu conditionnel dans React fonctionne de la même manière que les conditions fonctionnent dans JavaScript. Utilisez les opérateurs JavaScript comme [`if`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else) ou le [conditional operator](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) pour créer des éléments représentant l'état actuel, et laissez React mettre à jour l'interface utilisateur pour les faire correspondre.

Consider these two components:

```js
function UserGreeting(props) {
  return <h1>Welcome back!</h1>;
}

function GuestGreeting(props) {
  return <h1>Please sign up.</h1>;
}
```

Nous allons créer un composant `Greeting` qui affiche l'un de ces composants selon que l'utilisateur est connecté:

```javascript
function Greeting(props) {
  const isLoggedIn = props.isLoggedIn;
  if (isLoggedIn) {
    return <UserGreeting />;
  }
  return <GuestGreeting />;
}

ReactDOM.render(
  // Try changing to isLoggedIn={true}:
  <Greeting isLoggedIn={false} />,
  document.getElementById('root')
);
```

Cet exemple rend un message d'accueil différent en fonction de la valeur du prop `isLoggedIn`.

## Element Variables

Vous pouvez utiliser des variables pour stocker des éléments. Cela peut vous aider à rendre conditionnellement une partie du composant alors que le reste de la sortie ne change pas.

Considérez ces deux nouveaux composants représentant les boutons Déconnexion et Connexion:

```js
function LoginButton(props) {
  return (
    <button onClick={props.onClick}>
      Login
    </button>
  );
}

function LogoutButton(props) {
  return (
    <button onClick={props.onClick}>
      Logout
    </button>
  );
}
```

Dans l'exemple ci-dessous, nous allons créer un [stateful component](/docs/state-and-lifecycle.html#adding-local-state-to-a-class) appelé `LoginControl`.

Il affichera `<LoginButton />` ou `<LogoutButton />` en fonction de son état actuel. Il va aussi afficher un `<Greeting />` à partir de l'exemple précédent:

```javascript
class LoginControl extends React.Component {
  constructor(props) {
    super(props);
    this.handleLoginClick = this.handleLoginClick.bind(this);
    this.handleLogoutClick = this.handleLogoutClick.bind(this);
    this.state = {isLoggedIn: false};
  }

  handleLoginClick() {
    this.setState({isLoggedIn: true});
  }

  handleLogoutClick() {
    this.setState({isLoggedIn: false});
  }

  render() {
    const isLoggedIn = this.state.isLoggedIn;
    let button;

    if (isLoggedIn) {
      button = <LogoutButton onClick={this.handleLogoutClick} />;
    } else {
      button = <LoginButton onClick={this.handleLoginClick} />
    }

    return (
      <div>
        <Greeting isLoggedIn={isLoggedIn} />
        {button}
      </div>
    );
  }
}

ReactDOM.render(
  <LoginControl />,
  document.getElementById('root')
);
```

Alors que déclarer une variable et utiliser une instruction `if` est un bon moyen de restituer de manière conditionnelle un composant, vous pouvez parfois vouloir utiliser une syntaxe plus courte. Il existe plusieurs façons d'intégrer les conditions dans JSX, expliquées ci-dessous.

## Inline If with Logical && Operator

Vous pouvez [embed any expressions in JSX](/docs/introducing-jsx.html#embedding-expressions-in-jsx) en les enveloppant dans des accolades. Cela inclut l'opérateur logique JavaScript `&&`. Il peut être pratique pour conditionnellement y compris un élément:

```js
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 &&
        <h2>
          You have {unreadMessages.length} unread messages.
        </h2>
      }
    </div>
  );
}

const messages = ['React', 'Re: React', 'Re:Re: React'];
ReactDOM.render(
  <Mailbox unreadMessages={messages} />,
  document.getElementById('root')
);
```

Cela fonctionne car en JavaScript, `true && expression` est toujours évalué à `expression`, et `false && expression` est toujours évalué à `false`. Par conséquent, si la condition est `true`, l'élément juste après `&&` apparaîtra dans la sortie. Si c'est faux, React l'ignorera.

## Inline If-Else with Conditional Operator

Une autre méthode de rendu conditionnel des éléments en ligne consiste à utiliser l'opérateur conditionnel JavaScript [`condition ? true : false`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Conditional_Operator).

Dans l'exemple ci-dessous, nous l'utilisons pour rendre conditionnellement un petit bloc de texte.

```javascript
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      The user is <b>{isLoggedIn ? 'currently' : 'not'}</b> logged in.
    </div>
  );
}
```

Il peut également être utilisé pour des expressions plus larges bien que ce qui se passe soit moins évident:

```js
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      {isLoggedIn ? (
        <LogoutButton onClick={this.handleLogoutClick} />
      ) : (
        <LoginButton onClick={this.handleLoginClick} />
      )}
    </div>
  );
}
```

Tout comme en JavaScript, il vous appartient de choisir un style approprié en fonction de ce que votre équipe et vous considérez comme plus lisible. Rappelez-vous également que chaque fois que les conditions deviennent trop complexes, il pourrait être un bon moment pour [extract a component](/docs/components-and-props.html#extracting-components).

## Preventing Component from Rendering

Dans de rares cas, vous pourriez vouloir qu'un composant se cache même s'il a été rendu par un autre composant. Pour ce faire, renvoyez null à la place de sa sortie de rendu.

```javascript
function WarningBanner(props) {
  if (!props.warn) {
    return null;
  }

  return (
    <div className="warning">
      Warning!
    </div>
  );
}

class Page extends React.Component {
  constructor(props) {
    super(props);
    this.state = {showWarning: true};
    this.handleToggleClick = this.handleToggleClick.bind(this);
  }

  handleToggleClick() {
    this.setState(prevState => ({
      showWarning: !prevState.showWarning
    }));
  }

  render() {
    return (
      <div>
        <WarningBanner warn={this.state.showWarning} />
        <button onClick={this.handleToggleClick}>
          {this.state.showWarning ? 'Hide' : 'Show'}
        </button>
      </div>
    );
  }
}

ReactDOM.render(
  <Page />,
  document.getElementById('root')
);
```

Renvoyer `null` à partir de la méthode `render` d'un composant n'affecte pas le déclenchement des méthodes lifecycle du composant. Par exemple `componentDidUpdate` sera toujours appelé.
