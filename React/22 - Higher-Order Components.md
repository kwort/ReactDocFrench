# Higher-order components

## Introduction

Un **higher-order component (HOC)** est une technique avancée dans React pour réutiliser la logique de composant. Les HOC ne font pas partie de l'API React, ils sont un modèle qui émerge de la nature compositionnelle de React.

Concrètement, **un higher-order component (HOC) est une fonction qui prend un composant et renvoie un nouveau composant.**

```js
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

Alors qu'un composant transforme les props en UI, un HOC transforme un composant en un autre composant.

Les HOC sont communs dans les bibliothèques React tierces, telles que Redux [`connect`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) et Relay [`createFragmentContainer`](http://facebook.github.io/relay/docs/en/fragment-container.html).

## Use HOCs For Cross-Cutting Concerns

Les composants sont l'unité primaire de réutilisation du code dans React. Cependant, vous trouverez que certains modèles ne sont pas adaptés aux composants traditionnels.

Par exemple, supposons que vous ayez un composant `CommentList` qui subscribe à une source de données externe pour afficher une liste de commentaires:

```js
class CommentList extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      // "DataSource" is some global data source
      comments: DataSource.getComments()
    };
  }

  componentDidMount() {
    // Subscribe to changes
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    // Clean up listener
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    // Update component state whenever the data source changes
    this.setState({
      comments: DataSource.getComments()
    });
  }

  render() {
    return (
      <div>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id} />
        ))}
      </div>
    );
  }
}
```

Plus tard, vous écrivez un composant pour subscribe à un seul article de blog, qui suit un modèle similaire:

```js
class BlogPost extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      blogPost: DataSource.getBlogPost(props.id)
    };
  }

  componentDidMount() {
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    this.setState({
      blogPost: DataSource.getBlogPost(this.props.id)
    });
  }

  render() {
    return <TextBlock text={this.state.blogPost} />;
  }
}
```

`CommentList` et `BlogPost` ne sont pas identiques - ils appellent des méthodes différentes sur `DataSource`, et ils rendent différentes sorties. Mais une grande partie de leur mise en œuvre est la même:

- sur componentDidMount, ajoutez un listener de modification à `DataSource`.
- Dans le listener, appelez `setState` quand la source de données change.
- Une fois démonté, supprimez le listener de modification.

Vous pouvez imaginer que dans une grande app, ce même schéma de subscribing à `DataSource` et l'appel à `setState` se répète encore et encore. Nous voulons une abstraction qui nous permette de définir cette logique en un seul endroit et de la partager entre de nombreux composants. C'est là que les HOC excellent.

Nous pouvons écrire une fonction qui crée des composants, comme `CommentList` et `BlogPost`, qui subscribe à `DataSource`. La fonction accepte comme l'un de ses arguments un composant enfant qui reçoit les données souscrites en tant que prop. Appelons la fonction `withSubscription`:

```js
const CommentListWithSubscription = withSubscription(
  CommentList,
  (DataSource) => DataSource.getComments()
);

