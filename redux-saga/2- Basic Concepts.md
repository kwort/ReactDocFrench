# Basic Concepts

## Using Saga Helpers

redux-saga fournit des effets d'aide enveloppant les fonctions internes pour générer des tâches lorsque certaines actions spécifiques sont envoyées au store.

Les fonctions d'aide sont construites au-dessus de l'API de niveau inférieur. Dans la section avancée, nous verrons comment ces fonctions peuvent être implémentées.

La première fonction, takeEvery est la plus familière et fournit un comportement similaire à redux-thunk.

Illustrons avec l'exemple commun AJAX. À chaque clic sur un bouton Fetch, nous envoyons une action FETCH_REQUESTED. Nous voulons gérer cette action en lançant une tâche qui va chercher des données sur le serveur.

Nous créons d'abord la tâche qui va effectuer l'action asynchrone:

```javascript
import { call, put } from 'redux-saga/effects'

export function* fetchData(action) {
   try {
      const data = yield call(Api.fetchUser, action.payload.url)
      yield put({type: "FETCH_SUCCEEDED", data})
   } catch (error) {
      yield put({type: "FETCH_FAILED", error})
   }
}
```

Pour lancer la tâche ci-dessus sur chaque action FETCH_REQUESTED:

```javascript
import { takeEvery } from 'redux-saga/effects'

function* watchFetchData() {
  yield takeEvery('FETCH_REQUESTED', fetchData)
}
```

Dans l'exemple ci-dessus, takeEvery permet le démarrage simultané de plusieurs instances de fetchData. À un moment donné, nous pouvons lancer une nouvelle tâche fetchData alors qu'il existe encore une ou plusieurs tâches fetchData récédentes qui n'ont pas encore été terminées.

Si nous voulons seulement obtenir la réponse de la dernière requête lancée (par exemple pour toujours afficher la dernière version des données), nous pouvons utiliser l'assistant takeLatest:

```javascript
import { takeLatest } from 'redux-saga/effects'

function* watchFetchData() {
  yield takeLatest('FETCH_REQUESTED', fetchData)
}
```

Contrairement à takeEvery, takeLatest n'autorise qu'une seule tâche fetchData à s'exécuter à tout moment. Et ce sera la dernière tâche commencée. Si une tâche précédente est toujours en cours lorsqu'une autre tâche fetchData est démarrée, la tâche précédente sera automatiquement annulée.

Si vous avez plusieurs sagas qui surveillent différentes actions, vous pouvez créer plusieurs observateurs avec ces aides intégrées, qui se comporteront comme s'il y avait une fourchette utilisée pour les engendrer (nous parlerons plus tard de la fourchette. Effet qui nous permet de lancer plusieurs sagas en arrière-plan).

Par exemple:

```javascript
import { takeEvery } from 'redux-saga/effects'

// FETCH_USERS
function* fetchUsers(action) { ... }

// CREATE_USER
function* createUser(action) { ... }

// use them in parallel
export default function* rootSaga() {
  yield takeEvery('FETCH_USERS', fetchUsers)
  yield takeEvery('CREATE_USER', createUser)
}
```

## Declarative Effects

Dans redux-saga, les Sagas sont implémentés en utilisant les fonctions Generator. Pour exprimer la logique Saga, nous générons des objets JavaScript simples à partir du générateur. Nous appelons ces effets d'objets. Un effet est simplement un objet qui contient des informations à interpréter par le middleware. Vous pouvez afficher des effets comme des instructions sur le middleware pour effectuer certaines opérations (par exemple, invoquer une fonction asynchrone, envoyer une action au store, etc.).

Pour créer des effets, vous utilisez les fonctions fournies par la bibliothèque dans le paquet redux-saga / effets.

Dans cette section et les suivantes, nous allons présenter quelques effets de base. Et voyez comment le concept permet aux Sagas d'être facilement testés.

Les sagas peuvent produire des effets sous plusieurs formes. Le moyen le plus simple est de donner une promise.

Par exemple, supposons que nous ayons une Saga qui surveille une action PRODUCTS_REQUESTED. À chaque action correspondante, il lance une tâche pour extraire une liste de produits d'un serveur.

