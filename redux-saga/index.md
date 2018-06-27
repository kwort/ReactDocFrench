<img src='https://redux-saga.js.org/logo/0800/Redux-Saga-Logo-Landscape.png' alt='Redux Logo Landscape' width='800px'>

# redux-saga

`redux-saga` est une bibliothèque qui vise à rendre les effets secondaires des applications plus faciles à gérer (c'est-à-dire des choses asynchrones comme la récupération de données et des choses impures comme accéder au cache du navigateur), plus efficaces à exécuter, plus faciles à tester et mieux gérer les échecs 

Le modèle mental est qu'une saga est comme un fil séparé dans votre application qui est seul responsable des effets secondaires. `redux-saga` est un middleware redux, ce qui signifie que ce thread peut être démarré, suspendu et annulé à partir de l'application principale avec des actions de redux normales, il a accès à l'état redux de l'application et peut également distribuer des actions de redux.

Il utilise une fonction ES6 appelée Generators pour faciliter la lecture, l'écriture et le test de ces flux asynchrones. *(Si vous n'êtes pas familier avec ceci [here are some introductory links](https://redux-saga.js.org/docs/ExternalResources.html))* En faisant cela, ces flux asynchrones ressemblent à votre synchrone standard Code JavaScript. (un peu comme `async` / `await`, mais les générateurs ont quelques fonctionnalités supplémentaires dont nous avons besoin)

Vous avez peut-être utilisé `redux-thunk` avant de gérer votre récupération de données. Contrairement à redux thunk, vous ne finissez pas en callback hell, vous pouvez tester vos flux asynchrones facilement et vos actions restent pures.

# Getting started

## Install

```sh
$ npm install --save redux-saga
```
or

```sh
$ yarn add redux-saga
```

Alternativement, vous pouvez utiliser les builds UMD fournis directement dans la balise `<script>` d'une page HTML. Voir [cette section](# using-umd-build-in-the-browser).

## Usage Example

Supposons que nous ayons une interface utilisateur pour récupérer des données utilisateur à partir d'un serveur distant lorsqu'un bouton est cliqué. (Par souci de concision, nous afficherons simplement le code de déclenchement de l'action.)

```javascript
class UserComponent extends React.Component {
  ...
  onSomeButtonClicked() {
    const { userId, dispatch } = this.props
    dispatch({type: 'USER_FETCH_REQUESTED', payload: {userId}})
  }
  ...
}
```

Le composant distribue une action Object simple au Store. Nous allons créer une Saga qui surveille toutes les actions `USER_FETCH_REQUESTED` et déclenche un appel d'API pour récupérer les données de l'utilisateur.

#### `sagas.js`

```javascript
import { call, put, takeEvery, takeLatest } from 'redux-saga/effects'
import Api from '...'

// worker Saga: will be fired on USER_FETCH_REQUESTED actions
function* fetchUser(action) {
   try {
      const user = yield call(Api.fetchUser, action.payload.userId);
      yield put({type: "USER_FETCH_SUCCEEDED", user: user});
   } catch (e) {
      yield put({type: "USER_FETCH_FAILED", message: e.message});
   }
}

/*
  Starts fetchUser on each dispatched `USER_FETCH_REQUESTED` action.
  Allows concurrent fetches of user.
*/
function* mySaga() {
  yield takeEvery("USER_FETCH_REQUESTED", fetchUser);
}

/*
  Alternatively you may use takeLatest.

  Does not allow concurrent fetches of user. If "USER_FETCH_REQUESTED" gets
  dispatched while a fetch is already pending, that pending fetch is cancelled
  and only the latest one will be run.
*/
function* mySaga() {
  yield takeLatest("USER_FETCH_REQUESTED", fetchUser);
}

export default mySaga;
```

Pour lancer notre Saga, nous devrons la connecter au Redux Store en utilisant le middleware `redux-saga`.

#### `main.js`

```javascript
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

import reducer from './reducers'
import mySaga from './sagas'

// create the saga middleware
const sagaMiddleware = createSagaMiddleware()
// mount it on the Store
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)

// then run the saga
sagaMiddleware.run(mySaga)

// render the application
```

# Documentation

- [Introduction](https://redux-saga.js.org/docs/introduction/BeginnerTutorial.html)
- [Basic Concepts](https://redux-saga.js.org/docs/basics/index.html)
- [Advanced Concepts](https://redux-saga.js.org/docs/advanced/index.html)
- [Recipes](https://redux-saga.js.org/docs/recipes/index.html)
- [External Resources](https://redux-saga.js.org/docs/ExternalResources.html)
- [Troubleshooting](https://redux-saga.js.org/docs/Troubleshooting.html)
- [Glossary](https://redux-saga.js.org/docs/Glossary.html)
- [API Reference](https://redux-saga.js.org/docs/api/index.html)

# Translation

- [Chinese](https://github.com/superRaytin/redux-saga-in-chinese)
- [Traditional Chinese](https://github.com/neighborhood999/redux-saga)
- [Japanese](https://github.com/redux-saga/redux-saga/blob/master/README_ja.md)
- [Korean](https://github.com/mskims/redux-saga-in-korean)
- [Portuguese](https://github.com/joelbarbosa/redux-saga-pt_BR)
- [Russian](https://github.com/redux-saga/redux-saga/blob/master/README_ru.md)

# Using umd build in the browser

Il y a aussi une version **umd** de `redux-saga` disponible dans le dossier` dist / `. Lorsque vous utilisez la construction umd `redux-saga` est disponible en tant que`ReduxSaga` dans l'objet window. Cela vous permet de créer un middleware Saga sans utiliser la syntaxe `import` d'ES6 comme ceci:

```javascript
var sagaMiddleware = ReduxSaga.default()
```

La version umd est utile si vous n'utilisez pas Webpack ou Browserify. Vous pouvez y accéder directement depuis [unpkg](https://unpkg.com/).

Les versions suivantes sont disponibles:

- [https://unpkg.com/redux-saga/dist/redux-saga.js](https://unpkg.com/redux-saga/dist/redux-saga.js)
- [https://unpkg.com/redux-saga/dist/redux-saga.min.js](https://unpkg.com/redux-saga/dist/redux-saga.min.js)

**Important!** Si le navigateur que vous ciblez ne supporte pas les *ES2015 generators*, vous devez les transplanter (c'est-à-dire with [babel plugin](https://github.com/facebook/regenerator/tree/master/packages / regenerator-transform)) et fournit une durée d'exécution valide, telle que [ici](https://unpkg.com/regenerator-runtime/runtime.js). Le runtime doit être importé avant **redux-saga**:

```javascript
import 'regenerator-runtime/runtime'
// then
import sagaMiddleware from 'redux-saga'
```

# Building examples from sources

```sh
$ git clone https://github.com/redux-saga/redux-saga.git
$ cd redux-saga
$ npm install
$ npm test
```

Voici les exemples portés (jusqu'à présent) des repos Redux.

### Counter examples

Il y a trois exemples de counter.

#### counter-vanilla

Démonstration utilisant des versions JavaScript et UMD de vanilla. Toute la source est inline dans `index.html`.

Pour lancer l'exemple, ouvrez `index.html` dans votre navigateur.

> Important: votre navigateur doit supporter les générateurs. Les dernières versions de Chrome / Firefox / Edge sont adaptées.

#### counter

Démo utilisant `webpack` et API de haut niveau` takeEvery`.

```sh
$ npm run counter

# test sample for the generator
$ npm run test-counter
```

#### cancellable-counter

Démonstration utilisant une API de bas niveau pour démontrer l'annulation d'une tâche.

```sh
$ npm run cancellable-counter
```

### Shopping Cart example

```sh
$ npm run shop

# test sample for the generator
$ npm run test-shop
```

### async example

```sh
$ npm run async

# test sample for the generators
$ npm run test-async
```

### real-world example (with webpack hot reloading)

```sh
$ npm run real-world

# sorry, no tests yet
```
