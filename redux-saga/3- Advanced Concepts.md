Pulling future actions
Until now we've used the helper effect takeEvery in order to spawn a new task on each incoming action. This mimics somewhat the behavior of redux-thunk: each time a Component, for example, invokes a fetchProducts Action Creator, the Action Creator will dispatch a thunk to execute the control flow.

In reality, takeEvery is just a wrapper effect for internal helper function built on top of the lower-level and more powerful API. In this section we'll see a new Effect, take, which makes it possible to build complex control flow by allowing total control of the action observation process.

A simple logger
Let's take a simple example of a Saga that watches all actions dispatched to the store and logs them to the console.

Using takeEvery('*') (with the wildcard * pattern), we can catch all dispatched actions regardless of their types.

import { select, takeEvery } from 'redux-saga/effects'

function* watchAndLog() {
  yield takeEvery('*', function* logger(action) {
    const state = yield select()

    console.log('action', action)
    console.log('state after', state)
  })
}
Now let's see how to use the take Effect to implement the same flow as above:

import { select, take } from 'redux-saga/effects'

function* watchAndLog() {
  while (true) {
    const action = yield take('*')
    const state = yield select()

    console.log('action', action)
    console.log('state after', state)
  }
}
The take is just like call and put we saw earlier. It creates another command object that tells the middleware to wait for a specific action. The resulting behavior of the call Effect is the same as when the middleware suspends the Generator until a Promise resolves. In the take case, it'll suspend the Generator until a matching action is dispatched. In the above example, watchAndLog is suspended until any action is dispatched.

Note how we're running an endless loop while (true). Remember, this is a Generator function, which doesn't have a run-to-completion behavior. Our Generator will block on each iteration waiting for an action to happen.

Using take has a subtle impact on how we write our code. In the case of takeEvery, the invoked tasks have no control on when they'll be called. They will be invoked again and again on each matching action. They also have no control on when to stop the observation.

In the case of take, the control is inverted. Instead of the actions being pushed to the handler tasks, the Saga is pulling the action by itself. It looks as if the Saga is performing a normal function call action = getNextAction() which will resolve when the action is dispatched.

This inversion of control allows us to implement control flows that are non-trivial to do with the traditional push approach.

As a simple example, suppose that in our Todo application, we want to watch user actions and show a congratulation message after the user has created his first three todos.

import { take, put } from 'redux-saga/effects'

function* watchFirstThreeTodosCreation() {
  for (let i = 0; i < 3; i++) {
    const action = yield take('TODO_CREATED')
  }
  yield put({type: 'SHOW_CONGRATULATION'})
}
Instead of a while (true), we're running a for loop, which will iterate only three times. After taking the first three TODO_CREATED actions, watchFirstThreeTodosCreation will cause the application to display a congratulation message then terminate. This means the Generator will be garbage collected and no more observation will take place.

Another benefit of the pull approach is that we can describe our control flow using a familiar synchronous style. For example, suppose we want to implement a login flow with two actions: LOGIN and LOGOUT. Using takeEvery (or redux-thunk), we'll have to write two separate tasks (or thunks): one for LOGIN and the other for LOGOUT.

The result is that our logic is now spread in two places. In order for someone reading our code to understand it, they would have to read the source of the two handlers and make the link between the logic in both in their head. In other words, it means they would have to rebuild the model of the flow in their head by rearranging mentally the logic placed in various places of the code in the correct order.

Using the pull model, we can write our flow in the same place instead of handling the same action repeatedly.

function* loginFlow() {
  while (true) {
    yield take('LOGIN')
    // ... perform the login logic
    yield take('LOGOUT')
    // ... perform the logout logic
  }
}
The loginFlow Saga more clearly conveys the expected action sequence. It knows that the LOGIN action should always be followed by a LOGOUT action, and that LOGOUT is always followed by a LOGIN (a good UI should always enforce a consistent order of the actions, by hiding or disabling unexpected actions).

