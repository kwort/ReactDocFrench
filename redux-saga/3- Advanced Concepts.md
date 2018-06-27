# Advanced Concepts

## Pulling future actions

Jusqu'à présent, nous avons utilisé l'effet d'assistance `takeEvery` pour générer une nouvelle tâche sur chaque action entrante. Cela imite un peu le comportement de `redux-thunk`: chaque fois qu'un Component, par exemple, appelle un créateur d'action `fetchProducts`, le créateur d'action enverra un thunk pour exécuter le flux de contrôle.

En réalité, `takeEvery` est juste un effet d'encapsulation pour la fonction d'aide interne construite au-dessus de l'API de niveau inférieur et plus puissante. Dans cette section, nous verrons un nouvel effet, `take`, qui permet de construire un flux de contrôle complexe en permettant un contrôle total du processus d'observation d'action.

### A basic logger

Prenons un exemple de base d'une Saga qui surveille toutes les actions envoyées au store et les enregistre dans la console.

En utilisant `takeEvery ('*')` (avec le motif générique `*`), nous pouvons capturer toutes les actions distribuées indépendamment de leurs types.

```javascript
import { select, takeEvery } from 'redux-saga/effects'

function* watchAndLog() {
  yield takeEvery('*', function* logger(action) {
    const state = yield select()

    console.log('action', action)
    console.log('state after', state)
  })
}
```

Voyons maintenant comment utiliser l'effet `take` pour implémenter le même flux que ci-dessus:

```javascript
import { select, take } from 'redux-saga/effects'

function* watchAndLog() {
  while (true) {
    const action = yield take('*')
    const state = yield select()

    console.log('action', action)
    console.log('state after', state)
  }
}
```

Le `take` est juste comme `call` et `put` comme nous l'avons vu plus tôt. Il crée un autre objet de commande qui indique au middleware d'attendre une action spécifique. Le comportement résultant de l'effet `call` est le même que lorsque le middleware suspend le générateur jusqu'à la résolution d'une promesse. Dans le cas `take`, le générateur sera suspendu jusqu'à ce qu'une action correspondante soit envoyée. Dans l'exemple ci-dessus, `watchAndLog` est suspendu jusqu'à ce qu'une action soit envoyée.

Notez comment nous exécutons une boucle sans fin `while (true)`. Rappelez-vous, il s'agit d'une fonction Generator, qui n'a pas de comportement d'exécution. Notre Générateur bloquera à chaque itération en attendant qu'une action se produise.

Utiliser `take` a un impact subtil sur la façon dont nous écrivons notre code. Dans le cas de `takeEvery`, les tâches invoquées n'ont aucun contrôle sur le moment où elles seront appelées. Ils seront invoqués encore et encore à chaque action correspondante. Ils n'ont également aucun contrôle sur quand arrêter l'observation.

Dans le cas de `take`, le contrôle est inversé. Au lieu que les actions soient *poussées* vers les tâches du gestionnaire, la Saga *tire* l'action par elle-même. Il semble que la Saga exécute un appel de fonction normal `action = getNextAction()` qui se résoudra quand l'action sera distribuée.

Cette inversion de contrôle nous permet d'implémenter des flux de contrôle non triviaux à faire avec l'approche traditionnelle *push*.

À titre d'exemple de base, supposons que dans notre application Todo, nous souhaitons observer les actions de l'utilisateur et afficher un message de félicitations après que l'utilisateur a créé ses trois premiers todos.

```javascript
import { take, put } from 'redux-saga/effects'

function* watchFirstThreeTodosCreation() {
  for (let i = 0; i < 3; i++) {
    const action = yield take('TODO_CREATED')
  }
  yield put({type: 'SHOW_CONGRATULATION'})
}
```

Au lieu d'un `while (true)`, nous lançons une boucle `for`, qui ne sera répétée que trois fois. Après avoir pris les trois premières actions `TODO_CREATED`, `watchFirstThreeTodosCreation` entraînera l'application à afficher un message de félicitations puis se terminer. Cela signifie que le générateur sera collecté et qu'aucune autre observation n'aura lieu.

Un autre avantage de l'approche par traction est que nous pouvons décrire notre flux de contrôle en utilisant un style synchrone familier. Par exemple, supposons que nous souhaitons implémenter un flux de connexion avec deux actions: `LOGIN` et `LOGOUT`. En utilisant `takeEvery` (ou `redux-thunk`), nous allons écrire deux tâches séparées (ou thunk): une pour `LOGIN` et l'autre pour `LOGOUT`.

Le résultat est que notre logique est maintenant répartie en deux endroits. Pour que quelqu'un qui lit notre code le comprenne, ils devraient lire la source des deux gestionnaires et faire le lien entre la logique dans les deux dans leur tête. En d'autres termes, cela signifie qu'ils devraient reconstruire le modèle de l'écoulement dans leur tête en réarrangeant mentalement la logique placée en différents endroits du code dans le bon ordre.

En utilisant le modèle de traction, nous pouvons écrire notre flux au même endroit au lieu de manipuler la même action à plusieurs reprises.

```javascript
function* loginFlow() {
  while (true) {
    yield take('LOGIN')
    // ... perform the login logic
    yield take('LOGOUT')
    // ... perform the logout logic
  }
}
```

La saga `loginFlow` transmet plus clairement la séquence d'action attendue. Il sait que l'action `LOGIN` devrait toujours être suivie d'une action `LOGOUT`, et que `LOGOUT` est toujours suivi d'un `LOGIN` (une bonne interface devrait toujours imposer un ordre cohérent des actions, en masquant ou en désactivant actions inattendues).

## Non-blocking calls

Dans la section précédente, nous avons vu comment l'effet `take` nous permet de mieux décrire un flux non trivial dans un endroit central.

Revisiter l'exemple de flux de connexion:

```javascript
function* loginFlow() {
  while (true) {
    yield take('LOGIN')
    // ... perform the login logic
    yield take('LOGOUT')
    // ... perform the logout logic
  }
}
```

Lorsque l'utilisateur se déconnecte, nous supprimons le jeton d'autorisation précédemment stocké.

### First try

Jusqu'à présent, nous avons tous besoin d'effets afin de mettre en œuvre le flux ci-dessus. Nous pouvons attendre des actions spécifiques dans le store en utilisant l'effet `take`. Nous pouvons faire des appels asynchrones en utilisant l'effet `call`. Enfin, nous pouvons envoyer des actions au store en utilisant l'effet `put`.

Alors essayons:

> Remarque: le code ci-dessous a un problème subtil. Assurez-vous de lire la section jusqu'à la fin.

```javascript
import { take, call, put } from 'redux-saga/effects'
import Api from '...'

function* authorize(user, password) {
  try {
    const token = yield call(Api.authorize, user, password)
    yield put({type: 'LOGIN_SUCCESS', token})
    return token
  } catch(error) {
    yield put({type: 'LOGIN_ERROR', error})
  }
}

function* loginFlow() {
  while (true) {
    const {user, password} = yield take('LOGIN_REQUEST')
    const token = yield call(authorize, user, password)
    if (token) {
      yield call(Api.storeItem, {token})
      yield take('LOGOUT')
      yield call(Api.clearItem, 'token')
    }
  }
}
```

