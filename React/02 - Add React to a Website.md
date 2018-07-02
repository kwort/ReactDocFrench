# Add React to a Website

## Introduction

La majorité des sites Web ne sont pas et ne doivent pas nécessairement être des applications à une seule page.

## Step 1: Add a DOM Container to the HTML

Commencez par ouvrir la page HTML que vous souhaitez modifier. Ajoutez une balise "div" vide pour marquer l'endroit où vous voulez afficher quelque chose avec React.

```javascript
<!-- ... existing HTML ... -->
<div id="like_button_container"></div>
<!-- ... existing HTML ... -->
```

Nous avons donné à cet attribut **div** un identifiant HTML unique. Cela nous permettra de le trouver plus tard à partir du code JavaScript et d'afficher un composant React à l'intérieur de celui-ci.

Pointe

Vous pouvez placer un "container" **div** comme ça n'importe où dans la balise **body**. Vous pouvez avoir autant de conteneurs DOM indépendants sur une page que vous le souhaitez. Ils sont généralement vides - React remplacera tout contenu existant dans à l'interieur des container DOM.

## Step 2: Add the Script Tags

Ensuite, ajoutez trois balises **script** à la page HTML juste avant la balise de fermeture **body**:

```html
  <!-- ... other HTML ... -->

  <!-- Load React. -->
  <!-- Note: when deploying, replace "development.js" with "production.min.js". -->
  <script src="https://unpkg.com/react@16/umd/react.development.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@16/umd/react-dom.development.js" crossorigin></script>

  <!-- Load our React component. -->
  <script src="like_button.js"></script>

  <script>
    const domContainer = document.getElementById('like_button_container');
    ReactDOM.render(React.createElement(LikeButton), domContainer);
  </script>

</body>
```

Les deux premiers tags chargent React. Le troisième chargera votre code de composant.

## Step 3: Create a React Component

Créez un fichier appelé like_button.js à côté de votre page HTML et ajoutez le code :

```javascript
'use strict';

class LikeButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = { liked: false };
  }

  render() {
    if (this.state.liked) {
      return 'You liked this.';
    }

    return React.createElement(
      'button',
      { onClick: () => this.setState({ liked: true }) },
      'Like'
    );
  }
}
```

Généralement, vous pouvez afficher les composants React à plusieurs endroits sur la page HTML. Voici un exemple qui affiche trois fois le bouton like :

index.html :

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Add React in One Minute</title>
  </head>
  <body>

    <h2>Add React in One Minute</h2>
    <p>This page demonstrates using React with no build tooling.</p>
    <p>React is loaded as a script tag.</p>

    <p>
      This is the first comment.
      <!-- We will put our React component inside this div. -->
      <div class="like_button_container" data-commentid="1"></div>
    </p>

    <p>
      This is the second comment.
      <!-- We will put our React component inside this div. -->
      <div class="like_button_container" data-commentid="2"></div>
    </p>

    <p>
      This is the third comment.
      <!-- We will put our React component inside this div. -->
      <div class="like_button_container" data-commentid="3"></div>
    </p>

    <!-- Load React. -->
    <!-- Note: when deploying, replace "development.js" with "production.min.js". -->
    <script src="https://unpkg.com/react@16/umd/react.development.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@16/umd/react-dom.development.js" crossorigin></script>

    <!-- Load our React component. -->
    <script src="like_button.js"></script>

    <script>
      document.querySelectorAll('.like_button_container')
        .forEach(domContainer => {
          // Read the comment ID from a data-* attribute.
          const commentID = parseInt(domContainer.dataset.commentid, 10);
          ReactDOM.render(
            e(LikeButton, { commentID: commentID }),
            domContainer
          );
        });
    </script>

  </body>
</html>
```

like_button.js :

```javascript
'use strict';

class LikeButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = { liked: false };
  }

  render() {
    if (this.state.liked) {
      return 'You liked comment number ' + this.props.commentID;
    }

    return React.createElement(
      'button',
      { onClick: () => this.setState({ liked: true }) },
      'Like'
    );
  }
}

// Find all DOM containers, and render Like buttons into them.
document.querySelectorAll('.like_button_container')
  .forEach(domContainer => {
    // Read the comment ID from a data-* attribute.
    const commentID = parseInt(domContainer.dataset.commentid, 10);
    ReactDOM.render(
      React.createElement(LikeButton, { commentID: commentID }),
      domContainer
    );
  });