Non-blocking calls
In the previous section, we saw how the take Effect allows us to better describe a non-trivial flow in a central place.

Revisiting the login flow example:

function* loginFlow() {
  while (true) {
    yield take('LOGIN')
    // ... perform the login logic
    yield take('LOGOUT')
    // ... perform the logout logic
  }
}
Let's complete the example and implement the actual login/logout logic. Suppose we have an API which permits us to authorize the user on a remote server. If the authorization is successful, the server will return an authorization token which will be stored by our application using DOM storage (assume our API provides another service for DOM storage).

When the user logs out, we'll simply delete the authorization token stored previously.

First try
So far we have all needed Effects in order to implement the above flow. We can wait for specific actions in the store using the take Effect. We can make asynchronous calls using the call Effect. Finally, we can dispatch actions to the store using the put Effect.

So let's give it a try:

Note: the code below has a subtle issue. Make sure to read the section until the end.

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
First we created a separate Generator authorize which will perform the actual API call and notify the Store upon success.

The loginFlow implements its entire flow inside a while (true) loop, which means once we reach the last step in the flow (LOGOUT) we start a new iteration by waiting for a new LOGIN_REQUEST action.

loginFlow first waits for a LOGIN_REQUEST action. Then retrieves the credentials in the action payload (user and password) and makes a call to the authorize task.

As you noted, call isn't only for invoking functions returning Promises. We can also use it to invoke other Generator functions. In the above example, loginFlow will wait for authorize until it terminates and returns (i.e. after performing the api call, dispatching the action and then returning the token to loginFlow).

If the API call succeeds, authorize will dispatch a LOGIN_SUCCESS action then return the fetched token. If it results in an error, it'll dispatch a LOGIN_ERROR action.

If the call to authorize is successful, loginFlow will store the returned token in the DOM storage and wait for a LOGOUT action. When the user logouts, we remove the stored token and wait for a new user login.

In the case of authorize failed, it'll return an undefined value, which will cause loginFlow to skip the previous process and wait for a new LOGIN_REQUEST action.

Observe how the entire logic is stored in one place. A new developer reading our code doesn't have to travel between various places in order to understand the control flow. It's like reading a synchronous algorithm: steps are laid out in their natural order. And we have functions which call other functions and wait for their results.

But there is still a subtle issue with the above approach
Suppose that when the loginFlow is waiting for the following call to resolve:

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
The user clicks on the Logout button causing a LOGOUT action to be dispatched.

The following example illustrates the hypothetical sequence of the events:

UI                              loginFlow
--------------------------------------------------------
LOGIN_REQUEST...................call authorize.......... waiting to resolve
........................................................
........................................................
LOGOUT.................................................. missed!
........................................................
................................authorize returned...... dispatch a `LOGIN_SUCCESS`!!
........................................................
When loginFlow is blocked on the authorize call, an eventual LOGOUT occurring in between the call and the response will be missed, because loginFlow hasn't yet performed the yield take('LOGOUT').

The problem with the above code is that call is a blocking Effect. i.e. the Generator can't perform/handle anything else until the call terminates. But in our case we do not only want loginFlow to execute the authorization call, but also watch for an eventual LOGOUT action that may occur in the middle of this call. That's because LOGOUT is concurrent to the authorize call.

So what's needed is some way to start authorize without blocking so loginFlow can continue and watch for an eventual/concurrent LOGOUT action.

To express non-blocking calls, the library provides another Effect: fork. When we fork a task, the task is started in the background and the caller can continue its flow without waiting for the forked task to terminate.

So in order for loginFlow to not miss a concurrent LOGOUT, we must not call the authorize task, instead we have to fork it.

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
The issue now is since our authorize action is started in the background, we can't get the token result (because we'd have to wait for it). So we need to move the token storage operation into the authorize task.

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
We're also doing yield take(['LOGOUT', 'LOGIN_ERROR']). It means we are watching for 2 concurrent actions:

