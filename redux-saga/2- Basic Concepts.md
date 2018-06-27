# Basic Concepts

## Using Saga Helpers

redux-saga provides some helper effects wrapping internal functions to spawn tasks when some specific actions are dispatched to the Store.

The helper functions are built on top of the lower level API. In the advanced section, we'll see how those functions can be implemented.

The first function, takeEvery is the most familiar and provides a behavior similar to redux-thunk.

Let's illustrate with the common AJAX example. On each click on a Fetch button we dispatch a FETCH_REQUESTED action. We want to handle this action by launching a task that will fetch some data from the server.

First we create the task that will perform the asynchronous action:

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

To launch the above task on each FETCH_REQUESTED action:

```javascript
import { takeEvery } from 'redux-saga/effects'

function* watchFetchData() {
  yield takeEvery('FETCH_REQUESTED', fetchData)
}
```

In the above example, takeEvery allows multiple fetchData instances to be started concurrently. At a given moment, we can start a new fetchData task while there are still one or more previous fetchData tasks which have not yet terminated.

If we want to only get the response of the latest request fired (e.g. to always display the latest version of data) we can use the takeLatest helper:

```javascript
import { takeLatest } from 'redux-saga/effects'

function* watchFetchData() {
  yield takeLatest('FETCH_REQUESTED', fetchData)
}
```

Unlike takeEvery, takeLatest allows only one fetchData task to run at any moment. And it will be the latest started task. If a previous task is still running when another fetchData task is started, the previous task will be automatically cancelled.

If you have multiple Sagas watching for different actions, you can create multiple watchers with those built-in helpers, which will behave like there was fork used to spawn them (we'll talk about fork later. For now, consider it to be an Effect that allows us to start multiple sagas in the background).

For example:

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

In redux-saga, Sagas are implemented using Generator functions. To express the Saga logic, we yield plain JavaScript Objects from the Generator. We call those Objects Effects. An Effect is simply an object that contains some information to be interpreted by the middleware. You can view Effects like instructions to the middleware to perform some operation (e.g., invoke some asynchronous function, dispatch an action to the store, etc.).

To create Effects, you use the functions provided by the library in the redux-saga/effects package.

In this section and the following, we will introduce some basic Effects. And see how the concept allows the Sagas to be easily tested.

Sagas can yield Effects in multiple forms. The simplest way is to yield a Promise.

For example suppose we have a Saga that watches a PRODUCTS_REQUESTED action. On each matching action, it starts a task to fetch a list of products from a server.

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

In the example above, we are invoking Api.fetch directly from inside the Generator (In Generator functions, any expression at the right of yield is evaluated then the result is yielded to the caller).

Api.fetch('/products') triggers an AJAX request and returns a Promise that will resolve with the resolved response, the AJAX request will be executed immediately. Simple and idiomatic, but...

Suppose we want to test the generator above:

```javascript
const iterator = fetchProducts()
assert.deepEqual(iterator.next().value, ??) // what do we expect ?
```

We want to check the result of the first value yielded by the generator. In our case it's the result of running Api.fetch('/products') which is a Promise . Executing the real service during tests is neither a viable nor practical approach, so we have to mock the Api.fetch function, i.e. we'll have to replace the real function with a fake one which doesn't actually run the AJAX request but only checks that we've called Api.fetch with the right arguments ('/products' in our case).

Mocks make testing more difficult and less reliable. On the other hand, functions that simply return values are easier to test, since we can use a simple equal() to check the result. This is the way to write the most reliable tests.

Not convinced? I encourage you to read Eric Elliott's article:

(...)equal(), by nature answers the two most important questions every unit test must answer, but most don’t:

What is the actual output?
What is the expected output?
If you finish a test without answering those two questions, you don’t have a real unit test. You have a sloppy, half-baked test.

What we actually need is just to make sure the fetchProducts task yields a call with the right function and the right arguments.

Instead of invoking the asynchronous function directly from inside the Generator, we can yield only a description of the function invocation. i.e. We'll simply yield an object which looks like

```javascript
// Effect -> call the function Api.fetch with `./products` as argument
{
  CALL: {
    fn: Api.fetch,
    args: ['./products']
  }
}
```

Put another way, the Generator will yield plain Objects containing instructions, and the redux-saga middleware will take care of executing those instructions and giving back the result of their execution to the Generator. This way, when testing the Generator, all we need to do is to check that it yields the expected instruction by doing a simple deepEqual on the yielded Object.

For this reason, the library provides a different way to perform asynchronous calls.

```javascript
import { call } from 'redux-saga/effects'

function* fetchProducts() {
  const products = yield call(Api.fetch, '/products')
  // ...
}
```

We're using now the call(fn, ...args) function. The difference from the preceding example is that now we're not executing the fetch call immediately, instead, call creates a description of the effect. Just as in Redux you use action creators to create a plain object describing the action that will get executed by the Store, call creates a plain object describing the function call. The redux-saga middleware takes care of executing the function call and resuming the generator with the resolved response.

This allows us to easily test the Generator outside the Redux environment. Because call is just a function which returns a plain Object.

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

Now we don't need to mock anything, and a simple equality test will suffice.

The advantage of those declarative calls is that we can test all the logic inside a Saga by simply iterating over the Generator and doing a deepEqual test on the values yielded successively. This is a real benefit, as your complex asynchronous operations are no longer black boxes, and you can test in detail their operational logic no matter how complex it is.

call also supports invoking object methods, you can provide a this context to the invoked functions using the following form:

```javascript
yield call([obj, obj.method], arg1, arg2, ...) // as if we did obj.method(arg1, arg2 ...)
```

apply is an alias for the method invocation form

```javascript
yield apply(obj, obj.method, [arg1, arg2, ...])
```

call and apply are well suited for functions that return Promise results. Another function cps can be used to handle Node style functions (e.g. fn(...args, callback) where callback is of the form (error, result) => ()). cps stands for Continuation Passing Style.

For example:

```javascript
import { cps } from 'redux-saga/effects'

const content = yield cps(readFile, '/path/to/file')
```

And of course you can test it just like you test call:

```javascript
import { cps } from 'redux-saga/effects'

const iterator = fetchSaga()
assert.deepEqual(iterator.next().value, cps(readFile, '/path/to/file') )
```

cps also supports the same method invocation form as call.

A full list of declarative effects can be found in the API reference.

## Dispatching actions to the store

Taking the previous example further, let's say that after each save, we want to dispatch some action to notify the Store that the fetch has succeeded (we'll omit the failure case for the moment).

