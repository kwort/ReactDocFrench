# Documentation

## Table of Contents

- [General](general)
  - [**CLI Commands**](general/commands.md)
  - [Introduction ](general/introduction.md)
  - [Tool Configuration](general/files.md)
  - [Server Configurations](general/server-configs.md)
  - [Deployment](general/deployment.md) _(currently Heroku and AWS S3 specific)_
  - [Debugging](general/debugging.md)
  - [FAQ](general/faq.md)
  - [Gotchas](general/gotchas.md)
  - [Remove](general/remove.md)
  - [Extracting components](general/components.md)
- [Testing](testing)
  - [Unit Testing](testing/unit-testing.md)
  - [Component Testing](testing/component-testing.md)
  - [Remote Testing](testing/remote-testing.md)
- [Styling (CSS)](css/README.md)
  - [Next Generation CSS](css/README.md#next-generation-css)
  - [CSS Support](css/README.md#css-we-support)
  - [styled-components](css/README.md#styled-components)
  - [Stylesheet](css/README.md#stylesheet)
  - [CSS Modules](css/README.md#css-modules)
  - [Sass](css/README.md#sass)
  - [LESS](css/README.md#less)
- [JS](js)
  - [Redux](js/redux.md)
  - [ImmutableJS](js/immutablejs.md)
  - [reselect](js/reselect.md)
  - [redux-saga](js/redux-saga.md)
  - [i18n](js/i18n.md)
  - [routing](js/routing.md)
- [Maintenance](maintenance)
  - [Dependency Update](maintenance/dependency.md)

## Overview

### Quickstart

1.  Lancer l'exemple de l'application livré avec ce projet en démo

    ```Shell
    npm run setup && npm start
    ```

1.  Ouvrir [localhost:3000](http://localhost:3000)

- Ajoutez un nom d'utilisateur Github pour voir Redux et Redux Sagas en action
- Modifier le fichier à `./app/components/Header/index.js` pour que le texte de `<Button>` composant lit "Features!!!" [Hot Module Reloading](https://webpack.js.org/guides/hot-module-replacement/)
- Cliquez sur votre bouton Features pour voir React Router en action.

1.  Il est temps de créer votre propre application

    ```shell
    npm run clean
    ```

    ... et utilisez les générateurs intégrés pour démarrer votre première fonction.

### Development

Lancer `npm start` et l'app sera disponible sur `localhost:3000`

### Building & Deploying

1.  Lancer `npm run build`, qui compilera tous les fichiers nécessaires au répertoire `build`.

2.  Uploader le contenu du dossier `build` dans le dossier racine de votre serveur Web.

### Structure

Le répertoire [`app/`](../../../tree/master/app) contient l'ensemble de votre code d'application, y compris CSS,
JavaScript, HTML et tests.

Le reste des dossiers et des fichiers n'existe que pour vous faciliter la vie, et ne devrait pas avoir besoin d'être touché.

_(Si elles doivent être changées, merci de [soumettre une issue](https://github.com/react-boilerplate/react-boilerplate/issues)!)_

### CSS

Utiliser [tagged template literals](https://www.styled-components.com/docs/advanced#tagged-template-literals)
(un ajout récent à JavaScript) et le [power of CSS](https://github.com/styled-components/styled-components/blob/master/docs/css-we-support.md),
`styled-components` vous permet d'écrire du code CSS pour styliser vos composants. Il supprime également le mappage entre les composants et les styles - l'utilisation de composants comme une construction de style bas niveau ne pourrait pas être plus facile!

Voir la [CSS documentation](./css/README.md) pour plus d'informations.

### JS

Nous regroupons tous vos scripts clients et les partageons en plusieurs fichiers en utilisant le partage de code lorsque cela est possible. Nous optimisons automatiquement votre code lors de la construction pour la production, donc vous n'avez pas à vous en préoccuper.

Voir la [JS documentation](./js/README.md) pour plus d'informations sur le côté JavaScript des choses.

### SEO

We use [react-helmet](https://github.com/nfl/react-helmet) for managing document head tags. Examples on how to
write head tags can be found [here](https://github.com/nfl/react-helmet#examples).

Nous utilisons [react-helmet](https://github.com/nfl/react-helmet) pour gérer les balises de tête de document. Exemples sur comment écrire les balises de tête peuvent être trouvées [here](https://github.com/nfl/react-helmet#examples).

### Testing

Pour une explication complète de la procédure de test, voir [testing documentation](./testing/README.md)!

#### Browser testing

`npm run start:tunnel` rend votre application localement disponible globalement sur le web via une URL temporaire: idéal pour tester sur différents appareils, démos client, etc!

#### Unit testing

Les tests unitaires sont dans les répertoires `test` des composants testés et sont exécutés avec `npm run test`.