If the authorize task succeeds before the user logs out, it'll dispatch a LOGIN_SUCCESS action, then terminate. Our loginFlow saga will then wait only for a future LOGOUT action (because LOGIN_ERROR will never happen).

If the authorize fails before the user logs out, it will dispatch a LOGIN_ERROR action, then terminate. So loginFlow will take the LOGIN_ERROR before the LOGOUT then it will enter in a another while iteration and will wait for the next LOGIN_REQUEST action.

If the user logs out before the authorize terminate, then loginFlow will take a LOGOUT action and also wait for the next LOGIN_REQUEST.

Note the call for Api.clearItem is supposed to be idempotent. It'll have no effect if no token was stored by the authorize call. loginFlow makes sure no token will be in the storage before waiting for the next login.

But we're not yet done. If we take a LOGOUT in the middle of an API call, we have to cancel the authorize process, otherwise we'll have 2 concurrent tasks evolving in parallel: The authorize task will continue running and upon a successful (resp. failed) result, will dispatch a LOGIN_SUCCESS (resp. a LOGIN_ERROR) action leading to an inconsistent state.

In order to cancel a forked task, we use a dedicated Effect cancel

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
yield fork results in a Task Object. We assign the returned object into a local constant task. Later if we take a LOGOUT action, we pass that task to the cancel Effect. If the task is still running, it'll be aborted. If the task has already completed then nothing will happen and the cancellation will result in a no-op. And finally, if the task completed with an error, then we do nothing, because we know the task already completed.

We are almost done (concurrency is not that easy; you have to take it seriously).

Suppose that when we receive a LOGIN_REQUEST action, our reducer sets some isLoginPending flag to true so it can display some message or spinner in the UI. If we get a LOGOUT in the middle of an API call and abort the task by simply killing it (i.e. the task is stopped right away), then we may end up again with an inconsistent state. We'll still have isLoginPending set to true and our reducer will be waiting for an outcome action (LOGIN_SUCCESS or LOGIN_ERROR).

Fortunately, the cancel Effect won't brutally kill our authorize task, it'll instead give it a chance to perform its cleanup logic. The cancelled task can handle any cancellation logic (as well as any other type of completion) in its finally block. Since a finally block execute on any type of completion (normal return, error, or forced cancellation), there is an Effect cancelled which you can use if you want handle cancellation in a special way:

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
You may have noticed that we haven't done anything about clearing our isLoginPending state. For that, there are at least two possible solutions:

dispatch a dedicated action RESET_LOGIN_PENDING
more simply, make the reducer clear the isLoginPending on a LOGOUT action

Running Tasks In Parallel
The yield statement is great for representing asynchronous control flow in a simple and linear style, but we also need to do things in parallel. We can't simply write:

// wrong, effects will be executed in sequence
const users = yield call(fetch, '/users'),
      repos = yield call(fetch, '/repos')
Because the 2nd effect will not get executed until the first call resolves. Instead we have to write:

import { all, call } from 'redux-saga/effects'

// correct, effects will get executed in parallel
const [users, repos] = yield all([
  call(fetch, '/users'),
  call(fetch, '/repos')
])
When we yield an array of effects, the generator is blocked until all the effects are resolved or as soon as one is rejected (just like how Promise.all behaves).

Starting a race between multiple Effects
Sometimes we start multiple tasks in parallel but we don't want to wait for all of them, we just need to get the winner: the first one that resolves (or rejects). The race Effect offers a way of triggering a race between multiple Effects.

The following sample shows a task that triggers a remote fetch request, and constrains the response within a 1 second timeout.

import { race, take, put } from 'redux-saga/effects'
import { delay } from 'redux-saga'

function* fetchPostsWithTimeout() {
  const {posts, timeout} = yield race({
    posts: call(fetchApi, '/posts'),
    timeout: call(delay, 1000)
  })

  if (posts)
    put({type: 'POSTS_RECEIVED', posts})
  else
    put({type: 'TIMEOUT_ERROR'})
}
Another useful feature of race is that it automatically cancels the loser Effects. For example, suppose we have 2 UI buttons:

The first starts a task in the background that runs in an endless loop while (true) (e.g. syncing some data with the server each x seconds).

Once the background task is started, we enable a second button which will cancel the task

import { race, take, put } from 'redux-saga/effects'

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
In the case a CANCEL_TASK action is dispatched, the race Effect will automatically cancel backgroundTask by throwing a cancellation error inside it.

Sequencing Sagas via yield*
You can use the builtin yield* operator to compose multiple Sagas in a sequential way. This allows you to sequence your macro-tasks in a simple procedural style.

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
Note that using yield* will cause the JavaScript runtime to spread the whole sequence. The resulting iterator (from game()) will yield all values from the nested iterators. A more powerful alternative is to use the more generic middleware composition mechanism.

Composing Sagas
While using yield* provides an idiomatic way of composing Sagas, this approach has some limitations:

You'll likely want to test nested generators separately. This leads to some duplication in the test code as well as the overhead of the duplicated execution. We don't want to execute a nested generator but only make sure the call to it was issued with the right argument.

More importantly, yield* allows only for sequential composition of tasks, so you can only yield* to one generator at a time.

You can simply use yield to start one or more subtasks in parallel. When yielding a call to a generator, the Saga will wait for the generator to terminate before progressing, then resume with the returned value (or throws if an error propagates from the subtask).

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
Yielding to an array of nested generators will start all the sub-generators in parallel, wait for them to finish, then resume with all the results

function* mainSaga(getState) {
  const results = yield all([call(task1), call(task2), ...])
  yield put(showResults(results))
}
In fact, yielding Sagas is no different than yielding other effects (future actions, timeouts, etc). This means you can combine those Sagas with all the other types using the effect combinators.

For example, you may want the user to finish some game in a limited amount of time:

function* game(getState) {
  let finished
  while (!finished) {
    // has to finish in 60 seconds
    const {score, timeout} = yield race({
      score: call(play, getState),
      timeout: call(delay, 60000)
    })

    if (!timeout) {
      finished = true
      yield put(showScore(score))
    }
  }
}

Task cancellation
We saw already an example of cancellation in the Non blocking calls section. In this section we'll review cancellation in more detail.

Once a task is forked, you can abort its execution using yield cancel(task).

To see how it works, let's consider a simple example: A background sync which can be started/stopped by some UI commands. Upon receiving a START_BACKGROUND_SYNC action, we fork a background task that will periodically sync some data from a remote server.

The task will execute continually until a STOP_BACKGROUND_SYNC action is triggered. Then we cancel the background task and wait again for the next START_BACKGROUND_SYNC action.

import { take, put, call, fork, cancel, cancelled } from 'redux-saga/effects'
import { delay } from 'redux-saga'
import { someApi, actions } from 'somewhere'