Nous avons d'abord créé un générateur séparé `authorize` qui exécutera l'appel API réel et notifiera le store en cas de succès.

Le `loginFlow` implémente tout son flux dans une boucle `while (true)`, ce qui signifie que lorsque nous atteignons la dernière étape du flux (`LOGOUT`), nous commençons une nouvelle itération en attendant une nouvelle action `LOGIN_REQUEST`.

`loginFlow` attend d'abord une action `LOGIN_REQUEST`. Récupère ensuite les informations d'identification dans la charge utile de l'action (`user` et `password`) et effectue un appel à la tâche `authorize`.

Comme vous l'avez noté, `call` n'est pas seulement pour invoquer des fonctions retournant Promises. Nous pouvons également l'utiliser pour invoquer d'autres fonctions de générateur. Dans l'exemple ci-dessus, **`loginFlow` attendra l'autorisation jusqu'à ce qu'il se termine et renvoie** (c'est-à-dire après avoir effectué l'appel api, envoyant l'action puis renvoyant le token à `loginFlow`).

Si l'appel de l'API réussit, `authorize` enverra une action `LOGIN_SUCCESS`, puis retournera le jeton récupéré. S'il en résulte une erreur, il enverra une action `LOGIN_ERROR`.

Si l'appel à `authorize` réussit, `loginFlow` va stocker le jeton retourné dans le stockage DOM et attendre une action `LOGOUT`. Lorsque l'utilisateur se déconnecte, nous supprimons le jeton stocké et attendons une nouvelle connexion de l'utilisateur.

En cas d'échec de `authorize`, il retournera une valeur indéfinie, ce qui fera que `loginFlow` ignorera le processus précédent et attendra une nouvelle action `LOGIN_REQUEST`.

Observez comment la logique entière est stockée dans un endroit. Un nouveau développeur lisant notre code n'a pas besoin de voyager entre différents endroits pour comprendre le flux de contrôle. C'est comme lire un algorithme synchrone: les étapes sont présentées dans leur ordre naturel. Et nous avons des fonctions qui appellent d'autres fonctions et attendent leurs résultats.

#### But there is still a subtle issue with the above approach

Supposons que lorsque `loginFlow` attend l'appel suivant pour résoudre:

```javascript
function* loginFlow() {
  while (true) {
    // ...
    try {
      const token = yield call(authorize, user, password)
      // ...
    }
    // ...
  }
}
```

L'utilisateur clique sur le bouton `Déconnexion`, ce qui provoque l'envoi d'une action `LOGOUT`.

L'exemple suivant illustre la séquence hypothétique des événements:

```text
UI                              loginFlow
--------------------------------------------------------
LOGIN_REQUEST...................call authorize.......... waiting to resolve
........................................................
........................................................
LOGOUT.................................................. missed!
........................................................
................................authorize returned...... dispatch a `LOGIN_SUCCESS`!!
........................................................
```

Lorsque `loginFlow` est bloqué lors de l'appel `authorize`, un éventuel `LOGOUT` se produisant entre l'appel et la réponse sera manqué, car `loginFlow` n'a pas encore exécuté `yield take ('LOGOUT')`.

Le problème avec le code ci-dessus est que `call` est un effet bloquant. c'est-à-dire que le générateur ne peut pas exécuter / gérer quoi que ce soit d'autre jusqu'à ce que l'appel se termine. Mais dans notre cas, nous ne voulons pas que `loginFlow` exécute l'appel d'autorisation, mais surveille aussi une éventuelle action `LOGOUT` qui pourrait se produire au milieu de cet appel. C'est parce que `LOGOUT` est *concurrent* à l'appel `authorize`.

Il est donc nécessaire de commencer `authorize` sans bloquer, donc `loginFlow` peut continuer et surveiller une action `LOGOUT` éventuelle / simultanée.

Pour exprimer des appels non bloquants, la bibliothèque fournit un autre effet: [`fork`](https://redux-saga.js.org/docs/api/index.html#forkfn-args). Lorsque nous générons une *tâche*, la tâche est démarrée en arrière-plan et l'appelant peut continuer son flux sans attendre que la tâche bifurquée se termine.

Donc, pour que `loginFlow` ne rate pas un `LOGOUT` concourant, nous ne devons pas 'appeler' la tâche `authorize`, mais plutôt la `forker`.

```javascript
import { fork, call, take, put } from 'redux-saga/effects'

function* loginFlow() {
  while (true) {
    ...
    try {
      // non-blocking call, what's the returned value here ?
      const ?? = yield fork(authorize, user, password)
      ...
    }
    ...
  }
}
```

Le problème est maintenant que notre action `authorize` est démarrée en arrière-plan, nous ne pouvons pas obtenir le résultat `token` (parce que nous devrions l'attendre). Nous devons donc déplacer l'opération de stockage de jetons dans la tâche `authorize`.

```javascript
import { fork, call, take, put } from 'redux-saga/effects'
import Api from '...'

function* authorize(user, password) {
  try {
    const token = yield call(Api.authorize, user, password)
    yield put({type: 'LOGIN_SUCCESS', token})
    yield call(Api.storeItem, {token})
  } catch(error) {
    yield put({type: 'LOGIN_ERROR', error})
  }
}

function* loginFlow() {
  while (true) {
    const {user, password} = yield take('LOGIN_REQUEST')
    yield fork(authorize, user, password)
    yield take(['LOGOUT', 'LOGIN_ERROR'])
    yield call(Api.clearItem, 'token')
  }
}
```

We're also doing `yield take(['LOGOUT', 'LOGIN_ERROR'])`. It means we are watching for 2 concurrent actions:

- If the `authorize` task succeeds before the user logs out, it'll dispatch a `LOGIN_SUCCESS` action, then terminate. Our `loginFlow` saga will then wait only for a future `LOGOUT` action (because `LOGIN_ERROR` will never happen).

- If the `authorize` fails before the user logs out, it will dispatch a `LOGIN_ERROR` action, then terminate. So `loginFlow` will take the `LOGIN_ERROR` before the `LOGOUT` then it will enter in a another `while` iteration and will wait for the next `LOGIN_REQUEST` action.

- If the user logs out before the `authorize` terminate, then `loginFlow` will take a `LOGOUT` action and also wait for the next `LOGIN_REQUEST`.

Note the call for `Api.clearItem` is supposed to be idempotent. It'll have no effect if no token was stored by the `authorize` call. `loginFlow` makes sure no token will be in the storage before waiting for the next login.

But we're not yet done. If we take a `LOGOUT` in the middle of an API call, we have to **cancel** the `authorize` process, otherwise we'll have 2 concurrent tasks evolving in parallel: The `authorize` task will continue running and upon a successful (resp. failed) result, will dispatch a `LOGIN_SUCCESS` (resp. a `LOGIN_ERROR`) action leading to an inconsistent state.

In order to cancel a forked task, we use a dedicated Effect [`cancel`](https://redux-saga.js.org/docs/api/index.html#canceltask)

```javascript
import { take, put, call, fork, cancel } from 'redux-saga/effects'

// ...

function* loginFlow() {
  while (true) {
    const {user, password} = yield take('LOGIN_REQUEST')
    // fork return a Task object
    const task = yield fork(authorize, user, password)
    const action = yield take(['LOGOUT', 'LOGIN_ERROR'])
    if (action.type === 'LOGOUT')
      yield cancel(task)
    yield call(Api.clearItem, 'token')
  }
}
```

`yield fork` results in a [Task Object](https://redux-saga.js.org/docs/api/index.html#task). We assign the returned object into a local constant `task`. Later if we take a `LOGOUT` action, we pass that task to the `cancel` Effect. If the task is still running, it'll be aborted. If the task has already completed then nothing will happen and the cancellation will result in a no-op. And finally, if the task completed with an error, then we do nothing, because we know the task already completed.

We are *almost* done (concurrency is not that easy; you have to take it seriously).

Suppose that when we receive a `LOGIN_REQUEST` action, our reducer sets some `isLoginPending` flag to true so it can display some message or spinner in the UI. If we get a `LOGOUT` in the middle of an API call and abort the task by *killing it* (i.e. the task is stopped right away), then we may end up again with an inconsistent state. We'll still have `isLoginPending` set to true and our reducer will be waiting for an outcome action (`LOGIN_SUCCESS` or `LOGIN_ERROR`).

Fortunately, the `cancel` Effect won't brutally kill our `authorize` task, it'll instead give it a chance to perform its cleanup logic. The cancelled task can handle any cancellation logic (as well as any other type of completion) in its `finally` block. Since a finally block execute on any type of completion (normal return, error, or forced cancellation), there is an Effect `cancelled` which you can use if you want handle cancellation in a special way:

```javascript
import { take, call, put, cancelled } from 'redux-saga/effects'
import Api from '...'

function* authorize(user, password) {
  try {
    const token = yield call(Api.authorize, user, password)
    yield put({type: 'LOGIN_SUCCESS', token})
    yield call(Api.storeItem, {token})
    return token
  } catch(error) {
    yield put({type: 'LOGIN_ERROR', error})
  } finally {
    if (yield cancelled()) {
      // ... put special cancellation handling code here
    }
  }
}
```

You may have noticed that we haven't done anything about clearing our `isLoginPending` state. For that, there are at least two possible solutions:

- dispatch a dedicated action `RESET_LOGIN_PENDING`
- make the reducer clear the `isLoginPending` on a `LOGOUT` action

## Running Tasks In Parallel

The `yield` statement is great for representing asynchronous control flow in a linear style, but we also need to do things in parallel. We can't write:

```javascript
// wrong, effects will be executed in sequence
const users = yield call(fetch, '/users'),
      repos = yield call(fetch, '/repos')
```

Because the 2nd effect will not get executed until the first call resolves. Instead we have to write:

```javascript
import { all, call } from 'redux-saga/effects'

// correct, effects will get executed in parallel
const [users, repos] = yield all([
  call(fetch, '/users'),
  call(fetch, '/repos')
])
```

When we yield an array of effects, the generator is blocked until all the effects are resolved or as soon as one is rejected (just like how `Promise.all` behaves).

### Starting a race between multiple Effects

Sometimes we start multiple tasks in parallel but we don't want to wait for all of them, we just need
to get the *winner*: the first one that resolves (or rejects). The `race` Effect offers a way of
triggering a race between multiple Effects.

The following sample shows a task that triggers a remote fetch request, and constrains the response within a
1 second timeout.

```javascript
import { race, call, put, delay } from 'redux-saga/effects'

function* fetchPostsWithTimeout() {
  const {posts, timeout} = yield race({
    posts: call(fetchApi, '/posts'),
    timeout: delay(1000)
  })

  if (posts)
    yield put({type: 'POSTS_RECEIVED', posts})
  else
    yield put({type: 'TIMEOUT_ERROR'})
}
```

Another useful feature of `race` is that it automatically cancels the loser Effects. For example,
suppose we have 2 UI buttons:

- The first starts a task in the background that runs in an endless loop `while (true)`
(e.g. syncing some data with the server each x seconds).

- Once the background task is started, we enable a second button which will cancel the task


```javascript
import { race, take, call } from 'redux-saga/effects'

function* backgroundTask() {
  while (true) { ... }
}

function* watchStartBackgroundTask() {
  while (true) {
    yield take('START_BACKGROUND_TASK')
    yield race({
      task: call(backgroundTask),
      cancel: take('CANCEL_TASK')
    })
  }
}
```

In the case a `CANCEL_TASK` action is dispatched, the `race` Effect will automatically cancel
`backgroundTask` by throwing a cancellation error inside it.

## Sequencing Sagas via `yield*`

You can use the builtin `yield*` operator to compose multiple Sagas in a sequential way. This allows you to sequence your *macro-tasks* in a procedural style.

```javascript
function* playLevelOne() { ... }

function* playLevelTwo() { ... }

function* playLevelThree() { ... }

function* game() {
  const score1 = yield* playLevelOne()
  yield put(showScore(score1))

  const score2 = yield* playLevelTwo()
  yield put(showScore(score2))

  const score3 = yield* playLevelThree()
  yield put(showScore(score3))
}
```

Note that using `yield*` will cause the JavaScript runtime to *spread* the whole sequence. The resulting iterator (from `game()`) will yield all values from the nested iterators. A more powerful alternative is to use the more generic middleware composition mechanism.

## Composing Sagas

While using `yield*` provides an idiomatic way of composing Sagas, this approach has some limitations:

- You'll likely want to test nested generators separately. This leads to some duplication in the test code as well as the overhead of the duplicated execution. We don't want to execute a nested generator but only make sure the call to it was issued with the right argument.

- More importantly, `yield*` allows only for sequential composition of tasks, so you can only `yield*` to one generator at a time.

You can use `yield` to start one or more subtasks in parallel. When yielding a call to a generator, the Saga will wait for the generator to terminate before progressing, then resume with the returned value (or throws if an error propagates from the subtask).

```javascript
function* fetchPosts() {
  yield put(actions.requestPosts())
  const products = yield call(fetchApi, '/products')
  yield put(actions.receivePosts(products))
}

function* watchFetch() {
  while (yield take(FETCH_POSTS)) {
    yield call(fetchPosts) // waits for the fetchPosts task to terminate
  }
}
```

Yielding to an array of nested generators will start all the sub-generators in parallel, wait
for them to finish, then resume with all the results

```javascript
function* mainSaga(getState) {
  const results = yield all([call(task1), call(task2), ...])
  yield put(showResults(results))
}
```

In fact, yielding Sagas is no different than yielding other effects (future actions, timeouts, etc). This means you can combine those Sagas with all the other types using the effect combinators.

For example, you may want the user to finish some game in a limited amount of time:

```javascript
function* game(getState) {
  let finished
  while (!finished) {
    // has to finish in 60 seconds
    const {score, timeout} = yield race({
      score: call(play, getState),
      timeout: delay(60000)
    })

    if (!timeout) {
      finished = true
      yield put(showScore(score))
    }
  }
}
```

# Task cancellation

We saw already an example of cancellation in the [Non blocking calls](NonBlockingCalls.md) section. In this section we'll review cancellation in more detail.

Once a task is forked, you can abort its execution using `yield cancel(task)`.

To see how it works, let's consider a basic example: A background sync which can be started/stopped by some UI commands. Upon receiving a `START_BACKGROUND_SYNC` action, we fork a background task that will periodically sync some data from a remote server.

The task will execute continually until a `STOP_BACKGROUND_SYNC` action is triggered. Then we cancel the background task and wait again for the next `START_BACKGROUND_SYNC` action.

```javascript
import { take, put, call, fork, cancel, cancelled, delay } from 'redux-saga/effects'
import { someApi, actions } from 'somewhere'

function* bgSync() {
  try {
    while (true) {
      yield put(actions.requestStart())
      const result = yield call(someApi)
      yield put(actions.requestSuccess(result))
      yield delay(5000)
    }
  } finally {
    if (yield cancelled())
      yield put(actions.requestFailure('Sync cancelled!'))
  }
}

function* main() {
  while ( yield take(START_BACKGROUND_SYNC) ) {
    // starts the task in the background
    const bgSyncTask = yield fork(bgSync)

    // wait for the user stop action
    yield take(STOP_BACKGROUND_SYNC)
    // user clicked stop. cancel the background task
    // this will cause the forked bgSync task to jump into its finally block
    yield cancel(bgSyncTask)
  }
}
```

In the above example, cancellation of `bgSyncTask` will cause the Generator to jump to the finally block. Here you can use `yield cancelled()` to check if the Generator has been cancelled or not.

Cancelling a running task will also cancel the current Effect where the task is blocked at the moment of cancellation.

For example, suppose that at a certain point in an application's lifetime, we have this pending call chain:

```javascript
function* main() {
  const task = yield fork(subtask)
  ...
  // later
  yield cancel(task)
}

function* subtask() {
  ...
  yield call(subtask2) // currently blocked on this call
  ...
}

function* subtask2() {
  ...
  yield call(someApi) // currently blocked on this call
  ...
}
```

`yield cancel(task)` triggers a cancellation on `subtask`, which in turn triggers a cancellation on `subtask2`.

So we saw that Cancellation propagates downward (in contrast returned values and uncaught errors propagates upward). You can see it as a *contract* between the caller (which invokes the async operation) and the callee (the invoked operation). The callee is responsible for performing the operation. If it has completed (either success or error) the outcome propagates up to its caller and eventually to the caller of the caller and so on. That is, callees are responsible for *completing the flow*.

Now if the callee is still pending and the caller decides to cancel the operation, it triggers a kind of a signal that propagates down to the callee (and possibly to any deep operations called by the callee itself). All deeply pending operations will be cancelled.

There is another direction where the cancellation propagates to as well: the joiners of a task (those blocked on a `yield join(task)`) will also be cancelled if the joined task is cancelled. Similarly, any potential callers of those joiners will be cancelled as well (because they are blocked on an operation that has been cancelled from outside).

### Testing generators with fork effect

When `fork` is called it starts the task in the background and also returns task object like we have learned previously. When testing this we have to use utility function `createMockTask`. Object returned from this function should be passed to next `next` call after fork test. Mock task can then be passed to `cancel` for example. Here is test for `main` generator which is on top of this page.

```javascript
import { createMockTask } from 'redux-saga/utils';

describe('main', () => {
  const generator = main();

  it('waits for start action', () => {
    const expectedYield = take(START_BACKGROUND_SYNC);
    expect(generator.next().value).to.deep.equal(expectedYield);
  });

  it('forks the service', () => {
    const expectedYield = fork(bgSync);
    const mockedAction = { type: 'START_BACKGROUND_SYNC' };
    expect(generator.next(mockedAction).value).to.deep.equal(expectedYield);
  });

  it('waits for stop action and then cancels the service', () => {
    const mockTask = createMockTask();

    const expectedTakeYield = take(STOP_BACKGROUND_SYNC);
    expect(generator.next(mockTask).value).to.deep.equal(expectedTakeYield);

    const expectedCancelYield = cancel(mockTask);
    expect(generator.next().value).to.deep.equal(expectedCancelYield);
  });
});
```

You can also use mock task's functions `setRunning`, `setResult` and `setError` to set mock task's state. For example `mockTask.setRunning(false)`.

#### Note

It's important to remember that `yield cancel(task)` doesn't wait for the cancelled task to finish (i.e. to perform its finally block). The cancel effect behaves like fork. It returns as soon as the cancel was initiated. Once cancelled, a task should normally return as soon as it finishes its cleanup logic.

### Automatic cancellation

Besides manual cancellation there are cases where cancellation is triggered automatically

1. In a `race` effect. All race competitors, except the winner, are automatically cancelled.

2. In a parallel effect (`yield all([...])`). The parallel effect is rejected as soon as one of the sub-effects is rejected (as implied by `Promise.all`). In this case, all the other sub-effects are automatically cancelled.

## redux-saga's fork model

In `redux-saga` you can dynamically fork tasks that execute in the background using 2 Effects

- `fork` is used to create *attached forks*
- `spawn` is used to create *detached forks*

### Attached forks (using `fork`)

Attached forks remain attached to their parent by the following rules

#### Completion

- A Saga terminates only after
  - It terminates its own body of instructions
  - All attached forks are themselves terminated

For example say we have the following

```js
import { fork, call, put, delay } from 'redux-saga/effects'
import api from './somewhere/api' // app specific
import { receiveData } from './somewhere/actions' // app specific

function* fetchAll() {
  const task1 = yield fork(fetchResource, 'users')
  const task2 = yield fork(fetchResource, 'comments')
  yield delay(1000)
}

function* fetchResource(resource) {
  const {data} = yield call(api.fetch, resource)
  yield put(receiveData(data))
}

function* main() {
  yield call(fetchAll)
}
```

`call(fetchAll)` will terminate after:

- The `fetchAll` body itself terminates, this means all 3 effects are performed. Since `fork` effects are non blocking, the
task will block on `delay(1000)`

- The 2 forked tasks terminate, i.e. after fetching the required resources and putting the corresponding `receiveData` actions

So the whole task will block until a delay of 1000 millisecond passed *and* both `task1` and `task2` finished their business.

Say for example, the delay of 1000 milliseconds elapsed and the 2 tasks haven't yet finished, then `fetchAll` will still wait
for all forked tasks to finish before terminating the whole task.

The attentive reader might have noticed the `fetchAll` saga could be rewritten using the parallel Effect

```js
function* fetchAll() {
  yield all([
    call(fetchResource, 'users'),     // task1
    call(fetchResource, 'comments'),  // task2,
    delay(1000)
  ])
}
```

In fact, attached forks shares the same semantics with the parallel Effect:

- We're executing tasks in parallel
- The parent will terminate after all launched tasks terminate


And this applies for all other semantics as well (error and cancellation propagation). You can understand how
attached forks behave by considering it as a *dynamic parallel* Effect.

### Error propagation

Following the same analogy, Let's examine in detail how errors are handled in parallel Effects

for example, let's say we have this Effect

```js
yield all([
  call(fetchResource, 'users'),
  call(fetchResource, 'comments'),
  delay(1000)
])
```

The above effect will fail as soon as any one of the 3 child Effects fails. Furthermore, the uncaught error will cause
the parallel Effect to cancel all the other pending Effects. So for example if `call(fetchResource, 'users')` raises an
uncaught error, the parallel Effect will cancel the 2 other tasks (if they are still pending) then aborts itself with the
same error from the failed call.

Similarly for attached forks, a Saga aborts as soon as

- Its main body of instructions throws an error

- An uncaught error was raised by one of its attached forks

So in the previous example

```js
//... imports

function* fetchAll() {
  const task1 = yield fork(fetchResource, 'users')
  const task2 = yield fork(fetchResource, 'comments')
  yield delay(1000)
}

function* fetchResource(resource) {
  const {data} = yield call(api.fetch, resource)
  yield put(receiveData(data))
}

function* main() {
  try {
    yield call(fetchAll)
  } catch (e) {
    // handle fetchAll errors
  }
}
```

If at a moment, for example, `fetchAll` is blocked on the `delay(1000)` Effect, and say, `task1` failed, then the whole
`fetchAll` task will fail causing

- Cancellation of all other pending tasks. This includes:
  - The *main task* (the body of `fetchAll`): cancelling it means cancelling the current Effect `delay(1000)`
  - The other forked tasks which are still pending. i.e. `task2` in our example.

- The `call(fetchAll)` will raise itself an error which will be caught in the `catch` body of `main`

Note we're able to catch the error from `call(fetchAll)` inside `main` only because we're using a blocking call. And that
we can't catch the error directly from `fetchAll`. This is a rule of thumb, **you can't catch errors from forked tasks**. A failure
in an attached fork will cause the forking parent to abort (Just like there is no way to catch an error *inside* a parallel Effect, only from
outside by blocking on the parallel Effect).


### Cancellation

Cancelling a Saga causes the cancellation of:

- The *main task* this means cancelling the current Effect where the Saga is blocked

- All attached forks that are still executing


**WIP**

### Detached forks (using `spawn`)

Detached forks live in their own execution context. A parent doesn't wait for detached forks to terminate. Uncaught
errors from spawned tasks are not bubbled up to the parent. And cancelling a parent doesn't automatically cancel detached
forks (you need to cancel them explicitly).

In short, detached forks behave like root Sagas started directly using the `middleware.run` API.


**WIP**

## Concurrency

In the basics section, we saw how to use the helper effects `takeEvery` and `takeLatest` in order to manage concurrency between Effects.

In this section we'll see how those helpers could be implemented using the low-level Effects.

### `takeEvery`

```javascript
import {fork, take} from "redux-saga/effects"

const takeEvery = (pattern, saga, ...args) => fork(function*() {
  while (true) {
    const action = yield take(pattern)
    yield fork(saga, ...args.concat(action))
  }
})
```

`takeEvery` allows multiple `saga` tasks to be forked concurrently.

### `takeLatest`

```javascript
import {cancel, fork, take} from "redux-saga/effects"

const takeLatest = (pattern, saga, ...args) => fork(function*() {
  let lastTask
  while (true) {
    const action = yield take(pattern)
    if (lastTask) {
      yield cancel(lastTask) // cancel is no-op if the task has already terminated
    }
    lastTask = yield fork(saga, ...args.concat(action))
  }
})
```

`takeLatest` doesn't allow multiple Saga tasks to be fired concurrently. As soon as it gets a new dispatched action, it cancels any previously-forked task (if still running).

`takeLatest` can be useful to handle AJAX requests where we want to only have the response to the latest request.

## Testing Sagas

There are two main ways to test Sagas: testing the saga generator function step-by-step or running the full saga and
asserting the side effects.

### Testing the Saga Generator Function

Suppose we have the following actions:
 
```javascript
const CHOOSE_COLOR = 'CHOOSE_COLOR';
const CHANGE_UI = 'CHANGE_UI';

const chooseColor = (color) => ({
  type: CHOOSE_COLOR,
  payload: {
    color,
  },
});

const changeUI = (color) => ({
  type: CHANGE_UI,
  payload: {
    color,
  },
});
```

We want to test the saga:

```javascript
function* changeColorSaga() {
  const action = yield take(CHOOSE_COLOR);
  yield put(changeUI(action.payload.color));
}
```

Since Sagas always yield an Effect, and these effects have basic factory functions (e.g. put, take etc.) a test may
inspect the yielded effect and compare it to an expected effect. To get the first yielded value from a saga,
call its `next().value`:

```javascript
  const gen = changeColorSaga();

  assert.deepEqual(
    gen.next().value,
    take(CHOOSE_COLOR),
    'it should wait for a user to choose a color'
  );
```

A value must then be returned to assign to the `action` constant, which is used for the argument to the `put` effect:

```javascript
  const color = 'red';
  assert.deepEqual(
    gen.next(chooseColor(color)).value,
    put(changeUI(color)),
    'it should dispatch an action to change the ui'
  );
```

Since there are no more `yield`s, then next time `next()` is called, the generator will be done:

```javascript
  assert.deepEqual(
    gen.next().done,
    true,
    'it should be done'
  );
```

#### Branching Saga 

Sometimes your saga will have different outcomes. To test the different branches without repeating all the steps that lead to it you can use the utility function **cloneableGenerator**

This time we add two new actions, `CHOOSE_NUMBER` and `DO_STUFF`, with a related action creators:

```javascript
const CHOOSE_NUMBER = 'CHOOSE_NUMBER';
const DO_STUFF = 'DO_STUFF';

const chooseNumber = (number) => ({
  type: CHOOSE_NUMBER,
  payload: {
    number,
  },
});

const doStuff = () => ({
  type: DO_STUFF, 
});
```

Now the saga under test will put two `DO_STUFF` actions before waiting for a `CHOOSE_NUMBER` action and then putting
either `changeUI('red')` or `changeUI('blue')`, depending on whether the number is even or odd.

```javascript
function* doStuffThenChangeColor() {
  yield put(doStuff());
  yield put(doStuff());
  const action = yield take(CHOOSE_NUMBER);
  if (action.payload.number % 2 === 0) {
    yield put(changeUI('red'));
  } else {
    yield put(changeUI('blue'));
  }
}
```

The test is as follows:

```javascript
import { put, take } from 'redux-saga/effects';
import { cloneableGenerator } from 'redux-saga/utils';

test('doStuffThenChangeColor', assert => {
  const gen = cloneableGenerator(doStuffThenChangeColor)();
  gen.next(); // DO_STUFF
  gen.next(); // DO_STUFF
  gen.next(); // CHOOSE_NUMBER

  assert.test('user choose an even number', a => {
    // cloning the generator before sending data
    const clone = gen.clone();
    a.deepEqual(
      clone.next(chooseNumber(2)).value,
      put(changeUI('red')),
      'should change the color to red'
    );

    a.equal(
      clone.next().done,
      true,
      'it should be done'
    );

    a.end();
  });

  assert.test('user choose an odd number', a => {
    const clone = gen.clone();
    a.deepEqual(
      clone.next(chooseNumber(3)).value,
      put(changeUI('blue')),
      'should change the color to blue'
    );

    a.equal(
      clone.next().done,
      true,
      'it should be done'
    );

    a.end();
  });
});
```

See also: [Task cancellation](TaskCancellation.md) for testing fork effects

### Testing the full Saga

Although it may be useful to test each step of a saga, in practise this makes for brittle tests. Instead, it may be
preferable to run the whole saga and assert that the expected effects have occurred.

Suppose we have a basic saga which calls an HTTP API:

```javascript
function* callApi(url) {
  const someValue = yield select(somethingFromState);
  try {
    const result = yield call(myApi, url, someValue);
    yield put(success(result.json()));
    return result.status;
  } catch (e) {
    yield put(error(e));
    return -1;
  }
}
```

We can run the saga with mocked values:

```javascript
const dispatched = [];

const saga = runSaga({
  dispatch: (action) => dispatched.push(action),
  getState: () => ({ value: 'test' }),
}, callApi, 'http://url');
```

A test could then be written to assert the dispatched actions and mock calls:

```javascript
import sinon from 'sinon';
import * as api from './api';

test('callApi', async (assert) => {
  const dispatched = [];
  sinon.stub(api, 'myApi').callsFake(() => ({
    json: () => ({
      some: 'value'
    })
  }));
  const url = 'http://url';
  const result = await runSaga({
    dispatch: (action) => dispatched.push(action),
    getState: () => ({ state: 'test' }),
  }, callApi, url).done;

  assert.true(myApi.calledWith(url, somethingFromState({ state: 'test' })));
  assert.deepEqual(dispatched, [success({ some: 'value' })]);
});
```

See also: Repository Examples:

https://github.com/redux-saga/redux-saga/blob/master/examples/counter/test/sagas.js

https://github.com/redux-saga/redux-saga/blob/master/examples/shopping-cart/test/sagas.js

### Testing libraries

While both of the above testing methods can be written natively, there exist several libraries to make both methods easier. Additionally, some libraries can be used to test sagas in a *third* way: recording specific side-effects (but not all).

Sam Hogarth's (@sh1989) [article](http://blog.scottlogic.com/2018/01/16/evaluating-redux-saga-test-libraries.html) summarizes the different options well.

For testing each generator yield step-by-step there is [`redux-saga-test`][1] and [`redux-saga-testing`][2]. [`redux-saga-test-engine`][3] is for recording and testing for specific side effects. For an integration test, [`redux-saga-tester`][4]. And [`redux-saga-test-plan`][5] can actually cover all three bases.

#### `redux-saga-test` and `redux-saga-testing` for step-by-step testing

The `redux-saga-test` library provides syntactic sugar for your step-by-step tests. The `fromGenerator` function returns a value that can be iterated manually with `.next()` and have an assertion made using the relevant saga effect method.

```javascript
import fromGenerator from 'redux-saga-test';

test('with redux-saga-test', () => {
  const generator = callApi('url');
  /* 
  * The assertions passed to fromGenerator
  * requires a `deepEqual` method
  */
  const expect = fromGenerator(assertions, generator);

  expect.next().select(somethingFromState);
  expect.next(selectedData).call(myApi, 'url', selectedData);
  expect.next(result).put(success(result.json));
});
```

`redux-saga-testing` library provides a method `sagaHelper` that takes your generator and returns a value that works a lot like Jest's `it()` function, but also advances the generator being tested. The `result` parameter passed into the callback is the value yielded by the generater

```javascript
import sagaHelper from 'redux-saga-testing';

test('with redux-saga-testing', () => {
  const it = sagaHelper(callApi());

  it('should select from state', selectResult => {
    // with Jest's `expect`
    expect(selectResult).toBe(value);
  });
  
  it('should select from state', apiResponse => {
    // without tape's `test`
    assert.deepEqual(apiResponse.json(), jsonResponse);
  });

  // an empty call to `it` can be used to skip an effect
  it('', () => {});
});
```

#### `redux-saga-test-plan`

This is the most versatile library. The `testSaga` API is used for exact order testing and `expectSaga` is for both recording side-effects and integration testing.

```javascript
import { expectSaga, testSaga } from 'redux-saga-testing';

test('exact order with redux-saga-test-plan', () => {
  return testSaga(callApi, 'url')
    .next()
    .select(selectFromState)
    .next()
    .call(myApi, 'url', valueFromSelect);
    
    ...
});

test('recorded effects with redux-saga-test-plan', () => {
  /*
  * With expectSaga, you can assert that any yield from
  * your saga occurs as expected, *regardless of order*.
  * You must call .run() at the end.
  */
  return expectSaga(callApi, 'url')
    .put(success(value)) // last effect from our saga, first one tested
    
    .call(myApi, 'url', value)
    .run();
    /* notice no assertion for the select call */
});

test('test only final effect with .provide()', () => {
  /* 
  * With the .provide() method from expectSaga
  * you can by pass in all expected values
  * and test only your saga's final effect.
  */
  return expectSaga(callApi, 'url')
    .provide([
      [select(selectFromState), selectedValue],
      [call(myApi, 'url', selectedValue), response]
    ])
    .put(success(response))
    .run();
});

test('integration test with withReducer', () => {
  /*
  * Using `withReducer` allows you to test
  * the state shape upon completion of your reducer -
  * a true integration test for your Redux store management.
  */

  return expectSaga(callApi, 'url')
    .withReducer(myReducer)
    .provide([
      [call(myApi, 'url', value), response]
    ])
    .hasFinalState({
      data: response
    })
    .run();
});
```


#### redux-saga-test-engine

This library functions very similarly in setup to `redux-saga-test-plan`, but is best used to record effects. Provide a collection of saga generic effects to be watched by `createSagaTestEngine` function which in turn returns a function. Then provide your saga and specific effects and their arguments.

```javascript
const collectedEffects  = createSagaTestEngine(['SELECT', 'CALL', 'PUT']);
const actualEffects = collectEffects(mySaga, [ [myEffect(arg), value], ... ], argsToMySaga);
```

The value of `actualEffects` is an array containing elements equal to the yielded values from all *collected* effects, in order of occurence.

```javascript
import createSagaTestEngine from 'redux-saga-test-engine';

test('testing with redux-saga-test-engine', () => {
  const collectEffects = createSagaTestEngine(['CALL', 'PUT']);

  const actualEffects = collectEffects(
    callApi,
    [
      [select(selectFromState), selectedValue],
      [call(myApi, 'url', selectedValue), response]
    ],
    // Any further args are passed to the saga
    // Here it is our URL, but typically would be the dispatched action
    'url'
  );
  
  // assert that the effects you care about occurred as expected, in order
  assert.equal(actualEffects[0], call(myApi, 'url', selectedValue));
  assert.equal(actualEffects[1], put(success, response));

  // assert that your saga does nothing unexpected
  assert.true(actualEffects.length === 2);
});
```

#### redux-saga-tester

A final library to consider for integration testing. this library provides a `sagaTester` class, to which you instantiate with your store's initial state and your reducer.

To test your saga, the `sagaTester` instance `start()` method with your saga and its argument(s). This runs your saga to its end. Then you may assert that effects occured, actions were dispatched and the state was updated as expected.

```javascript
import SagaTester from 'redux-saga-tester';

test('with redux-saga-tester', () => {
  const sagaTester = new SagaTester({
    initialState: defaultState,
    reducers: reducer
  });

  sagaTester.start(callApi);

  sagaTester.dispatch(actionToTriggerSaga());

  await sagaTester.waitFor(success);

  assert.true(sagaTester.wasCalled(success(response)));

  assert.deepEqual(sagaTester.getState(), { data: response });
});
```

### `effectMiddlwares`

This is a feature currently in the v1.0.0-beta release. This would provide a native way to perform integration like testing without one of the above libraries.

The idea is that you can create a real redux store with saga middleware in your test file. The saga middlware takes an object as an argument. That object would have an `effectMiddlewares` value: a function where you can intercept/hijack any effect and resolve it on your own - passing it very redux-style to the next middleware.

In your test, you would start a saga, intercept/resolve async effects with effectMiddlewares and assert on things like state updates to test integration between your saga and a store.

Here's an example from the [docs](https://github.com/redux-saga/redux-saga/blob/34c9093684323ab92eacdf2df958f31d9873d3b1/test/interpreter/effectMiddlewares.js#L88):

```javascript
test('effectMiddleware', assert => {
  assert.plan(1);

  let actual = [];

  function rootReducer(state = {}, action) {
    return action;
  }

  const effectMiddleware = next => effect => {
    if (effect === apiCall) {
      Promise.resolve().then(() => next('injected value'));
      return;
    }
    return next(effect);
  };

  const middleware = sagaMiddleware({ effectMiddlewares: [effectMiddleware] });
  const store = createStore(rootReducer, {}, applyMiddleware(middleware));

  const apiCall = call(() => new Promise(() => {}));

  function* root() {
    actual.push(yield all([call(fnA), apiCall]));
  }

  function* fnA() {
    const result = [];
    result.push((yield take('ACTION-1')).val);
    result.push((yield take('ACTION-2')).val);
    return result;
  }

  const task = middleware.run(root)

  Promise.resolve()
    .then(() => store.dispatch({ type: 'ACTION-1', val: 1 }))
    .then(() => store.dispatch({ type: 'ACTION-2', val: 2 }));

  const expected = [[[1, 2], 'injected value']];

  task
    .toPromise()
    .then(() => {
      assert.deepEqual(
        actual,
        expected,
        'effectMiddleware must be able to intercept and resolve effect in a custom way',
      )
    })
    .catch(err => assert.fail(err));
});
```

 [1]: https://github.com/stoeffel/redux-saga-test
 [2]: https://github.com/antoinejaussoin/redux-saga-testing/
 [3]: https://github.com/DNAinfo/redux-saga-test-engine
 [4]: https://github.com/wix/redux-saga-tester
 [5]: https://github.com/jfairbank/redux-saga-test-plan

 ## Connecting Sagas to external Input/Output

We saw that `take` Effects are resolved by waiting for actions to be dispatched to the Store. And that `put` Effects are resolved by dispatching the actions provided as argument.

When a Saga is started (either at startup or later dynamically), the middleware automatically connects its `take`/`put` to the store. The 2 Effects can be seen as a sort of Input/Output to the Saga.

`redux-saga` provides a way to run a Saga outside of the Redux middleware environment and connect it to a custom Input/Output.

```javascript
import { runSaga } from 'redux-saga'

function* saga() { ... }

const myIO = {
  subscribe: ..., // this will be used to resolve take Effects
  dispatch: ...,  // this will be used to resolve put Effects
  getState: ...,  // this will be used to resolve select Effects
}

runSaga(
  myIO,
  saga,
)
```

For more info, see the [API docs](https://redux-saga.js.org/docs/api/index.html##runsagaoptions-saga-args).

## Using Channels

Until now we've used the `take` and `put` effects to communicate with the Redux Store. Channels generalize those Effects to communicate with external event sources or between Sagas themselves. They can also be used to queue specific actions from the Store.

In this section, we'll see:

- How to use the `yield actionChannel` Effect to buffer specific actions from the Store.

- How to use the `eventChannel` factory function to connect `take` Effects to external event sources.

- How to create a channel using the generic `channel` factory function and use it in `take`/`put` Effects to
communicate between two Sagas.

### Using the `actionChannel` Effect

Let's review the canonical example:

```javascript
import { take, fork, ... } from 'redux-saga/effects'

function* watchRequests() {
  while (true) {
    const {payload} = yield take('REQUEST')
    yield fork(handleRequest, payload)
  }
}

function* handleRequest(payload) { ... }
```

The above example illustrates the typical *watch-and-fork* pattern. The `watchRequests` saga is using `fork` to avoid blocking and thus not missing any action from the store. A `handleRequest` task is created on each `REQUEST` action. So if there are many actions fired at a rapid rate there can be many `handleRequest` tasks executing concurrently.

Imagine now that our requirement is as follows: we want to process `REQUEST` serially. If we have at any moment four actions, we want to handle the first `REQUEST` action, then only after finishing this action we process the second action and so on...

So we want to *queue* all non-processed actions, and once we're done with processing the current request, we get the next message from the queue.

Redux-Saga provides a little helper Effect `actionChannel`, which can handle this for us. Let's see how we can rewrite the previous example with it:

```javascript
import { take, actionChannel, call, ... } from 'redux-saga/effects'

function* watchRequests() {
  // 1- Create a channel for request actions
  const requestChan = yield actionChannel('REQUEST')
  while (true) {
    // 2- take from the channel
    const {payload} = yield take(requestChan)
    // 3- Note that we're using a blocking call
    yield call(handleRequest, payload)
  }
}

function* handleRequest(payload) { ... }
```

The first thing is to create the action channel. We use `yield actionChannel(pattern)` where pattern is interpreted using the same rules we mentioned previously with `take(pattern)`. The difference between the 2 forms is that `actionChannel` **can buffer incoming messages** if the Saga is not yet ready to take them (e.g. blocked on an API call).

Next is the `yield take(requestChan)`. Besides usage with a `pattern` to take specific actions from the Redux Store, `take` can also be used with channels (above we created a channel object from specific Redux actions). The `take` will block the Saga until a message is available on the channel. The take may also resume immediately if there is a message stored in the underlying buffer.

The important thing to note is how we're using a blocking `call`. The Saga will remain blocked until `call(handleRequest)` returns. But meanwhile, if other `REQUEST` actions are dispatched while the Saga is still blocked, they will queued internally by `requestChan`. When the Saga resumes from `call(handleRequest)` and executes the next `yield take(requestChan)`, the take will resolve with the queued message.

By default, `actionChannel` buffers all incoming messages without limit. If you want a more control over the buffering, you can supply a Buffer argument to the effect creator. Redux-Saga provides some common buffers (none, dropping, sliding) but you can also supply your own buffer implementation. [See API docs](../api#buffers) for more details.

For example if you want to handle only the most recent five items you can use:

```javascript
import { buffers } from 'redux-saga'
import { actionChannel } from 'redux-saga/effects'

function* watchRequests() {
  const requestChan = yield actionChannel('REQUEST', buffers.sliding(5))
  ...
}
```

### Using the `eventChannel` factory to connect to external events

Like `actionChannel` (Effect), `eventChannel` (a factory function, not an Effect) creates a Channel for events but from event sources other than the Redux Store.

This basic example creates a Channel from an interval:

```javascript
import { eventChannel, END } from 'redux-saga'

function countdown(secs) {
  return eventChannel(emitter => {
      const iv = setInterval(() => {
        secs -= 1
        if (secs > 0) {
          emitter(secs)
        } else {
          // this causes the channel to close
          emitter(END)
        }
      }, 1000);
      // The subscriber must return an unsubscribe function
      return () => {
        clearInterval(iv)
      }
    }
  )
}
```

The first argument in `eventChannel` is a *subscriber* function. The role of the subscriber is to initialize the external event source (above using `setInterval`), then routes all incoming events from the source to the channel by invoking the supplied `emitter`. In the above example we're invoking `emitter` on each second.

> Note: You need to sanitize your event sources as to not pass null or undefined through the event channel. While it's fine to pass numbers through, we'd recommend structuring your event channel data like your redux actions. `{ number }` over `number`.

Note also the invocation `emitter(END)`. We use this to notify any channel consumer that the channel has been closed, meaning no other messages will come through this channel.

Let's see how we can use this channel from our Saga. (This is taken from the cancellable-counter example in the repo.)

```javascript
import { take, put, call } from 'redux-saga/effects'
import { eventChannel, END } from 'redux-saga'

// creates an event Channel from an interval of seconds
function countdown(seconds) { ... }

export function* saga() {
  const chan = yield call(countdown, value)
  try {    
    while (true) {
      // take(END) will cause the saga to terminate by jumping to the finally block
      let seconds = yield take(chan)
      console.log(`countdown: ${seconds}`)
    }
  } finally {
    console.log('countdown terminated')
  }
}
```

So the Saga is yielding a `take(chan)`. This causes the Saga to block until a message is put on the channel. In our example above, it corresponds to when we invoke `emitter(secs)`. Note also we're executing the whole `while (true) {...}` loop inside a `try/finally` block. When the interval terminates, the countdown function closes the event channel by invoking `emitter(END)`. Closing a channel has the effect of terminating all Sagas blocked on a `take` from that channel. In our example, terminating the Saga will cause it to jump to its `finally` block (if provided, otherwise the Saga terminates).

The subscriber returns an `unsubscribe` function. This is used by the channel to unsubscribe before the event source complete. Inside a Saga consuming messages from an event channel, if we want to *exit early* before the event source complete (e.g. Saga has been cancelled) you can call `chan.close()` to close the channel and unsubscribe from the source.

For example, we can make our Saga support cancellation:

```javascript
import { take, put, call, cancelled } from 'redux-saga/effects'
import { eventChannel, END } from 'redux-saga'

// creates an event Channel from an interval of seconds
function countdown(seconds) { ... }

export function* saga() {
  const chan = yield call(countdown, value)
  try {    
    while (true) {
      let seconds = yield take(chan)
      console.log(`countdown: ${seconds}`)
    }
  } finally {
    if (yield cancelled()) {
      chan.close()
      console.log('countdown cancelled')
    }    
  }
}
```

Here is another example of how you can use event channels to pass WebSocket events into your saga (e.g.: using socket.io library).
Suppose you are waiting for a server message `ping` then reply with a `pong` message after some delay.


```javascript
import { take, put, call, apply, delay } from 'redux-saga/effects'
import { eventChannel } from 'redux-saga'
import { createWebSocketConnection } from './socketConnection'

// this function creates an event channel from a given socket
// Setup subscription to incoming `ping` events
function createSocketChannel(socket) {
  // `eventChannel` takes a subscriber function
  // the subscriber function takes an `emit` argument to put messages onto the channel
  return eventChannel(emit => {

    const pingHandler = (event) => {
      // puts event payload into the channel
      // this allows a Saga to take this payload from the returned channel
      emit(event.payload)
    }

    // setup the subscription
    socket.on('ping', pingHandler)

    // the subscriber must return an unsubscribe function
    // this will be invoked when the saga calls `channel.close` method
    const unsubscribe = () => {
      socket.off('ping', pingHandler)
    }

    return unsubscribe
  })
}

// reply with a `pong` message by invoking `socket.emit('pong')`
function* pong(socket) {
  yield delay(5000)
  yield apply(socket, socket.emit, ['pong']) // call `emit` as a method with `socket` as context
}

export function* watchOnPings() {
  const socket = yield call(createWebSocketConnection)
  const socketChannel = yield call(createSocketChannel, socket)

  while (true) {
    const payload = yield take(socketChannel)
    yield put({ type: INCOMING_PONG_PAYLOAD, payload })
    yield fork(pong, socket)
  }
}
```

> Note: messages on an eventChannel are not buffered by default. You have to provide a buffer to the eventChannel factory in order to specify buffering strategy for the channel (e.g. `eventChannel(subscriber, buffer)`).
[See the API docs](../api#buffers) for more info.

#### Using channels to communicate between Sagas

Besides action channels and event channels. You can also directly create channels which are not connected to any source by default. You can then manually `put` on the channel. This is handy when you want to use a channel to communicate between sagas.

To illustrate, let's review the former example of request handling.

```javascript
import { take, fork, ... } from 'redux-saga/effects'

function* watchRequests() {
  while (true) {
    const {payload} = yield take('REQUEST')
    yield fork(handleRequest, payload)
  }
}

function* handleRequest(payload) { ... }
```

We saw that the watch-and-fork pattern allows handling multiple requests simultaneously, without limit on the number of worker tasks executing concurrently. Then, we used the `actionChannel` effect to limit the concurrency to one task at a time.

So let's say that our requirement is to have a maximum of three tasks executing at the same time. When we get a request and there are less than three tasks executing, we process the request immediately, otherwise we queue the task and wait for one of the three *slots* to become free.

Below is an example of a solution using channels:

```javascript
import { channel } from 'redux-saga'
import { take, fork, ... } from 'redux-saga/effects'

function* watchRequests() {
  // create a channel to queue incoming requests
  const chan = yield call(channel)

  // create 3 worker 'threads'
  for (var i = 0; i < 3; i++) {
    yield fork(handleRequest, chan)
  }

  while (true) {
    const {payload} = yield take('REQUEST')
    yield put(chan, payload)
  }
}

function* handleRequest(chan) {
  while (true) {
    const payload = yield take(chan)
    // process the request
  }
}
```

In the above example, we create a channel using the `channel` factory. We get back a channel which by default buffers all messages we put on it (unless there is a pending taker, in which the taker is resumed immediately with the message).

The `watchRequests` saga then forks three worker sagas. Note the created channel is supplied to all forked sagas. `watchRequests` will use this channel to *dispatch* work to the three worker sagas. On each `REQUEST` action the Saga will put the payload on the channel. The payload will then be taken by any *free* worker. Otherwise it will be queued by the channel until a worker Saga is ready to take it.

All the three workers run a typical while loop. On each iteration, a worker will take the next request, or will block until a message is available. Note that this mechanism provides an automatic load-balancing between the 3 workers. Rapid workers are not slowed down by slow workers.