const BlogPostWithSubscription = withSubscription(
  BlogPost,
  (DataSource, props) => DataSource.getBlogPost(props.id)
);
```

Le premier paramètre est le composant enveloppé. Le deuxième paramètre récupère les données qui nous intéressent, étant donné une `DataSource` et les props actuels.

Quand `CommentListWithSubscription` et `BlogPostWithSubscription` sont rendu, `CommentList` et `Blog Post` recevront un argument `data` avec les données les plus récentes récupérées dans `DataSource`:

```js
// This function takes a component...
function withSubscription(WrappedComponent, selectData) {
  // ...and returns another component...
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      // ... that takes care of the subscription...
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // ... and renders the wrapped component with the fresh data!
      // Notice that we pass through any additional props
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

Notez qu'un HOC ne modifie pas le composant d'entrée et n'utilise pas l'héritage pour copier son comportement. Plutôt, un HOC *compose* le composant d'origine en *wrappant* dans un composant conteneur. Un HOC est une fonction pure sans effets secondaires.

Et c'est tout! Le composant enveloppé reçoit tous les props du conteneur, ainsi qu'une nouvelle prop, `data`, qu'il utilise pour afficher sa sortie. Le HOC n'est pas concerné par le comment ou le pourquoi les données sont utilisées, et le composant enveloppé ne concerne pas l'origine des données.

Parce que `withSubscription` est une fonction normale, vous pouvez ajouter autant ou aussi peu d'arguments que vous le souhaitez. Par exemple, vous voudrez peut-être configurer le nom du prop `data` pour pouvoir isoler davantage le HOC du composant enveloppé. Ou vous pouvez accepter un argument qui configure `shouldComponentUpdate`, ou un qui configure la source de données. Tout cela est possible parce que le HOC contrôle totalement la définition du composant.

Comme les composants, le contrat entre `withSubscription` et le composant wrapped est entièrement basé sur les props. Cela facilite l'échange d'un HOC pour un HOC différent, à condition qu'il fournisse les mêmes props au composant wrapped. Cela peut être utile si vous modifiez des bibliothèques de récupération de données, par exemple.

## Don't Mutate the Original Component. Use Composition

Résistez à la tentation de modifier le prototype d'un composant (ou de le faire muter autrement) à l'intérieur d'un HOC.

```js
function logProps(InputComponent) {
  InputComponent.prototype.componentWillReceiveProps = function(nextProps) {
    console.log('Current props: ', this.props);
    console.log('Next props: ', nextProps);
  };
  // The fact that we're returning the original input is a hint that it has
  // been mutated.
  return InputComponent;
}

// EnhancedComponent will log whenever props are received
const EnhancedComponent = logProps(InputComponent);
```

Il y a quelques problèmes avec ça. La première est que le composant d'entrée ne peut pas être réutilisé séparément du composant amélioré. Plus important encore, si vous appliquez un autre HOC à `EnhancedComponent` qui * mutera aussi `componentWillReceiveProps`, la fonctionnalité du premier HOC sera surchargée! Ce HOC ne fonctionnera pas non plus avec les composants fonctionnels, qui n'ont pas de méthodes lifecycle.

Les HOC en mutation sont une abstraction qui fuit - le consommateur doit savoir comment ils sont mis en œuvre afin d'éviter les conflits avec d'autres HOC.

Au lieu de la mutation, les HOC devraient utiliser la composition, en wrappant le composant d'entrée dans un composant de conteneur:

```js
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log('Current props: ', this.props);
      console.log('Next props: ', nextProps);
    }
    render() {
      // Wraps the input component in a container, without mutating it. Good!
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

Ce HOC a la même fonctionnalité que la version en mutation tout en évitant les risques de conflits. Cela fonctionne aussi bien avec les composants de classe et fonctionnels. Et parce que c'est une fonction pure, elle est composable avec d'autres HOC, ou même avec elle-même.

Vous avez peut-être remarqué des similitudes entre les HOC et un modèle appelé **container components**. Les composants de conteneurs font partie d'une stratégie de séparation des responsabilités entre les préoccupations de haut niveau et de bas niveau. Les conteneurs gèrent des choses comme les abonnements et l'état, et transmettent des props aux composants qui gèrent des choses comme le rendu de l'interface utilisateur. Les HOC utilisent des conteneurs dans le cadre de leur mise en œuvre. Vous pouvez considérer les HOC comme des définitions de composants de conteneur paramétrées.

## Convention: Pass Unrelated Props Through to the Wrapped Component

Les HOC ajoutent des fonctionnalités à un composant. Ils ne devraient pas modifier radicalement son contrat. On s'attend à ce que le composant renvoyé par un HOC ait une interface similaire au composant enveloppé.

Les HOCs devraient passer par des props qui ne sont pas liés à leurs préoccupations spécifiques. La plupart des HOC contiennent une méthode de rendu qui ressemble à ceci:

```js
render() {
  // Filter out extra props that are specific to this HOC and shouldn't be
  // passed through
  const { extraProp, ...passThroughProps } = this.props;

  // Inject props into the wrapped component. These are usually state values or
  // instance methods.
  const injectedProp = someStateOrInstanceMethod;

  // Pass props to wrapped component
  return (
    <WrappedComponent
      injectedProp={injectedProp}
      {...passThroughProps}
    />
  );
}
```

Cette convention permet de s'assurer que les COH sont aussi flexibles et réutilisables que possible.

## Convention: Maximizing Composability

Tous les HOCs ne se ressemblent pas. Parfois, ils n'acceptent qu'un seul argument, le composant enveloppé:

```js
const NavbarWithRouter = withRouter(Navbar);
```

Habituellement, les HOCs acceptent des arguments supplémentaires. Dans cet exemple de Relay, un objet de configuration est utilisé pour spécifier les dépendances de données d'un composant:

```js
const CommentWithRelay = Relay.createContainer(Comment, config);
```

La signature la plus courante pour les HOC est la suivante:

```js
// React Redux's `connect`
const ConnectedComment = connect(commentSelector, commentActions)(CommentList);
```

Si vous le divisez, il est plus facile de voir ce qui se passe.

```js
// connect is a function that returns another function
const enhance = connect(commentListSelector, commentListActions);
// The returned function is a HOC, which returns a component that is connected
// to the Redux store
const ConnectedComment = enhance(CommentList);
```

En d'autres termes, `connect` est une fonction d'ordre supérieur qui renvoie un composant d'ordre supérieur!

Cette forme peut sembler confuse ou inutile, mais elle a une propriété utile. Les HOC à un seul argument comme celui renvoyé par la fonction `connect` ont la signature `Component => Component`. Les fonctions dont le type de sortie est le même que son type d'entrée sont vraiment faciles à composer ensemble.

```js
// Instead of doing this...
const EnhancedComponent = withRouter(connect(commentSelector)(WrappedComponent))

// ... you can use a function composition utility
// compose(f, g, h) is the same as (...args) => f(g(h(...args)))
const enhance = compose(
  // These are both single-argument HOCs
  withRouter,
  connect(commentSelector)
)
const EnhancedComponent = enhance(WrappedComponent)
```

(Cette même propriété permet également d'utiliser `connect` et d'autres HOC de style enhancer comme décorateurs, une proposition JavaScript expérimentale.)

La fonction utilitaire `compose` est fournie par de nombreuses bibliothèques tierces, y compris lodash (as [`lodash.flowRight`](https://lodash.com/docs/#flowRight)), [Redux](http://redux.js.org/docs/api/compose.html), et [Ramda](http://ramdajs.com/docs/#compose).

## Convention: Wrap the Display Name for Easy Debugging

Les composants du conteneur créés par HOC apparaissent dans les [React Developer Tools](https://github.com/facebook/react-devtools) comme tout autre composant. Pour faciliter le débogage, choisissez un nom d'affichage qui indique que c'est le résultat d'un HOC.

La technique la plus courante consiste à envelopper le nom d'affichage du composant enveloppé. Donc, si votre composant d'ordre supérieur est nommé `withSubscription`, et que le nom d'affichage du composant encapsulé est `CommentList`, utilisez le nom d'affichage `WithSubscription(CommentList)`:

```js
function withSubscription(WrappedComponent) {
  class WithSubscription extends React.Component {/* ... */}
  WithSubscription.displayName = `WithSubscription(${getDisplayName(WrappedComponent)})`;
  return WithSubscription;
}

function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName || WrappedComponent.name || 'Component';
}
```

## Caveats

Les composants d'ordre supérieur sont accompagnés de quelques mises en garde qui ne sont pas immédiatement évidentes si vous êtes nouveau dans React.

### Don't Use HOCs Inside the render Method

L'algorithme de comparaison de React (appelé reconciliation) utilise l'identité du composant pour déterminer s'il doit mettre à jour le sous-arbre existant ou le jeter et en monter un nouveau. Si le composant renvoyé par `render` est identique (`===`) au composant du rendu précédent, React met à jour de manière récursive le sous-arbre en le différant avec le nouveau. Si elles ne sont pas égales, le sous-arbre précédent est complètement démonté.

Normalement, vous ne devriez pas avoir à y penser. Mais c'est important pour les HOC car cela signifie que vous ne pouvez pas appliquer un HOC à un composant dans la méthode render d'un composant:

```js
render() {
  // A new version of EnhancedComponent is created on every render
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // That causes the entire subtree to unmount/remount each time!
  return <EnhancedComponent />;
}
```

Le problème ici ne concerne pas uniquement les performances: le remontage d'un composant entraîne la perte de l'état de ce composant et de tous ses enfants

Au lieu de cela, appliquez les HOCs en dehors de la définition du composant afin que le composant résultant ne soit créé qu'une seule fois. Ensuite, son identité sera cohérente entre les rendus. C'est généralement ce que vous voulez, de toute façon

Dans les rares cas où vous devez appliquer dynamiquement un HOC, vous pouvez également le faire à l'intérieur des méthodes de lifecycle d'un composant ou de son constructeur.

### Static Methods Must Be Copied Over

Parfois, il est utile de définir une méthode statique sur un composant React. Par exemple, les conteneurs Relay exposent une méthode statique `getFragment` pour faciliter la composition des fragments GraphQL.

Lorsque vous appliquez un HOC à un composant, le composant d'origine est enveloppé avec un composant de conteneur. Cela signifie que le nouveau composant n'a aucune des méthodes statiques du composant d'origine.

```js
// Define a static method
WrappedComponent.staticMethod = function() {/*...*/}
// Now apply a HOC
const EnhancedComponent = enhance(WrappedComponent);