function* bgSync() {
  try {
    while (true) {
      yield put(actions.requestStart())
      const result = yield call(someApi)
      yield put(actions.requestSuccess(result))
      yield call(delay, 5000)
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
In the above example, cancellation of bgSyncTask will cause the Generator to jump to the finally block. Here you can use yield cancelled() to check if the Generator has been cancelled or not.

Cancelling a running task will also cancel the current Effect where the task is blocked at the moment of cancellation.

For example, suppose that at a certain point in an application's lifetime, we have this pending call chain:

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
yield cancel(task) triggers a cancellation on subtask, which in turn triggers a cancellation on subtask2.

So we saw that Cancellation propagates downward (in contrast returned values and uncaught errors propagates upward). You can see it as a contract between the caller (which invokes the async operation) and the callee (the invoked operation). The callee is responsible for performing the operation. If it has completed (either success or error) the outcome propagates up to its caller and eventually to the caller of the caller and so on. That is, callees are responsible for completing the flow.

Now if the callee is still pending and the caller decides to cancel the operation, it triggers a kind of a signal that propagates down to the callee (and possibly to any deep operations called by the callee itself). All deeply pending operations will be cancelled.

There is another direction where the cancellation propagates to as well: the joiners of a task (those blocked on a yield join(task)) will also be cancelled if the joined task is cancelled. Similarly, any potential callers of those joiners will be cancelled as well (because they are blocked on an operation that has been cancelled from outside).

Testing generators with fork effect
When fork is called it starts the task in the background and also returns task object like we have learned previously. When testing this we have to use utility function createMockTask. Object returned from this function should be passed to next next call after fork test. Mock task can then be passed to cancel for example. Here is test for main generator which is on top of this page.

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
You can also use mock task's functions setRunning, setResult and setError to set mock task's state. For example mockTask.setRunning(false).

Note
It's important to remember that yield cancel(task) doesn't wait for the cancelled task to finish (i.e. to perform its finally block). The cancel effect behaves like fork. It returns as soon as the cancel was initiated. Once cancelled, a task should normally return as soon as it finishes its cleanup logic.

Automatic cancellation
Besides manual cancellation there are cases where cancellation is triggered automatically

In a race effect. All race competitors, except the winner, are automatically cancelled.

In a parallel effect (yield all([...])). The parallel effect is rejected as soon as one of the sub-effects is rejected (as implied by Promise.all). In this case, all the other sub-effects are automatically cancelled.

redux-saga's fork model
In redux-saga you can dynamically fork tasks that execute in the background using 2 Effects

fork is used to create attached forks
spawn is used to create detached forks
Attached forks (using fork)
Attached forks remains attached to their parent by the following rules

Completion
A Saga terminates only after
It terminates its own body of instructions
All attached forks are themselves terminated
For example say we have the following

import { delay } from 'redux-saga'
import { fork, call, put } from 'redux-saga/effects'
import api from './somewhere/api' // app specific
import { receiveData } from './somewhere/actions' // app specific

function* fetchAll() {
  const task1 = yield fork(fetchResource, 'users')
  const task2 = yield fork(fetchResource, 'comments')
  yield call(delay, 1000)
}

function* fetchResource(resource) {
  const {data} = yield call(api.fetch, resource)
  yield put(receiveData(data))
}

function* main() {
  yield call(fetchAll)
}
call(fetchAll) will terminate after:

The fetchAll body itself terminates, this means all 3 effects are performed. Since fork effects are non blocking, the task will block on call(delay, 1000)

The 2 forked tasks terminate, i.e. after fetching the required resources and putting the corresponding receiveData actions

So the whole task will block until a delay of 1000 millisecond passed and both task1 and task2 finished their business.

Say for example, the delay of 1000 milliseconds elapsed and the 2 tasks haven't yet finished, then fetchAll will still wait for all forked tasks to finish before terminating the whole task.

The attentive reader might have noticed the fetchAll saga could be rewritten using the parallel Effect

function* fetchAll() {
  yield all([
    call(fetchResource, 'users'),     // task1
    call(fetchResource, 'comments'),  // task2,
    call(delay, 1000)
  ])
}
In fact, attached forks shares the same semantics with the parallel Effect:

We're executing tasks in parallel
The parent will terminate after all launched tasks terminate
And this applies for all other semantics as well (error and cancellation propagation). You can understand how attached forks behave by simply considering it as a dynamic parallel Effect.

Error propagation
Following the same analogy, Let's examine in detail how errors are handled in parallel Effects

for example, let's say we have this Effect

yield all([
  call(fetchResource, 'users'),
  call(fetchResource, 'comments'),
  call(delay, 1000)
])
The above effect will fail as soon as any one of the 3 child Effects fails. Furthermore, the uncaught error will cause the parallel Effect to cancel all the other pending Effects. So for example if call(fetchResource, 'users') raises an uncaught error, the parallel Effect will cancel the 2 other tasks (if they are still pending) then aborts itself with the same error from the failed call.

Similarly for attached forks, a Saga aborts as soon as

Its main body of instructions throws an error

An uncaught error was raised by one of its attached forks

So in the previous example

//... imports

function* fetchAll() {
  const task1 = yield fork(fetchResource, 'users')
  const task2 = yield fork(fetchResource, 'comments')
  yield call(delay, 1000)
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
If at a moment, for example, fetchAll is blocked on the call(delay, 1000) Effect, and say, task1 failed, then the whole fetchAll task will fail causing

Cancellation of all other pending tasks. This includes:

The main task (the body of fetchAll): cancelling it means cancelling the current Effect call(delay, 1000)
The other forked tasks which are still pending. i.e. task2 in our example.
The call(fetchAll) will raise itself an error which will be caught in the catch body of main

Note we're able to catch the error from call(fetchAll) inside main only because we're using a blocking call. And that we can't catch the error directly from fetchAll. This is a rule of thumb, you can't catch errors from forked tasks. A failure in an attached fork will cause the forking parent to abort (Just like there is no way to catch an error inside a parallel Effect, only from outside by blocking on the parallel Effect).

Cancellation
Cancelling a Saga causes the cancellation of:

The main task this means cancelling the current Effect where the Saga is blocked

All attached forks that are still executing

WIP

Detached forks (using spawn)
Detached forks live in their own execution context. A parent doesn't wait for detached forks to terminate. Uncaught errors from spawned tasks are not bubbled up to the parent. And cancelling a parent doesn't automatically cancel detached forks (you need to cancel them explicitly).

In short, detached forks behave like root Sagas started directly using the middleware.run API.

WIP

Concurrency
In the basics section, we saw how to use the helper effects takeEvery and takeLatest in order to manage concurrency between Effects.

In this section we'll see how those helpers could be implemented using the low-level Effects.

takeEvery
import {fork, take} from "redux-saga/effects"

const takeEvery = (pattern, saga, ...args) => fork(function*() {
  while (true) {
    const action = yield take(pattern)
    yield fork(saga, ...args.concat(action))
  }
})
takeEvery allows multiple saga tasks to be forked concurrently.

takeLatest
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
takeLatest doesn't allow multiple Saga tasks to be fired concurrently. As soon as it gets a new dispatched action, it cancels any previously-forked task (if still running).

takeLatest can be useful to handle AJAX requests where we want to only have the response to the latest request.

Testing Sagas
There are two main ways to test Sagas: testing the saga generator function step-by-step or running the full saga and asserting the side effects.

Testing the Saga Generator Function
Suppose we have the following actions:

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
We want to test the saga:

function* changeColorSaga() {
  const action = yield take(CHOOSE_COLOR);
  yield put(changeUI(action.payload.color));
}
Since Sagas always yield an Effect, and these effects have simple factory functions (e.g. put, take etc.) a test may inspect the yielded effect and compare it to an expected effect. To get the first yielded value from a saga, call its next().value:

  const gen = changeColorSaga();

  assert.deepEqual(
    gen.next().value,
    take(CHOOSE_COLOR),
    'it should wait for a user to choose a color'
  );
A value must then be returned to assign to the action constant, which is used for the argument to the put effect:

  const color = 'red';
  assert.deepEqual(
    gen.next(chooseColor(color)).value,
    put(changeUI(color)),
    'it should dispatch an action to change the ui'
  );
Since there are no more yields, then next time next() is called, the generator will be done:

  assert.deepEqual(
    gen.next().done,
    true,
    'it should be done'
  );
Branching Saga
Sometimes your saga will have different outcomes. To test the different branches without repeating all the steps that lead to it you can use the utility function cloneableGenerator

This time we add two new actions, CHOOSE_NUMBER and DO_STUFF, with a related action creators:

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
Now the saga under test will put two DO_STUFF actions before waiting for a CHOOSE_NUMBER action and then putting either changeUI('red') or changeUI('blue'), depending on whether the number is even or odd.

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
The test is as follows:

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
See also: Task cancellation for testing fork effects

Testing the full Saga
Although it may be useful to test each step of a saga, in practise this makes for brittle tests. Instead, it may be preferable to run the whole saga and assert that the expected effects have occurred.

Suppose we have a simple saga which calls an HTTP API:

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
We can run the saga with mocked values:

const dispatched = [];

const saga = runSaga({
  dispatch: (action) => dispatched.push(action);
  getState: () => ({ value: 'test' });
}, callApi, 'http://url');
A test could then be written to assert the dispatched actions and mock calls:

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
See also: Repository Examples:

https://github.com/redux-saga/redux-saga/blob/master/examples/counter/test/sagas.js

https://github.com/redux-saga/redux-saga/blob/master/examples/shopping-cart/test/sagas.js

Connecting Sagas to external Input/Output
We saw that take Effects are resolved by waiting for actions to be dispatched to the Store. And that put Effects are resolved by dispatching the actions provided as argument.

When a Saga is started (either at startup or later dynamically), the middleware automatically connects its take/put to the store. The 2 Effects can be seen as a sort of Input/Output to the Saga.

redux-saga provides a way to run a Saga outside of the Redux middleware environment and connect it to a custom Input/Output.

import { runSaga } from 'redux-saga'

function* saga() { ... }

const myIO = {
  subscribe: ..., // this will be used to resolve take Effects
  dispatch: ...,  // this will be used to resolve put Effects
  getState: ...,  // this will be used to resolve select Effects
}

runSaga(
  myIO
  saga,
)
For more info, see the API docs.

Using Channels
Until now we've used the take and put effects to communicate with the Redux Store. Channels generalize those Effects to communicate with external event sources or between Sagas themselves. They can also be used to queue specific actions from the Store.

In this section, we'll see:

How to use the yield actionChannel Effect to buffer specific actions from the Store.

How to use the eventChannel factory function to connect take Effects to external event sources.

How to create a channel using the generic channel factory function and use it in take/put Effects to communicate between two Sagas.

Using the actionChannel Effect
Let's review the canonical example:

import { take, fork, ... } from 'redux-saga/effects'

function* watchRequests() {
  while (true) {
    const {payload} = yield take('REQUEST')
    yield fork(handleRequest, payload)
  }
}

function* handleRequest(payload) { ... }
The above example illustrates the typical watch-and-fork pattern. The watchRequests saga is using fork to avoid blocking and thus not missing any action from the store. A handleRequest task is created on each REQUEST action. So if there are many actions fired at a rapid rate there can be many handleRequest tasks executing concurrently.

Imagine now that our requirement is as follows: we want to process REQUEST serially. If we have at any moment four actions, we want to handle the first REQUEST action, then only after finishing this action we process the second action and so on...

So we want to queue all non-processed actions, and once we're done with processing the current request, we get the next message from the queue.

Redux-Saga provides a little helper Effect actionChannel, which can handle this for us. Let's see how we can rewrite the previous example with it:

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
The first thing is to create the action channel. We use yield actionChannel(pattern) where pattern is interpreted using the same rules we mentioned previously with take(pattern). The difference between the 2 forms is that actionChannel can buffer incoming messages if the Saga is not yet ready to take them (e.g. blocked on an API call).

Next is the yield take(requestChan). Besides usage with a pattern to take specific actions from the Redux Store, take can also be used with channels (above we created a channel object from specific Redux actions). The take will block the Saga until a message is available on the channel. The take may also resume immediately if there is a message stored in the underlying buffer.

The important thing to note is how we're using a blocking call. The Saga will remain blocked until call(handleRequest) returns. But meanwhile, if other REQUEST actions are dispatched while the Saga is still blocked, they will queued internally by requestChan. When the Saga resumes from call(handleRequest) and executes the next yield take(requestChan), the take will resolve with the queued message.

By default, actionChannel buffers all incoming messages without limit. If you want a more control over the buffering, you can supply a Buffer argument to the effect creator. Redux-Saga provides some common buffers (none, dropping, sliding) but you can also supply your own buffer implementation. See API docs for more details.

For example if you want to handle only the most recent five items you can use:

import { buffers } from 'redux-saga'
import { actionChannel } from 'redux-saga/effects'

function* watchRequests() {
  const requestChan = yield actionChannel('REQUEST', buffers.sliding(5))
  ...
}
Using the eventChannel factory to connect to external events
Like actionChannel (Effect), eventChannel (a factory function, not an Effect) creates a Channel for events but from event sources other than the Redux Store.

This simple example creates a Channel from an interval:

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
The first argument in eventChannel is a subscriber function. The role of the subscriber is to initialize the external event source (above using setInterval), then routes all incoming events from the source to the channel by invoking the supplied emitter. In the above example we're invoking emitter on each second.

Note: You need to sanitize your event sources as to not pass null or undefined through the event channel. While it's fine to pass numbers through, we'd recommend structuring your event channel data like your redux actions. { number } over number.

Note also the invocation emitter(END). We use this to notify any channel consumer that the channel has been closed, meaning no other messages will come through this channel.

Let's see how we can use this channel from our Saga. (This is taken from the cancellable-counter example in the repo.)

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
So the Saga is yielding a take(chan). This causes the Saga to block until a message is put on the channel. In our example above, it corresponds to when we invoke emitter(secs). Note also we're executing the whole while (true) {...} loop inside a try/finally block. When the interval terminates, the countdown function closes the event channel by invoking emitter(END). Closing a channel has the effect of terminating all Sagas blocked on a take from that channel. In our example, terminating the Saga will cause it to jump to its finally block (if provided, otherwise the Saga simply terminates).

The subscriber returns an unsubscribe function. This is used by the channel to unsubscribe before the event source complete. Inside a Saga consuming messages from an event channel, if we want to exit early before the event source complete (e.g. Saga has been cancelled) you can call chan.close() to close the channel and unsubscribe from the source.

For example, we can make our Saga support cancellation:

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
Here is another example of how you can use event channels to pass WebSocket events into your saga (e.g.: using socket.io library). Suppose you are waiting for a server message ping then reply with a pong message after some delay.

import { take, put, call, apply } from 'redux-saga/effects'
import { eventChannel, delay } from 'redux-saga'
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
  yield call(delay, 5000)
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
Note: messages on an eventChannel are not buffered by default. You have to provide a buffer to the eventChannel factory in order to specify buffering strategy for the channel (e.g. eventChannel(subscriber, buffer)). See the API docs for more info.

Using channels to communicate between Sagas
Besides action channels and event channels. You can also directly create channels which are not connected to any source by default. You can then manually put on the channel. This is handy when you want to use a channel to communicate between sagas.

To illustrate, let's review the former example of request handling.

import { take, fork, ... } from 'redux-saga/effects'

function* watchRequests() {
  while (true) {
    const {payload} = yield take('REQUEST')
    yield fork(handleRequest, payload)
  }
}

function* handleRequest(payload) { ... }
We saw that the watch-and-fork pattern allows handling multiple requests simultaneously, without limit on the number of worker tasks executing concurrently. Then, we used the actionChannel effect to limit the concurrency to one task at a time.

So let's say that our requirement is to have a maximum of three tasks executing at the same time. When we get a request and there are less than three tasks executing, we process the request immediately, otherwise we queue the task and wait for one of the three slots to become free.

Below is an example of a solution using channels:

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
In the above example, we create a channel using the channel factory. We get back a channel which by default buffers all messages we put on it (unless there is a pending taker, in which the taker is resumed immediately with the message).

The watchRequests saga then forks three worker sagas. Note the created channel is supplied to all forked sagas. watchRequests will use this channel to dispatch work to the three worker sagas. On each REQUEST action the Saga will simply put the payload on the channel. The payload will then be taken by any free worker. Otherwise it will be queued by the channel until a worker Saga is ready to take it.

All the three workers run a typical while loop. On each iteration, a worker will take the next request, or will block until a message is available. Note that this mechanism provides an automatic load-balancing between the 3 workers. Rapid workers are not slowed down by slow workers.