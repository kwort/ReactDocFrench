# Redux

Redux est un conteneur d'état prévisible pour les applications JavaScript. (Ne pas confondre avec un framework WordPress - [Redux Framework](https://reduxframework.com/))

Il vous aide à écrire des applications qui se comportent de manière cohérente, s'exécutent dans des environnements différents (client, serveur et natif) et sont faciles à tester. En plus de cela, il fournit une excellente expérience de développement, comme [live code editing combined with a time traveling debugger](https://github.com/reduxjs/redux-devtools).

Vous pouvez utiliser Redux avec [React](https://reactjs.org), ou avec n'importe quelle autre bibliothèque de vues. Il est minuscule (2 Ko, y compris les dépendances).

## Learn Redux

Nous avons une variété de ressources disponibles pour vous aider à apprendre Redux, peu importe votre formation ou votre style d'apprentissage.

### Just the Basics

Si vous êtes nouveau à Redux et que vous voulez comprendre les concepts de base, voir:

- Le **[basic tutorial in the Redux docs](https://redux.js.org/basics)**
- Redux creator Dan Abramov's **free ["Getting Started with Redux" video series](https://egghead.io/series/getting-started-with-redux)** sur Egghead.io
- Redux co-maintainer Mark Erikson's **["Redux Fundamentals" slideshow](http://blog.isquaredsoftware.com/2018/03/presentation-reactathon-redux-fundamentals/)** et **[list of suggested resources for learning Redux](http://blog.isquaredsoftware.com/2017/12/blogged-answers-learn-redux/)**
- Si vous apprenez mieux en regardant le code et en jouant avec, consultez notre liste de **[Redux example applications](https://redux.js.org/introduction/examples)**, disponible en tant que projets distincts dans le dépôt Redux, et également en tant qu'exemples interactifs en ligne sur CodeSandbox.
- Le **[Redux Tutorials](https://github.com/markerikson/react-redux-links/blob/master/redux-tutorials.md)** section de **[React/Redux links list](https://github.com/markerikson/react-redux-links)**. Voici une liste des tutoriels recommandés:
  - Dave Ceddia's posts [What Does Redux Do? (and when should you use it?)](https://daveceddia.com/what-does-redux-do/) et [How Redux Works: A Counter-Example](https://daveceddia.com/how-does-redux-work/) sont une bonne introduction aux bases de Redux et comment l'utiliser avec React, comme [React and Redux: An Introduction](http://jakesidsmith.com/blog/post/2017-11-18-redux-and-react-an-introduction/).
  - Valentino Gagliardi's post [React Redux Tutorial for Beginners: Learning Redux in 2018](https://www.valentinog.com/blog/react-redux-tutorial-beginners/) est une excellente introduction à de nombreux aspects de l'utilisation de Redux.
  - The CSS Tricks article [Leveling Up with React: Redux](https://css-tricks.com/learning-react-redux/) couvre bien les bases de Redux.
  - This [DevGuides: Introduction to Redux](http://devguides.io/redux/) tutoriel couvre plusieurs aspects de Redux, y compris les actions, les réducteurs, l'utilisation de React et middleware.

### Intermediate Concepts

Une fois que vous avez appris les bases du travail avec les actions, les réducteurs et le store, vous pouvez avoir des questions sur des sujets comme la logique asynchrone et les requêtes AJAX, la connexion d'un framework UI comme React à votre store Redux et la mise en place d'une application pour utiliser Redux:

- Le **["Advanced" docs section](https://redux.js.org/advanced)** couvre comment travailler avec la logique asynchrone, middleware, routage.
- Les documents Redux **["Learning Resources"](https://redux.js.org/introduction/learning-resources)** La page pointe vers des articles recommandés sur divers sujets liés à Redux.
- Sophie DeBenedetto's 8-part **[Building a Simple CRUD App with React + Redux](http://www.thegreatcodeadventure.com/building-a-simple-crud-app-with-react-redux-part-1/)** série montre comment mettre en place une application de base CRUD à partir de zéro.

### Real-World Usage

Passer d'une application TodoMVC à une application de production réelle peut être un grand saut, mais nous avons beaucoup de ressources pour aider:

- Redux creator Dan Abramov's **[free "Building React Applications with Idiomatic Redux" video series](https://egghead.io/courses/building-react-applications-with-idiomatic-redux)** s'appuie sur sa première série de vidéos et couvre des sujets tels que le middleware, le routage et la persistance.
- Le **[Redux FAQ](https://redux.js.org/faq)** répond à de nombreuses questions fréquentes sur l'utilisation de Redux et le **["Recipes" docs section](https://redux.js.org/recipes)** contient de l'information sur la manipulation des données dérivées, les essais, la logique de réduction du structurant et la réduction de la norme.
- Redux co-maintainer Mark Erikson's **["Practical Redux" tutorial series](http://blog.isquaredsoftware.com/series/practical-redux/)** démontre des techniques intermédiaires et avancées du monde réel pour travailler avec React et Redux (également disponible sur **[an interactive course on Educative.io](https://www.educative.io/collection/5687753853370368/5707702298738688)**).  
- The **[React/Redux links list](https://github.com/markerikson/react-redux-links)** a classé les articles sur le travail avec [reducers and selectors](https://github.com/markerikson/react-redux-links/blob/master/redux-reducers-selectors.md), [managing side effects](https://github.com/markerikson/react-redux-links/blob/master/redux-side-effects.md), [Redux architecture and best practices](https://github.com/markerikson/react-redux-links/blob/master/redux-architecture.md), et plus.
- Notre communauté a créé des milliers de bibliothèques, d'addons et d'outils liés à Redux.  L' **["Ecosystem" docs page](https://redux.js.org/introduction/ecosystem)** liste de nos recommandations, et il y a une liste complète disponible dans le **[Redux addons catalog](https://github.com/markerikson/redux-ecosystem-links)**.
- Si vous souhaitez apprendre à partir de bases de code d'application réelles, le catalogue d'addons contient également une liste de **[purpose-built examples and real-world applications](https://github.com/markerikson/redux-ecosystem-links/blob/master/apps-and-examples.md)**.

Enfin, Mark Erikson enseigne une série de **[Redux workshops through Workshop.me](#redux-workshops)**.  Vérifier la [workshop schedule](https://workshop.me/?a=mark) pour les dates et lieux à venir.

### Help and Discussion

Le **[#redux channel](https://discord.gg/0ZcbPKXt5bZ6au5t)** et le **[Reactiflux Discord community](http://www.reactiflux.com)** est notre ressource officielle pour toutes les questions liées à l'apprentissage et à l'utilisation de Redux. Reactiflux est un endroit idéal pour sortir, poser des questions et apprendre - venez nous rejoindre!

## Before Proceeding Further

Redux est un outil précieux pour organiser votre state, mais vous devriez également considérer si cela est approprié à votre situation. N'utilisez pas Redux juste parce que quelqu'un a dit que vous devriez - prenez le temps de comprendre les avantages et les compromis potentiels de son utilisation.

Voici quelques suggestions sur quand il est judicieux d'utiliser Redux:

- Vous avez des quantités raisonnables de données qui changent au fil du temps
- Vous avez besoin d'une source unique de vérité pour votre état
- Vous trouvez que garder tout votre état dans un composant de haut niveau n'est plus suffisant

Oui, ces directives sont subjectives et vagues, mais c'est pour une bonne raison. Le point auquel vous devez intégrer Redux dans votre application est différent pour chaque utilisateur et différent pour chaque application.

>**Pour plus d'idées sur la façon dont Redux est destiné à être utilisé, voir:**
>
>- **[You Might Not Need Redux](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)**
>- **[The Tao of Redux, Part 1 - Implementation and Intent](http://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-1/)**
>- **[The Tao of Redux, Part 2 - Practice and Philosophy](http://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-2/)**
>- **[Redux FAQ](https://redux.js.org/faq)**

## Developer Experience

Dan Abramov (auteur de Redux) a écrit Redux pendant qu'il travaillait sur sa conférence React Europe intitulée [“Hot Reloading with Time Travel”](https://www.youtube.com/watch?v=xsSnOQynTHs). Son but était de créer une bibliothèque de gestion d'état avec une API minimale mais un comportement complètement prévisible. Redux permet d'implémenter la journalisation, le rechargement à chaud, le voyage dans le temps, les applications universelles, l'enregistrement et la relecture, sans aucun engagement de la part du développeur.

## Influences

Redux fait évoluer les idées de [Flux](http://facebook.github.io/flux/), mais évite sa complexité en s'inspirant de [Elm](https://github.com/evancz/elm-architecture-tutorial/). Même si vous n'avez pas utilisé Flux ou Elm, Redux ne prend que quelques minutes pour commencer.

## Installation

Pour installer la version stable:

```bash
npm install --save redux
```

This assumes you are using [npm](https://www.npmjs.com/) as your package manager.  

If you're not, you can [access these files on unpkg](https://unpkg.com/redux/), download them, or point your package manager to them.

Most commonly, people consume Redux as a collection of [CommonJS](http://webpack.github.io/docs/commonjs.html) modules. These modules are what you get when you import `redux` in a [Webpack](https://webpack.js.org/), [Browserify](http://browserify.org/), or a Node environment. If you like to live on the edge and use [Rollup](http://rollupjs.org), we support that as well.

If you don't use a module bundler, it's also fine. The `redux` npm package includes precompiled production and development [UMD](https://github.com/umdjs/umd) builds in the [`dist` folder](https://unpkg.com/redux/dist/). They can be used directly without a bundler and are thus compatible with many popular JavaScript module loaders and environments. For example, you can drop a UMD build as a [`<script>` tag](https://unpkg.com/redux/dist/redux.js) on the page, or [tell Bower to install it](https://github.com/reduxjs/redux/pull/1181#issuecomment-167361975). The UMD builds make Redux available as a `window.Redux` global variable.

The Redux source code is written in ES2015 but we precompile both CommonJS and UMD builds to ES5 so they work in [any modern browser](http://caniuse.com/#feat=es5). You don't need to use Babel or a module bundler to [get started with Redux](https://github.com/reduxjs/redux/blob/master/examples/counter-vanilla/index.html).

### Complementary Packages

Most likely, you'll also need [the React bindings](https://github.com/reduxjs/react-redux) and [the developer tools](https://github.com/reduxjs/redux-devtools).

```
npm install --save react-redux
npm install --save-dev redux-devtools
```

Note that unlike Redux itself, many packages in the Redux ecosystem don't provide UMD builds, so we recommend using CommonJS module bundlers like [Webpack](https://webpack.js.org/) and [Browserify](http://browserify.org/) for the most comfortable development experience.

## The Gist

The whole state of your app is stored in an object tree inside a single *store*.  
The only way to change the state tree is to emit an *action*, an object describing what happened.  
To specify how the actions transform the state tree, you write pure *reducers*.

That's it!

```js
import { createStore } from 'redux'

/**
 * This is a reducer, a pure function with (state, action) => state signature.
 * It describes how an action transforms the state into the next state.
 *
 * The shape of the state is up to you: it can be a primitive, an array, an object,
 * or even an Immutable.js data structure. The only important part is that you should
 * not mutate the state object, but return a new object if the state changes.
 *
 * In this example, we use a `switch` statement and strings, but you can use a helper that
 * follows a different convention (such as function maps) if it makes sense for your
 * project.
 */
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}

// Create a Redux store holding the state of your app.
// Its API is { subscribe, dispatch, getState }.
let store = createStore(counter)

// You can use subscribe() to update the UI in response to state changes.
// Normally you'd use a view binding library (e.g. React Redux) rather than subscribe() directly.
// However it can also be handy to persist the current state in the localStorage.

store.subscribe(() =>
  console.log(store.getState())
)

// The only way to mutate the internal state is to dispatch an action.
// The actions can be serialized, logged or stored and later replayed.
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```

Instead of mutating the state directly, you specify the mutations you want to happen with plain objects called *actions*. Then you write a special function called a *reducer* to decide how every action transforms the entire application's state.

If you're coming from Flux, there is a single important difference you need to understand. Redux doesn't have a Dispatcher or support many stores. Instead, there is just a single store with a single root reducing function. As your app grows, instead of adding stores, you split the root reducer into smaller reducers independently operating on the different parts of the state tree. This is exactly like how there is just one root component in a React app, but it is composed out of many small components.

This architecture might seem like an overkill for a counter app, but the beauty of this pattern is how well it scales to large and complex apps. It also enables very powerful developer tools, because it is possible to trace every mutation to the action that caused it. You can record user sessions and reproduce them just by replaying every action.

## Learn Redux from Its Authors

### Redux Video Tutorials by Dan Abramov

#### Getting Started with Redux
**[Getting Started with Redux](https://egghead.io/series/getting-started-with-redux)** is a video course consisting of 30 videos narrated by [Dan Abramov](https://twitter.com/dan_abramov), author of Redux. It is designed to complement the “Basics” part of the docs while bringing additional insights about immutability, testing, Redux best practices, and using Redux with React. **This course is free and will always be.**

>[“Great course on egghead.io by @dan_abramov - instead of just showing you how to use #redux, it also shows how and why redux was built!”](https://twitter.com/sandrinodm/status/670548531422326785)  
>Sandrino Di Mattia

>[“Plowing through @dan_abramov 'Getting Started with Redux' - its amazing how much simpler concepts get with video.”](https://twitter.com/chrisdhanaraj/status/670328025553219584)  
>Chris Dhanaraj

>[“This video series on Redux by @dan_abramov on @eggheadio is spectacular!”](https://twitter.com/eddiezane/status/670333133242408960)  
>Eddie Zaneski

>[“Come for the name hype. Stay for the rock solid fundamentals. (Thanks, and great job @dan_abramov and @eggheadio!)”](https://twitter.com/danott/status/669909126554607617)  
>Dan

>[“This series of videos on Redux by @dan_abramov is repeatedly blowing my mind - gunna do some serious refactoring”](https://twitter.com/gelatindesign/status/669658358643892224)  
>Laurence Roberts

So, what are you waiting for?

#### [Watch the free "Getting Started with Redux" video series](https://egghead.io/series/getting-started-with-redux)

> Note: If you enjoyed Dan's course, consider supporting Egghead by [buying a subscription](https://egghead.io/pricing). Subscribers have access to the source code of every example in my videos and tons of advanced lessons on other topics, including JavaScript in depth, React, Angular, and more. Many [Egghead instructors](https://egghead.io/instructors) are also open source library authors, so buying a subscription is a nice way to thank them for the work that they've done.


#### Building React Applications with Idiomatic Redux

The **[Building React Applications with Idiomatic Redux](https://egghead.io/courses/building-react-applications-with-idiomatic-redux)** course is a second free video series by Dan Abramov.  It picks up where the first series left off, and covers practical production ready techniques for building your React and Redux applications: advanced state management, middleware, React Router integration, and other common problems you are likely to encounter while building applications for your clients and customers.  As with the first series, **this course will always be free**.


#### [Watch the free "Idiomatic Redux" video series](https://egghead.io/courses/building-react-applications-with-idiomatic-redux)



### Practical Redux course

**[Practical Redux](https://www.educative.io/collection/5687753853370368/5707702298738688/)** is a paid interactive course by Redux co-maintainer [Mark Erikson](https://twitter.com/acemarke).  The course is designed to show how to apply the basic concepts of Redux to building something larger than a TodoMVC application.  It includes real-world topics like:


- Adding Redux to a new Create-React-App project and configuring Hot Module Replacement for faster development 
- Controling your UI behavior with Redux 
- Using the Redux-ORM library to manage relational data in your Redux store 
- Building a master/detail view to display and edit data 
- Writing custom advanced Redux reducer logic to solve specific problems 
- Optimizing performance of Redux-connected form inputs 

And much more!  

The course is based on Mark's original free **["Practical Redux" blog tutorial series](http://blog.isquaredsoftware.com/series/practical-redux/)**, but with updated and improved content.


### Redux Workshops

Redux co-maintainer [Mark Erikson](https://twitter.com/acemarke) has partnered with [Workshop.me](https://workshop.me/) to teach a series of Redux workshops.

The first [**Redux Fundamentals** workshop will be held in New York City, April 19-20](https://workshop.me/2018-04-react-redux?a=mark), and will cover:

- The history and purpose of Redux
- Reducers, actions, and working with a Redux store
- Using Redux with React
- Using and writing Redux middleware
- Working with AJAX calls and other side effects
- Unit testing Redux apps
- Real-world Redux app structure and development

Tickets are still available, and [can be purchased through Workshop.me](https://workshop.me/2018-04-react-redux?a=mark).


## Documentation

* [Introduction](http://redux.js.org/introduction/index.html)
* [Basics](http://redux.js.org/basics/index.html)
* [Advanced](http://redux.js.org/advanced/index.html)
* [Recipes](http://redux.js.org/recipes/index.html)
* [FAQ](http://redux.js.org/FAQ.html)
* [Troubleshooting](http://redux.js.org/Troubleshooting.html)
* [Glossary](http://redux.js.org/Glossary.html)
* [API Reference](http://redux.js.org/api/index.html)

For PDF, ePub, and MOBI exports for offline reading, and instructions on how to create them, please see: [paulkogel/redux-offline-docs](https://github.com/paulkogel/redux-offline-docs).

For Offline docs, please see: [devdocs](http://devdocs.io/redux/)

## Examples

Almost all examples have a corresponding CodeSandbox sandbox. This is an interactive version of the code that you can play with online.

* [**Counter Vanilla**](https://redux.js.org/introduction/examples#counter-vanilla): [Source](https://github.com/reduxjs/redux/tree/master/examples/counter-vanilla)
* [**Counter**](https://redux.js.org/introduction/examples#counter): [Source](https://github.com/reduxjs/redux/tree/master/examples/counter) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/counter)
* [**Todos**](https://redux.js.org/introduction/examples#todos): [Source](https://github.com/reduxjs/redux/tree/master/examples/todos) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/todos)
* [**Todos with Undo**](https://redux.js.org/introduction/examples#todos-with-undo): [Source](https://github.com/reduxjs/redux/tree/master/examples/todos-with-undo) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/todos-with-undo)
* [**TodoMVC**](https://redux.js.org/introduction/examples#todomvc): [Source](https://github.com/reduxjs/redux/tree/master/examples/todomvc) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/todomvc)
* [**Shopping Cart**](https://redux.js.org/introduction/examples#shopping-cart): [Source](https://github.com/reduxjs/redux/tree/master/examples/shopping-cart) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/shopping-cart)
* [**Tree View**](https://redux.js.org/introduction/examples#tree-view): [Source](https://github.com/reduxjs/redux/tree/master/examples/tree-view) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/tree-view)
* [**Async**](https://redux.js.org/introduction/examples#async): [Source](https://github.com/reduxjs/redux/tree/master/examples/async) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/async)
* [**Universal**](https://redux.js.org/introduction/examples#universal): [Source](https://github.com/reduxjs/redux/tree/master/examples/universal)
* [**Real World**](https://redux.js.org/introduction/examples#real-world): [Source](https://github.com/reduxjs/redux/tree/master/examples/real-world) | [Sandbox](https://codesandbox.io/s/github/reduxjs/redux/tree/master/examples/real-world)

If you're new to the NPM ecosystem and have troubles getting a project up and running, or aren't sure where to paste the gist above, check out [simplest-redux-example](https://github.com/jackielii/simplest-redux-example) that uses Redux together with React and Browserify.

## Testimonials

>[“Love what you're doing with Redux”](https://twitter.com/jingc/status/616608251463909376)  
>Jing Chen, creator of Flux

>[“I asked for comments on Redux in FB's internal JS discussion group, and it was universally praised. Really awesome work.”](https://twitter.com/fisherwebdev/status/616286955693682688)  
>Bill Fisher, author of Flux documentation

>[“It's cool that you are inventing a better Flux by not doing Flux at all.”](https://twitter.com/andrestaltz/status/616271392930201604)  
>André Staltz, creator of Cycle

## Thanks

* [The Elm Architecture](https://github.com/evancz/elm-architecture-tutorial) for a great intro to modeling state updates with reducers;
* [Turning the database inside-out](http://www.confluent.io/blog/turning-the-database-inside-out-with-apache-samza/) for blowing my mind;
* [Developing ClojureScript with Figwheel](https://www.youtube.com/watch?v=j-kj2qwJa_E) for convincing me that re-evaluation should “just work”;
* [Webpack](https://webpack.js.org/concepts/hot-module-replacement/) for Hot Module Replacement;
* [Flummox](https://github.com/acdlite/flummox) for teaching me to approach Flux without boilerplate or singletons;
* [disto](https://github.com/threepointone/disto) for a proof of concept of hot reloadable Stores;
* [NuclearJS](https://github.com/optimizely/nuclear-js) for proving this architecture can be performant;
* [Om](https://github.com/omcljs/om) for popularizing the idea of a single state atom;
* [Cycle](https://github.com/cyclejs/cycle-core) for showing how often a function is the best tool;
* [React](https://github.com/facebook/react) for the pragmatic innovation.

Special thanks to [Jamie Paton](http://jdpaton.github.io) for handing over the `redux` NPM package name.

## Logo

You can find the official logo [on GitHub](https://github.com/reduxjs/redux/tree/master/logo).

## Change Log

This project adheres to [Semantic Versioning](http://semver.org/).  
Every release, along with the migration instructions, is documented on the Github [Releases](https://github.com/reduxjs/redux/releases) page.

## Patrons

The work on Redux was [funded by the community](https://www.patreon.com/reactdx).  
Meet some of the outstanding companies that made it possible:

* [Webflow](https://github.com/webflow)
* [Ximedes](https://www.ximedes.com/)

[See the full list of Redux patrons](PATRONS.md), as well as the always-growing list of [people and companies that use Redux](https://github.com/reduxjs/redux/issues/310).

## License

MIT