We could pass the Store's dispatch function to the Generator. Then the Generator could invoke it after receiving the fetch response:

```javascript
// ...

function* fetchProducts(dispatch) {
  const products = yield call(Api.fetch, '/products')
  dispatch({ type: 'PRODUCTS_RECEIVED', products })
}
```

However, this solution has the same drawbacks as invoking functions directly from inside the Generator (as discussed in the previous section). If we want to test that fetchProducts performs the dispatch after receiving the AJAX response, we'll need again to mock the dispatch function.

Instead, we need the same declarative solution. Just create an Object to instruct the middleware that we need to dispatch some action, and let the middleware perform the real dispatch. This way we can test the Generator's dispatch in the same way: by just inspecting the yielded Effect and making sure it contains the correct instructions.

The library provides, for this purpose, another function put which creates the dispatch Effect.

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

Note how we pass the fake response to the Generator via its next method. Outside the middleware environment, we have total control over the Generator, we can simulate a real environment by simply mocking results and resuming the Generator with them. Mocking data is a lot simpler than mocking functions and spying calls.

## Error handling

In this section we'll see how to handle the failure case from the previous example. Let's suppose that our API function Api.fetch returns a Promise which gets rejected when the remote fetch fails for some reason.

We want to handle those errors inside our Saga by dispatching a PRODUCTS_REQUEST_FAILED action to the Store.

We can catch errors inside the Saga using the familiar try/catch syntax.

```javascript
import Api from './path/to/api'
import { call, put } from 'redux-saga/effects'

// ...

function* fetchProducts() {
  try {
    const products = yield call(Api.fetch, '/products')
    yield put({ type: 'PRODUCTS_RECEIVED', products })
  }
  catch(error) {
    yield put({ type: 'PRODUCTS_REQUEST_FAILED', error })
  }
}
```

In order to test the failure case, we'll use the throw method of the Generator

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

In this case, we're passing the throw method a fake error. This will cause the Generator to break the current flow and execute the catch block.

Of course, you're not forced to handle your API errors inside try/catch blocks. You can also make your API service return a normal value with some error flag on it. For example, you can catch Promise rejections and map them to an object with an error field.


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

To generalize, triggering Side Effects from inside a Saga is always done by yielding some declarative Effect. (You can also yield Promise directly, but this will make testing difficult as we saw in the first section.)

What a Saga does is actually compose all those Effects together to implement the desired control flow. The simplest example is to sequence yielded Effects by just putting the yields one after another. You can also use the familiar control flow operators (if, while, for) to implement more sophisticated control flows.

We saw that using Effects like call and put, combined with high-level APIs like takeEvery allows us to achieve the same things as redux-thunk, but with the added benefit of easy testability.

But redux-saga provides another advantage over redux-thunk. In the Advanced section you'll encounter some more powerful Effects that let you express complex control flows while still allowing the same testability benefit.