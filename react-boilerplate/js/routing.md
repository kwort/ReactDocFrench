# Routing via `react-router` and `react-router-redux`

`react-router` est la solution de routage standard pour les applications de react. La chose est qu'avec redux et un seul arbre d'état, l'URL fait partie de cet état. `react-router-redux` prend en charge la synchronisation de l'emplacement de notre application avec l'état de l'application.

(Voir [`react-router-redux` documentation](https://github.com/reactjs/react-router-redux) pour plus d'information)

## Usage

Pour ajouter une nouvelle route, utilisez le générateur avec `npm run generate route`.
Voici à quoi ressemble une route standard (générée) pour un conteneur:

```JS
{
  path: '/',
  name: 'home',
  getComponent(nextState, cb) {
    const importModules = Promise.all([
      import('containers/HomePage')
    ]);

    const renderRoute = loadModule(cb);

    importModules.then(([component]) => {
      renderRoute(component);
    });

    importModules.catch(errorLoading);
  },
}
```

Pour aller à une nouvelle page, utilisez la fonction `push` de` react-router-redux`:

```JS
import { push } from 'react-router-redux';

dispatch(push('/some/page'));
```

## Child Routes

`npm run generate route` ne supporte pas actuellement la génération automatique de routes enfants si vous en avez besoin, mais elles peuvent être facilement créées manuellement.

For example, if you have a route called `about` at `/about` and want to make a child route called `team` at `/about/our-team` you can just add that child page to the parent page's `childRoutes` array like so:

Par exemple, si vous avez une route appelée `about` à `/about` et que vous voulez créer une route enfant nommée `team` à `/about/our-team` vous pouvez simplement ajouter cette page enfant à `childRoutes` tableau comme:

```JS
/* your app's other routes would already be in this array */
{
  path: '/about',
  name: 'about',
  getComponent(nextState, cb) {
    const importModules = Promise.all([
      import('containers/AboutPage'),
    ]);

    const renderRoute = loadModule(cb);

    importModules.then(([component]) => {
      renderRoute(component);
    });

    importModules.catch(errorLoading);
  },
  childRoutes: [
    {
      path: '/about/our-team',
      name: 'team',
      getComponent(nextState, cb) {
        const importModules = Promise.all([
          import('containers/TeamPage'),
        ]);

        const renderRoute = loadModule(cb);

        importModules.then(([component]) => {
          renderRoute(component);
        });

        importModules.catch(errorLoading);
      },
    },
  ]
}
```

## Index routes

Pour ajouter un itinéraire d'index, utilisez le modèle suivant:

```JS
{
  path: '/',
  name: 'home',
  getComponent(nextState, cb) {
    const importModules = Promise.all([
      import('containers/HomePage')
    ]);

    const renderRoute = loadModule(cb);

    importModules.then(([component]) => {
      renderRoute(component);
    });

    importModules.catch(errorLoading);
  },
  indexRoute: {
    getComponent(partialNextState, cb) {
      const importModules = Promise.all([
        import('containers/HomeView')
      ]);

      const renderRoute = loadModule(cb);

      importModules.then(([component]) => {
        renderRoute(component);
      });

      importModules.catch(errorLoading);
    },
  },
}
```

## Dynamic routes

Pour accéder à un itinéraire dynamique tel que 'post/:slug', par exemple 'post/cool-new-post', ajoutez d'abord la route à votre `routes.js`, comme indiqué dans la documentation:

```JS
path: '/posts/:slug',
name: 'post',
getComponent(nextState, cb) {
 const importModules = Promise.all([
   import('containers/Post/reducer'),
   import('containers/Post/sagas'),
   import('containers/Post'),
 ]);

 const renderRoute = loadModule(cb);

 importModules.then(([reducer, sagas, component]) => {
   injectReducer('post', reducer.default);
   injectSagas(sagas.default);
   renderRoute(component);
 });

 importModules.catch(errorLoading);
},
```

###Container:

```JSX
<Link to={`/posts/${post.slug}`} key={post._id}>
```

Lien cliquable avec la charge utile (vous pouvez utiliser appuyer si nécessaire).

###Action:

```JS
export function getPost(slug) {
  return {
    type: LOAD_POST,
    slug,
  };
}

export function postLoaded(post) {
  return {
    type: LOAD_POST_SUCCESS,
    podcast,
  };
}
```

###Saga:

```JS
const { slug } = yield take(LOAD_POST);
yield call(getXhrPodcast, slug);

export function* getXhrPodcast(slug) {
  const requestURL = `http://your.api.com/api/posts/${slug}`;
  const post = yield call(request, requestURL);
  if (!post.err) {
    yield put(postLoaded(post));
  } else {
    yield put(postLoadingError(post.err));
  }
}
```

Attendez (`take`) pour la constante LOAD_POST, qui contient la charge utile slug de la fonction `getPost()` dans actions.js.

Lorsque l'action est déclenchée, envoyez la fonction `getXhrPodcast()` pour obtenir la réponse de votre API. En cas de succès, envoyez l'action `postLoaded()` (`yield put`) qui renvoie la réponse et peut être ajoutée dans l'état du réducteur.


Vous pouvez en savoir plus sur la documentation de [`react-router`] (https://github.com/reactjs/react-router/blob/master/docs/API.md#props-3).