// The enhanced component has no static method
typeof EnhancedComponent.staticMethod === 'undefined' // true
```

Pour résoudre ce problème, vous pouvez copier les méthodes sur le conteneur avant de le renvoyer:

```js
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  // Must know exactly which method(s) to copy :(
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
}
```

Cependant, cela nécessite que vous sachiez exactement quelles méthodes doivent être copiées. Vous pouvez utiliser [hoist-non-react-statics](https://github.com/mridgway/hoist-non-react-statics) pour copier automatiquement toutes les méthodes statiques non-React:

```js
import hoistNonReactStatic from 'hoist-non-react-statics';
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  hoistNonReactStatic(Enhance, WrappedComponent);
  return Enhance;
}
```

Une autre solution possible consiste à exporter la méthode statique séparément du composant lui-même.

```js
// Instead of...
MyComponent.someFunction = someFunction;
export default MyComponent;

// ...export the method separately...
export { someFunction };

// ...and in the consuming module, import both
import MyComponent, { someFunction } from './MyComponent.js';
```

### Refs Aren't Passed Through

Alors que la convention pour les composants d'ordre supérieur est de passer tous les props au composant wrappé, cela ne fonctionne pas pour les refs. C'est parce que `ref` n'est pas vraiment une sorte de 'clé', elle est gérée spécialement par React. Si vous ajoutez un ref à un élément dont le composant est le résultat d'un HOC, le ref fait référence à une instance du composant de conteneur le plus à l'extérieur, pas au composant enveloppé.

La solution à ce problème consiste à utiliser l'API `React.forwardRef` (introduite avec React 16.3). [Learn more about it in the forwarding refs section](/docs/forwarding-refs.html).
