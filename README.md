# Bachero Utilities

> [!WARNING]
> Le projet Bachero n'est plus maintenu et n'est plus recommandé pour une utilisation en production. La documentation est laissée à titre d'[archive](https://web.archive.org/web/https://bachero.johanstick.fr), et [un article](https://johanstick.fr/fr/bachero/) a été écrit pour expliquer les raisons de cette décision.

Un simple site permettant de créé votre config Bachero en quelques clics. Ce site vous posera des questions sur votre configuration et vous proposera le fichier de configuration selon vos réponse.

![Bachero Utilities](docs/preview.png)

## Installation

Pour installer le site, vous devez avoir [NodeJS](https://nodejs.org/en/) d'installé sur votre machine. Ensuite, vous pouvez cloner le projet et installer les dépendances avec les commandes suivantes:

```bash 
git clone https://github.com/bacherobot/utilities.git
cd utilities
npm install
```

## Développement

Pour tester vos modifications sur le site, vous pouvez utiliser la commande suivante:
```bash
npm run dev

# ou pour ouvrir le site dans votre navigateur lors du dev
npm run dev -- --open
```

## Technologies utilisées

- [Svelte](https://svelte.dev/)
- [Shadcn/UI for Svelte](https://www.shadcn-svelte.com/)
- [TailwindCSS](https://tailwindcss.com/)
- [Vite](https://vitejs.dev/)
- [TypeScript](https://www.typescriptlang.org/)
- [ESLint](https://eslint.org/)

## Licence

Ce projet est sous licence [MIT](LICENSE).