```

## React with JSX

Dans les exemples ci-dessus, nous ne nous sommes appuyés que sur des fonctionnalités supportées nativement par les navigateurs. C'est pourquoi nous avons utilisé un appel de fonction JavaScript pour dire à React ce qu'il faut afficher:

```javascript
return React.createElement(
  'button',
  { onClick: () => this.setState({ liked: true }) },
  'Like'
);
```

Cependant, React offre également une option pour utiliser JSX à la place:

```javascript
return (
  <button onClick={() => this.setState({ liked: true })}>
    Like
  </button>
);
```

Ces deux extraits de code sont équivalents. Bien que JSX soit complètement optionnel, beaucoup de gens trouvent cela utile pour écrire du code d'interface utilisateur - à la fois avec React et avec d'autres bibliothèques.

## Quickly Try JSX

Le moyen le plus rapide d'essayer JSX dans votre projet est d'ajouter cette balise **script** à votre page:

```javascript
<script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
```

Vous pouvez maintenant utiliser JSX dans n'importe quelle balise **script** en ajoutant l'attribut type="text/babel". Voici un exemple de fichier HTML avec JSX :

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Hello World</title>
    <script src="https://unpkg.com/react@16/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"></script>

    <!-- Don't use this in production: -->
    <script src="https://unpkg.com/babel-standalone@6.15.0/babel.min.js"></script>
  </head>
  <body>
    <div id="root"></div>
    <script type="text/babel">
      ReactDOM.render(
        <h1>Hello, world!</h1>,
        document.getElementById('root')
      );
    </script>
    <!--
      Note: cette page est un excellent moyen d'essayer React mais ce n'est pas adapté à la production.
      Il compile lentement JSX avec Babel dans le navigateur et utilise une grande version de développement de React.

      Lisez cette section pour une configuration prête pour la production avec JSX:
      http://reactjs.org/docs/add-react-to-a-website.html#add-jsx-to-a-project

      Dans un projet plus important, vous pouvez utiliser une chaîne d'outils intégrée qui inclut JSX à la place:
      http://reactjs.org/docs/create-a-new-react-app.html

      Vous pouvez également utiliser React sans JSX, auquel cas vous pouvez supprimer Babel:
      https://reactjs.org/docs/react-without-jsx.html
    -->
  </body>
</html>
```

Cette approche est parfaite pour apprendre et créer des démos simples. Cependant, cela rend votre site Web lent et ne convient pas à la production. Lorsque vous êtes prêt à aller de l'avant, supprimez cette nouvelle balise **script** et les attributs type="text/babel" que vous avez ajoutés. Au lieu de cela, dans la section suivante, vous allez configurer un préprocesseur JSX pour convertir automatiquement toutes vos balises **script**.

## Add JSX to a Project

L'ajout de JSX à un projet ne nécessite pas d'outils compliqués comme un bundler ou un serveur de développement. Essentiellement, l'ajout de JSX ressemble beaucoup à l'ajout d'un préprocesseur CSS. La seule exigence est d'avoir Node.js installé sur votre ordinateur.

Ouvrir un terminal :

```bash
npm init -y
npm install babel-cli@6 babel-preset-react-app@3
```

## Run JSX Preprocessor

Créez un dossier appelé src et exécutez cette commande de terminal:

```bash
npx babel --watch src --out-dir . --presets react-app/prod
```

npx n'est pas une faute de frappe - c'est un outil de runner de paquet qui vient avec npm 5.2+.

N'attendez pas que cela se termine - cette commande démarre un observateur automatisé pour JSX. Si vous créez maintenant un fichier appelé src/like_button.js avec ce code de démarrage JSX, l'observateur créera un like_button.js prétraité avec le code JavaScript normal adapté au navigateur. Lorsque vous modifiez le fichier source avec JSX, la transformation est automatiquement réexécutée.

En prime, cela vous permet également d'utiliser des fonctionnalités de syntaxe JavaScript modernes comme les classes sans vous soucier de casser les anciens navigateurs. L'outil que nous venons d'utiliser s'appelle Babel, et vous pouvez en apprendre plus à ce sujet dans sa documentation.

Si vous remarquez que vous vous sentez à l'aise avec les outils de construction et que vous voulez qu'ils en fassent plus pour vous, la section suivante décrit certaines des chaînes d'outils les plus populaires et les plus accessibles. Si ce n'est pas le cas, ces balises de script seront parfaites!