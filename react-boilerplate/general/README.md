# Introduction

L'écosystème JavaScript évolue à une vitesse incroyable: se tenir à jour peut être écrasant. Ainsi, au lieu de devoir vous tenir au courant de tous les nouveaux outils, caractéristiques et techniques pour faire la une des journaux, ce projet vise à alléger la charge en fournissant une base de référence des plus utiles.

En utilisant React Boilerplate, vous commencez votre application avec les idées actuelles de notre communauté sur ce qui représente une expérience de développement optimale, les meilleures pratiques, l'outillage le plus efficace et la structure de projet la plus propre.

- [**CLI Commands**](commands.md)
- [Tool Configuration](files.md)
- [Server Configurations](server-configs.md)
- [Deployment](deployment.md) *(currently Heroku specific)*
- [FAQ](faq.md)
- [Gotchas](gotchas.md)

# Feature overview

## Quick scaffolding

Automatisez la création de composants, de conteneurs, de routes, de sélecteurs et de sagas - et leurs tests - directement à partir de la CLI!

Exécutez `npm run generate` dans votre terminal et choisissez l'une des parties que vous voulez générer. Ils seront automatiquement importés aux bons endroits et tout sera configuré correctement.

> Nous utilisons [plop] pour générer de nouveaux composants, vous pouvez trouver toute la logique et les templates pour la génération dans `internals/generators`.

[plop]: https://github.com/amwmedia/plop

## Instant feedback

Profitez du meilleur DX et codez votre application à la vitesse de la pensée! Vos modifications enregistrées dans CSS et JS sont répercutées instantanément sans actualiser la page. Préserve l'état de l'application même lorsque vous mettez à jour quelque chose dans le code sous-jacent!

## Predictable state management

Nous utilisons Redux pour gérer l'état de nos applications. Nous avons également ajouté un support optionnel pour [Chrome Redux DevTools Extension] - si vous l'avez installé, vous pouvez voir, lire et modifier votre historique d'action!

[Chrome Redux DevTools Extension]: https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd

## Next generation JavaScript

Utilisez les chaînes de template ESNext : object destructuring, arrow functions, JSX syntax, etc., aujourd'hui. C'est possible grâce à Babel avec les préréglages `latest`,` stage-0` et `react`!

## Next generation CSS

Ecrivez des CSS composables qui sont co-localisés avec vos composants en utilisant [`style-composants`] pour une modularité complète. Les noms de classe générés uniques maintiennent la spécificité à un niveau bas tout en éliminant les conflits de style. Expédier uniquement les styles utilisés sur la page visible pour obtenir les meilleures performances.

[`styled-components`]: ../css/styled-components.md

## Industry-standard routing

Il est naturel de vouloir ajouter des pages (par exemple `/about`) à votre application, et le routage rend cela possible. Grâce à [react-router] avec [react-router-redux], c'est simple et l'URL est automatiquement synchronisée avec l'état de votre application!

[react-router]: https://github.com/reactjs/react-router
[react-router-redux]: https://github.com/reactjs/react-router-redux

# Optional extras

_ N'aimes-tu aucune de ces fonctionnalités? [Click here](remove.md)_

## Offline-first

La prochaine frontière dans les applications Web performantes: la disponibilité sans connexion réseau à partir du moment où vos utilisateurs chargent l'application. Ceci est fait avec un ServiceWorker et un retour à AppCache, donc cette fonctionnalité fonctionne même sur les anciens navigateurs!

> Tous vos fichiers sont inclus automatiquement. Aucune intervention manuelle nécessaire grâce à [`offline-plugin`] de Webpack (https://github.com/NekR/offline-plugin)

### Add To Homescreen

Après des visites répétées sur votre site, les utilisateurs recevront une invite pour ajouter votre application à leur écran d'accueil. Combiné avec la mise en cache hors ligne, cela signifie que votre application Web peut être utilisée exactement comme une application native (sans les limitations d'un magasin d'applications).

Le nom et l'icône à afficher sont définis dans le fichier `app/manifest.json`. Changez-les pour le nom et l'icône de votre projet, et essayez-le!

## Performant Web Font Loading

Si vous utilisez simplement des polices Web dans votre projet, la page restera vide jusqu'à ce que ces polices soient téléchargées. Cela signifie beaucoup de temps d'attente dans lequel les utilisateurs peuvent déjà lire le contenu.

[FontFaceObserver](https://github.com/bramstein/fontfaceobserver) ajoute une classe à `body` lorsque les polices ont été chargées. (voir [`app.js`] (../../app/app.js # L26-L36) et [` App / styles.css`] (../../ app / conteneurs / App / styles .css))

### Adding a new font

1. Ajoutez la déclaration `@font-face` à `App/styles.css` ou ajoutez une balise `<link>` à [`index.html`](../../app/index.html). (N'oubliez pas d'enlever le `<link>` pour Open Sans depuis [`index.html`](../../app/index.html)!)

2. Dans `App/styles.css`, spécifiez votre `font-family` initiale dans la balise` body` avec uniquement les polices web-save. Dans la balise `body.jsFontLoaded`, spécifiez votre stack `font-family` avec votre police web.

3. Dans `app.js`, ajoutez un `<fontName>Observer` pour votre police.

## Image optimization

Images often represent the majority of bytes downloaded on a web page, so image optimization can often be a notable performance improvement. Thanks to Webpack's [`image-loader`](https://github.com/tcoopman/image-webpack-loader), every PNG, JPEG, GIF and SVG images is optimized.

Les images représentent souvent la majorité des octets téléchargés sur une page Web, de sorte que l'optimisation de l'image peut souvent être une amélioration notable des performances. Grâce à [`image-loader`](https://github.com/tcoopman/image-webpack-loader) de Webpack, chaque image PNG, JPEG, GIF et SVG est optimisée.

Voir [`image-loader`](https://github.com/tcoopman/image-webpack-loader) pour personnaliser les options d'optimisation.