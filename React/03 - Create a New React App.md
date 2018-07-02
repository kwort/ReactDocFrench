# Create a New React App

## Create React App

Create React App est un environnement confortable pour apprendre à réagir, et c'est la meilleure façon de commencer à construire une nouvelle application d'une seule page dans React. Il configure votre environnement de développement afin que vous puissiez utiliser les dernières fonctionnalités JavaScript, fournir une expérience de développement agréable et optimiser votre application pour la production. Vous devez avoir Node >= 6 et npm >= 5.2 sur votre machine. Pour créer un projet, exécutez:

```javacript
npx create-react-app my-app
cd my-app
npm start
```

Create React App crée juste un pipeline de build frontend. Sous le capot, il utilise Babel et webpack, mais vous n'avez pas besoin de savoir quoi que ce soit à leur sujet. Lorsque vous êtes prêt à déployer en production, l'exécution de npm run build crée une version optimisée de votre application dans le dossier de génération. Vous pouvez en apprendre plus sur Create React App à partir de son README et le Guide de l'utilisateur.

## Next.js

Next.js est un framework populaire et léger pour les applications **server-side-rendered** avec React. Il inclut des solutions de style et de routage prêtes à l'emploi, et suppose que vous utilisez Node.js comme environnement de serveur.

## Gatsby

Gatsby est le meilleur moyen de créer des **static content-oriented website** avec React. Il vous permet d'utiliser des composants React, mais génère des rendus HTML et CSS pré-rendus pour garantir le temps de chargement le plus rapide.

## Creating a Toolchain from Scratch

Une chaîne de compilation JavaScript comprend généralement:

- Un gestionnaire de paquets, tel que Yarn ou npm
- Un bundler, tel que webpack ou Parcel
- Un compilateur tel que Babel. Il vous permet d'écrire du code JavaScript moderne qui fonctionne toujours dans les anciens navigateurs.

Si vous préférez créer votre propre chaîne d'outils JavaScript, consultez ce guide qui recrée certaines fonctionnalités de l'application Create React.
N'oubliez pas de vous assurer que votre chaîne d'outils personnalisée est correctement configurée pour la production.