```javascript
import { takeEvery } from 'redux-saga/effects'
import Api from './path/to/api'

function* watchFetchProducts() {
  yield takeEvery('PRODUCTS_REQUESTED', fetchProducts)
}

function* fetchProducts() {
  const products = yield Api.fetch('/products')
  console.log(products)
}
```

Dans l'exemple ci-dessus, nous appelons Api.fetch directement depuis l'intérieur du générateur (dans les fonctions Générateur, toute expression à droite du rendement est évaluée puis le résultat est cédé à l'appelant).

Api.fetch ('/ products') déclenche une requête AJAX et renvoie une Promesse qui sera résolue avec la réponse résolue, la requête AJAX sera exécutée immédiatement. Simple et idiomatique, mais ...

Supposons que nous voulons tester le générateur ci-dessus:

```javascript
const iterator = fetchProducts()
assert.deepEqual(iterator.next().value, ??) // what do we expect ?
```


Nous voulons vérifier le résultat de la première valeur produite par le générateur. Dans notre cas, c'est le résultat de l'exécution d'Api.fetch ('/ products') qui est une promesse. L'exécution du service réel pendant les tests n'est ni une approche viable ni pratique, donc nous devons nous moquer de la fonction Api.fetch, c'est-à-dire que nous allons devoir remplacer la fonction réelle par une fausse qui ne lance pas la requête AJAX. vérifie que nous avons appelé Api.fetch avec les bons arguments ('/ products' dans notre cas).

Les mocks rendent les tests plus difficiles et moins fiables. D'un autre côté, les fonctions qui retournent simplement des valeurs sont plus faciles à tester, puisque nous pouvons utiliser un simple equal () pour vérifier le résultat. C'est la façon d'écrire les tests les plus fiables.

Quelle est la sortie réelle?
Quelle est l'attente de production?
Si vous terminez un test sans répondre à ces deux questions, vous n'avez pas de véritable test unitaire. Vous avez un test bâclé et semi-cuit.

Ce dont nous avons réellement besoin est juste de nous assurer que la tâche fetchProducts génère un appel avec la bonne fonction et les bons arguments.

Au lieu d'invoquer la fonction asynchrone directement depuis l'intérieur du générateur, nous ne pouvons donner qu'une description de l'invocation de la fonction. C'est-à-dire que nous allons simplement donner un objet qui ressemble à

```javascript
// Effect -> call the function Api.fetch with `./products` as argument
{
  CALL: {
    fn: Api.fetch,
    args: ['./products']
  }
}
```

En d'autres termes, le générateur générera des objets simples contenant des instructions, et le middleware redux-saga se chargera d'exécuter ces instructions et de restituer le résultat de leur exécution au générateur. De cette façon, lors du test du Générateur, tout ce que nous devons faire est de vérifier qu'il donne l'instruction attendue en faisant un simple deepEqual sur l'Object produit.

Pour cette raison, la bibliothèque fournit une manière différente d'effectuer des appels asynchrones.

```javascript
import { call } from 'redux-saga/effects'

function* fetchProducts() {
  const products = yield call(Api.fetch, '/products')
  // ...
}
```

Nous utilisons maintenant la fonction call (fn, ...args). La différence par rapport à l'exemple précédent est que maintenant, nous n'exécutons pas l'appel de récupération immédiatement, call crée une description de l'effet. Tout comme dans Redux, vous utilisez des créateurs d'actions pour créer un objet simple décrivant l'action qui sera exécutée par le magasin, l'appel crée un objet simple décrivant l'appel de la fonction. Le middleware redux-saga prend en charge l'exécution de l'appel de fonction et la reprise du générateur avec la réponse résolue.

Cela nous permet de tester facilement le générateur en dehors de l'environnement Redux. Parce que call est juste une fonction qui renvoie un objet simple.

```javascript
import { call } from 'redux-saga/effects'
import Api from '...'

const iterator = fetchProducts()

// expects a call instruction
assert.deepEqual(
  iterator.next().value,
  call(Api.fetch, '/products'),
  "fetchProducts should yield an Effect call(Api.fetch, './products')"
)
```

Maintenant, nous n'avons plus besoin de nous moquer de rien, et un simple test d'égalité suffira.

L'avantage de ces appels déclaratifs est que nous pouvons tester toute la logique à l'intérieur d'une saga en passant simplement par itération sur le générateur et en effectuant un test deepEqual sur les valeurs successives. C'est un réel avantage, car vos opérations asynchrones complexes ne sont plus des boîtes noires, et vous pouvez tester en détail leur logique opérationnelle, aussi complexe soit-elle.

call prend également en charge l'appel de méthodes d'objet, vous pouvez fournir un contexte aux fonctions invoquées en utilisant le formulaire suivant:

```javascript
yield call([obj, obj.method], arg1, arg2, ...) // as if we did obj.method(arg1, arg2 ...)
```

apply est un alias pour le formulaire d'invocation de méthode

```javascript
yield apply(obj, obj.method, [arg1, arg2, ...])
```

call et apply conviennent bien aux fonctions qui retournent les résultats de Promise. Une autre fonction cps peut être utilisée pour gérer les fonctions de style Node (e.g. fn(...args, callback) où callback est de la forme (error, result) => ()). cps signifie Continuation Passing Style.

For example:

```javascript
import { cps } from 'redux-saga/effects'

const content = yield cps(readFile, '/path/to/file')
```

Et bien sûr, vous pouvez le tester comme vous l'appelez:

```javascript
import { cps } from 'redux-saga/effects'

const iterator = fetchSaga()
assert.deepEqual(iterator.next().value, cps(readFile, '/path/to/file') )
```

cps prend également en charge le même formulaire d'appel de méthode que call.

Une liste complète des effets déclaratifs peut être trouvée dans la référence de l'API.

## Dispatching actions to the store

En prenant l'exemple précédent plus loin, disons qu'après chaque sauvegarde, nous voulons envoyer une action pour avertir le Store que la récupération a réussi (nous omettrons le cas d'échec pour le moment).

Nous pourrions passer la fonction d'expédition du magasin au store. Ensuite, le générateur pourrait l'appeler après avoir reçu la réponse de récupération:

```javascript
// ...

function* fetchProducts(dispatch) {
  const products = yield call(Api.fetch, '/products')
  dispatch({ type: 'PRODUCTS_RECEIVED', products })
}
```

Cependant, cette solution présente les mêmes inconvénients que l'invocation de fonctions directement depuis l'intérieur du générateur (comme indiqué dans la section précédente). Si nous voulons tester que fetchProducts effectue la distribution après avoir reçu la réponse AJAX, nous aurons encore besoin de simuler la fonction de distribution.

Au lieu de cela, nous avons besoin de la même solution déclarative. Créez simplement un objet pour indiquer au middleware dont nous avons besoin pour envoyer une action, et laissez le middleware effectuer la répartition réelle. De cette façon, nous pouvons tester la répartition du générateur de la même manière: en inspectant simplement l'effet produit et en vérifiant qu'il contient les instructions correctes.

La bibliothèque fournit, à cet effet, une autre fonction put qui crée l'effet de répartition.

```javascript
import { call, put } from 'redux-saga/effects'
// ...

function* fetchProducts() {
  const products = yield call(Api.fetch, '/products')
  // create and yield a dispatch Effect
  yield put({ type: 'PRODUCTS_RECEIVED', products })
}
```

Now, we can test the Generator easily as in the previous section

```javascript
import { call, put } from 'redux-saga/effects'
import Api from '...'

const iterator = fetchProducts()

// expects a call instruction
assert.deepEqual(
  iterator.next().value,
  call(Api.fetch, '/products'),
  "fetchProducts should yield an Effect call(Api.fetch, './products')"
)

// create a fake response
const products = {}

// expects a dispatch instruction
assert.deepEqual(
  iterator.next(products).value,
  put({ type: 'PRODUCTS_RECEIVED', products }),
  "fetchProducts should yield an Effect put({ type: 'PRODUCTS_RECEIVED', products })"
)
```

Notez comment nous passons la fausse réponse au générateur via sa prochaine méthode. En dehors de l'environnement middleware, nous avons un contrôle total sur le générateur, nous pouvons simuler un environnement réel en simulant simplement les résultats et en reprenant le générateur avec eux. Les données moqueuses sont beaucoup plus simples que les fonctions moqueuses et les appels d'espionnage.

## Error handling

Dans cette section, nous verrons comment gérer le cas d'échec de l'exemple précédent. Supposons que notre fonction API Api.fetch renvoie une promesse qui est rejetée lorsque la récupération à distance échoue pour une raison quelconque.

Nous voulons gérer ces erreurs dans notre Saga en envoyant une action PRODUCTS_REQUEST_FAILED au Store.

Nous pouvons détecter les erreurs dans la Saga en utilisant la syntaxe try / catch familière.

```javascript
import Api from './path/to/api'
import { call, put } from 'redux-saga/effects'

// ...

function* fetchProducts() {
  try {
    const products = yield call(Api.fetch, '/products')
    yield put({ type: 'PRODUCTS_RECEIVED', products })
  } catch(error) {
    yield put({ type: 'PRODUCTS_REQUEST_FAILED', error })
  }
}
```

Afin de tester le cas d'échec, nous allons utiliser la méthode de lancement du générateur

```javascript
import { call, put } from 'redux-saga/effects'
import Api from '...'

const iterator = fetchProducts()

// expects a call instruction
assert.deepEqual(
  iterator.next().value,
  call(Api.fetch, '/products'),
  "fetchProducts should yield an Effect call(Api.fetch, './products')"
)

// create a fake error
const error = {}

// expects a dispatch instruction
assert.deepEqual(
  iterator.throw(error).value,
  put({ type: 'PRODUCTS_REQUEST_FAILED', error }),
  "fetchProducts should yield an Effect put({ type: 'PRODUCTS_REQUEST_FAILED', error })"
)
```

Dans ce cas, nous passons la méthode throw une fausse erreur. Cela entraînera le générateur à interrompre le flux actuel et à exécuter le bloc catch.

Bien sûr, vous n'êtes pas obligé de gérer vos erreurs d'API à l'intérieur des blocs try / catch. Vous pouvez également faire en sorte que votre service API renvoie une valeur normale avec un indicateur d'erreur. Par exemple, vous pouvez capturer les rejets de Promise et les mapper à un objet avec un champ d'erreur.

```javascript
import Api from './path/to/api'
import { call, put } from 'redux-saga/effects'

function fetchProductsApi() {
  return Api.fetch('/products')
    .then(response => ({ response }))
    .catch(error => ({ error }))
}

function* fetchProducts() {
  const { response, error } = yield call(fetchProductsApi)
  if (response)
    yield put({ type: 'PRODUCTS_RECEIVED', products: response })
  else
    yield put({ type: 'PRODUCTS_REQUEST_FAILED', error })
}
```

## A common abstraction: Effect

Pour généraliser, le déclenchement d'effets secondaires depuis l'intérieur d'une saga est toujours effectué en produisant un effet déclaratif. (Vous pouvez également céder la Promise directement, mais cela rendra le test difficile comme nous l'avons vu dans la première section.)

Qu'est-ce qu'une Saga fait est de composer tous ces effets ensemble pour mettre en œuvre le flux de contrôle souhaité. L'exemple le plus simple consiste à séquencer les effets obtenus en mettant simplement les rendements l'un après l'autre. Vous pouvez également utiliser les opérateurs de flux de contrôle familiers (if, while, for) pour implémenter des flux de contrôle plus sophistiqués.

Nous avons vu qu'utiliser des effets comme call et put, combinés avec des API de haut niveau comme takeEvery, nous permet de réaliser les mêmes choses que redux-thunk, mais avec l'avantage supplémentaire de tester facilement.

Mais redux-saga offre un autre avantage sur redux-thunk. Dans la section Advanced, vous découvrirez des effets plus puissants qui vous permettront d'exprimer des flux de contrôle complexes tout en conservant le même bénéfice de testabilité.