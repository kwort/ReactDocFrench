# Beginner Tutorial

### The initial setup

Avant de commencer, clonez le [référentiel du didacticiel](https://github.com/redux-saga/redux-saga-beginner-tutorial).
Le code final de ce tutoriel se trouve dans la branche des sagas.

Ensuite, dans la ligne de commande, exécutez:

```bash
cd redux-saga-beginner-tutorial
npm install
```

Pour démarrer l'application, exécutez:

```bash
npm start
```

## Hello Sagas!

Nous allons créer notre première Saga. Nous écrirons notre version "Hello, world" pour Sagas.

Créez un fichier sagas.js puis ajoutez l'extrait suivant:

```javascript
export function* helloSaga() {
  console.log('Hello Sagas!')
}
```

Juste une fonction normale (sauf pour le *). Tout ce qu'il fait est afficher un message d'accueil dans la console.

Pour faire tourner notre Saga, nous devons:

Créer un middleware Saga avec une liste de Sagas à exécuter (pour l'instant, nous n'avons qu'un seul HelloSaga) et connecter le middleware Saga au store Redux

Nous allons apporter les modifications à main.js:

```javascript
// ...
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

// ...
import { helloSaga } from './sagas'

const sagaMiddleware = createSagaMiddleware()
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)
sagaMiddleware.run(helloSaga)

const action = type => store.dispatch({type})

// rest unchanged
```

Nous importons d'abord notre Saga à partir du module ./sagas. Ensuite, nous créons un middleware en utilisant la fonction factory createSagaMiddleware exportée par la bibliothèque redux-saga.

Avant de lancer notre HelloSaga, nous devons connecter notre middleware au Store en utilisant applyMiddleware. Ensuite, nous pouvons utiliser le sagaMiddleware.run(helloSaga) pour lancer notre Saga.

Jusqu'à présent, notre Saga n'a rien de spécial. Il enregistre juste un message puis quitte.

## Making Asynchronous calls

Maintenant, ajoutons quelque chose de plus proche de la démo de Counter originale. Pour illustrer les appels asynchrones, nous allons ajouter un autre bouton pour incrémenter le compteur 1 seconde après le clic.

Tout d'abord, nous allons fournir un bouton supplémentaire et un rappel onIncrementAsync au composant de l'interface utilisateur.

```javascript
  const Counter = ({ value, onIncrement, onDecrement, onIncrementAsync }) =>
    <div>
      <button onClick={onIncrementAsync}>
        Increment after 1 second
      </button>
      {' '}
      <button onClick={onIncrement}>
        Increment
      </button>
      {' '}
      <button onClick={onDecrement}>
        Decrement
      </button>
      <hr />
      <div>
        Clicked: {value} times
      </div>
    </div>
```

Ensuite, nous devons connecter l'onIncrementAsync du composant à une action du store.

Nous allons modifier le module main.js comme suit

```javascript
function render() {
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => action('INCREMENT')}
      onDecrement={() => action('DECREMENT')}
      onIncrementAsync={() => action('INCREMENT_ASYNC')} />,
    document.getElementById('root')
  )
}
```

Notez que contrairement à redux-thunk, notre composant distribue une action d'objet simple.

Nous allons maintenant présenter une autre Saga pour effectuer l'appel asynchrone. Notre cas d'utilisation est le suivant:

Sur chaque action INCREMENT_ASYNC, nous voulons démarrer une tâche qui va faire ce qui suit

Attendez 1 seconde puis incrémentez le compteur
Ajoutez le code suivant au module sagas.js:

```javascript
import { delay } from 'redux-saga'
import { put, takeEvery } from 'redux-saga/effects'

// ...

// Our worker Saga: will perform the async increment task
export function* incrementAsync() {
  yield delay(1000)
  yield put({ type: 'INCREMENT' })
}

// Our watcher Saga: spawn a new incrementAsync task on each INCREMENT_ASYNC
export function* watchIncrementAsync() {
  yield takeEvery('INCREMENT_ASYNC', incrementAsync)
}
```

Temps pour quelques explications.

Nous importons delay, une fonction d'utilité qui renvoie une promise qui sera résolue après un nombre spécifié de millisecondes. Nous allons utiliser cette fonction pour bloquer le générateur.

Les sagas sont implémentées comme des fonctions de générateur qui fournissent des objets au middleware redux-saga. Les objets cédés sont une sorte d'instruction à interpréter par le middleware. Lorsqu'une Promesse est cédée au middleware, le middleware suspendra la Saga jusqu'à la fin de la Promesse. Dans l'exemple ci-dessus, la Saga incrementAsync est suspendue jusqu'à ce que la Promesse retournée par le délai se résolve, ce qui arrivera après 1 seconde.

Une fois la Promesse résolue, le middleware reprendra la Saga en exécutant le code jusqu'au prochain rendement. Dans cet exemple, l'instruction suivante est un autre objet généré: le résultat de l'appel de put ({type: 'INCREMENT'}), qui demande au middleware d'envoyer une action INCREMENT.

put est un exemple de ce que nous appelons un effet. Les effets sont de simples objets JavaScript contenant des instructions à remplir par le middleware. Lorsqu'un middleware récupère un effet généré par une saga, la saga est mise en pause jusqu'à ce que l'effet soit satisfait.

Donc pour résumer, la Saga incrementAsync dort pendant 1 seconde via l'appel à delay (1000), puis distribue une action INCREMENT.

Ensuite, nous avons créé une autre Saga watchIncrementAsync. Nous utilisons takeEvery, une fonction d'aide fournie par redux-saga, pour écouter les actions INCREMENT_ASYNC envoyées et exécuter incrementAsync à chaque fois.

Maintenant, nous avons 2 Sagas, et nous devons les démarrer tous les deux à la fois. Pour ce faire, nous allons ajouter un rootSaga responsable du démarrage de nos autres Sagas. Dans le même fichier sagas.js, refactorisez le fichier comme suit:

```javascript
import { delay } from 'redux-saga'
import { put, takeEvery, all } from 'redux-saga/effects'


function* incrementAsync() {
  yield delay(1000)
  yield put({ type: 'INCREMENT' })
}


function* watchIncrementAsync() {
  yield takeEvery('INCREMENT_ASYNC', incrementAsync)
}


// notice how we now only export the rootSaga
// single entry point to start all Sagas at once
export default function* rootSaga() {
  yield all([
    helloSaga(),
    watchIncrementAsync()
  ])
}
```

Cette Saga donne un tableau avec les résultats de l'appel de nos deux sagas, helloSaga et watchIncrementAsync. Cela signifie que les deux générateurs résultants seront démarrés en parallèle. Maintenant, il suffit d'invoquer sagaMiddleware.run sur la Saga root dans main.js.

```javascript
// ...
import rootSaga from './sagas'

const sagaMiddleware = createSagaMiddleware()
const store = ...
sagaMiddleware.run(rootSaga)

// ...
```

## Making our code testable

Nous voulons tester notre incrementAsync Saga pour s'assurer qu'il effectue la tâche désirée.

Créer un autre fichier sagas.spec.js:

```javascript
import test from 'tape';

import { incrementAsync } from './sagas'

test('incrementAsync Saga test', (assert) => {
  const gen = incrementAsync()

  // now what ?
});
```

incrementAsync est une fonction de générateur. Lorsqu'il est exécuté, il renvoie un objet itérateur et la méthode suivante de l'itérateur renvoie un objet de la forme suivante

```javascript
gen.next() // => { done: boolean, value: any }
```

Le champ de valeur contient l'expression produite, c'est-à-dire le résultat de l'expression après le yield. Le champ done indique si le générateur s'est terminé ou s'il y a encore plus d'expressions 'yield'.

Dans le cas d'incrementAsync, le générateur fournit 2 valeurs consécutivement:

yield delay(1000)
yield put({type: 'INCREMENT'})
Donc, si nous invoquons la méthode suivante du générateur 3 fois de suite, nous obtenons les résultats suivants:

```javascript
gen.next() // => { done: false, value: <result of calling delay(1000)> }
gen.next() // => { done: false, value: <result of calling put({type: 'INCREMENT'})> }
gen.next() // => { done: true, value: undefined }
```

Les 2 premières invocations renvoient les résultats des expressions de rendement. Sur la troisième invocation, puisqu'il n'y a plus de rendement, le champ done est mis à true. Et puisque le générateur incrementAsync ne renvoie rien (aucune instruction return), le champ valeur est défini sur undefined.

Donc, maintenant, afin de tester la logique dans incrementAsync, nous devrons simplement parcourir le générateur retourné et vérifier les valeurs générées par le générateur.

```javascript
import test from 'tape';

import { incrementAsync } from './sagas'

test('incrementAsync Saga test', (assert) => {
  const gen = incrementAsync()

  assert.deepEqual(
    gen.next(),
    { done: false, value: ??? },
    'incrementAsync should return a Promise that will resolve after 1 second'
  )
});
```

Le problème est de savoir comment tester la valeur de retour du délai? Nous ne pouvons pas faire un simple test d'égalité sur Promises. Si le délai renvoyait une valeur normale, les choses auraient été plus faciles à tester.

Eh bien, redux-saga fournit un moyen de rendre l'instruction ci-dessus possible. Au lieu d'appeler delay (1000) directement dans incrementAsync, nous l'appellerons indirectement:

```javascript
import { delay } from 'redux-saga'
import { put, takeEvery, all, call } from 'redux-saga/effects'

// ...

export function* incrementAsync() {
  // use the call Effect
  yield call(delay, 1000)
  yield put({ type: 'INCREMENT' })
}
```

Au lieu de faire un retard de rendement (1000), nous faisons maintenant appel à yield (delay, 1000). Quelle est la différence?

Dans le premier cas, le délai d'expression du rendement (1000) est évalué avant d'être transmis à l'appelant suivant (l'appelant peut être le middleware lors de l'exécution de notre code), notre code de test qui exécute la fonction Generator et itère sur le générateur retourné). Donc, ce que l'appelant obtient est une promesse, comme dans le code de test ci-dessus.

Dans le second cas, l'appel yield expression (delay, 1000) est ce qui est passé à l'appelant de next. call, tout comme put, renvoie un effet qui indique au middleware d'appeler une fonction donnée avec les arguments donnés. En fait, ni put, ni call n'effectuent aucun dispatch ou appel asynchrone par eux-mêmes, ils renvoient simplement des objets JavaScript simples.

```javascript
put({type: 'INCREMENT'}) // => { PUT: {type: 'INCREMENT'} }
call(delay, 1000)        // => { CALL: {fn: delay, args: [1000]}}
```

Ce qui se passe est que le middleware examine le type de chaque effet produit puis décide comment accomplir cet effet. Si le type d'effet est un PUT, il enverra une action au store. Si l'effet est un call, il appellera la fonction donnée.

Cette séparation entre la création d'effets et l'exécution d'effets permet de tester notre générateur d'une manière étonnamment simple:

```javascript
import test from 'tape';

import { put, call } from 'redux-saga/effects'
import { delay } from 'redux-saga'
import { incrementAsync } from './sagas'

test('incrementAsync Saga test', (assert) => {
  const gen = incrementAsync()

  assert.deepEqual(
    gen.next().value,
    call(delay, 1000),
    'incrementAsync Saga must call delay(1000)'
  )

  assert.deepEqual(
    gen.next().value,
    put({type: 'INCREMENT'}),
    'incrementAsync Saga must dispatch an INCREMENT action'
  )

  assert.deepEqual(
    gen.next(),
    { done: true, value: undefined },
    'incrementAsync Saga must be done'
  )

  assert.end()
});
```

Depuis que put et call renvoient des objets simples, nous pouvons réutiliser les mêmes fonctions dans notre code de test. Et pour tester la logique d'incrementAsync, on fait simplement une itération sur le générateur et on fait des tests deepEqual sur ses valeurs.

Pour exécuter le test ci-dessus, exécutez:

```bash
npm test
```