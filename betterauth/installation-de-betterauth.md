---
description: 'Prêt à ajouter BetterAuth à votre projet ? 🚀 Voici les étapes à suivre :'
---

# ⚙️ Installation de BetterAuth

### 📦 1. Installer le package

Ajoutez BetterAuth dans votre projet avec votre gestionnaire de packages préféré :

```bash
# npm
npm install better-auth

# pnpm
pnpm add better-auth

# yarn
yarn add better-auth

# bun
bun add better-auth
```

👉 **Astuce** : si vous avez une configuration séparée **client / serveur**, installez **BetterAuth dans les deux parties** de votre projet.

***

### 🔑 2. Définir les variables d’environnement

Créez un fichier **`.env`** à la racine de votre projet et ajoutez-y :

#### Secret Key

Une valeur aléatoire utilisée par la librairie pour **le chiffrement et la génération de hash**.\
Vous pouvez la générer avec `openssl` ou via le bouton “Generate Secret” (si disponible).

```env
BETTER_AUTH_SECRET=maSuperCléUltraSecrète123
```

#### Base URL

Définissez l’URL de base de votre application :

```env
BETTER_AUTH_URL=http://localhost:3000 # Base URL de votre app
```

***

### 🛠️ 3. Créer une instance BetterAuth

Créez un fichier **`auth.ts`** dans l’un de ces dossiers :

* 📂 racine du projet
* 📂 `lib/`
* 📂 `utils/`
* ou sous `src/`, `app/`, `server/` (ex: `src/lib/auth.ts`)

👉 Exemple minimal :

```ts
// auth.ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  // ... vos options
});
```

⚠️ Important : exportez l’instance sous le nom **`auth`** ou comme **export par défaut**.

***

### 🗄️ 4. Configurer la base de données

BetterAuth stocke les utilisateurs dans une **base de données**.\
Il supporte nativement : **SQLite**, **PostgreSQL**, **MySQL**, et plus encore ✅

#### Exemple avec SQLite :

```ts
import { betterAuth } from "better-auth";
import Database from "better-sqlite3";

export const auth = betterAuth({
  database: new Database("./sqlite.db"),
});
```

#### Exemple avec un ORM (Drizzle, Prisma, MongoDB, etc.) :

```ts
import { betterAuth } from "better-auth";
import { drizzleAdapter } from "better-auth/adapters/drizzle";
import { db } from "@/db"; // instance drizzle

export const auth = betterAuth({
  database: drizzleAdapter(db, {
    provider: "pg", // ou "mysql", "sqlite"
  }),
});
```

👉 Si votre base n’est pas listée, consultez la doc des **adapters**.

***

### 🗃️ 5. Créer les tables de la base

BetterAuth fournit un **CLI** pour générer ou appliquer les schémas requis.

#### Générer un schéma / migration :

```bash
npx @better-auth/cli generate
```

👉 Utile si vous voulez appliquer la migration manuellement.

#### Appliquer directement la migration (Kysely uniquement) :

```bash
npx @better-auth/cli migrate
```

📌 Consultez la section **CLI** pour plus d’infos.\
Si vous préférez créer le schéma **manuellement**, vous trouverez la structure de base dans la section **Database**.

***

### 🔐 6. Configurer les méthodes d’authentification

BetterAuth supporte **email + mot de passe** et plusieurs **fournisseurs sociaux** par défaut.

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  emailAndPassword: { enabled: true }, 
  socialProviders: { 
    github: { 
      clientId: process.env.GITHUB_CLIENT_ID as string, 
      clientSecret: process.env.GITHUB_CLIENT_SECRET as string, 
    }, 
  }, 
});
```

👉 Vous pouvez aussi activer d’autres méthodes via **plugins** :

* 🔑 **Passkey**
* 📩 **Magic Link**
* 👤 **Username login**
* … et bien plus !

***

### 🌍 7. Monter le handler d’API

Pour gérer les requêtes API, créez une route qui capture :

```
/api/auth/*
```

👉 BetterAuth fonctionne avec **tous les frameworks backend** disposant d’objets `Request` / `Response`.\
Il propose aussi des helpers pour les plus populaires :

* Next.js
* Nuxt
* SvelteKit
* Remix
* SolidStart
* Hono
* Express
* Elysia
* TanStack Start
* Expo

***

### 💻 8. Créer une instance client

Côté client, importez **`createAuthClient`** depuis le package de votre framework (ex: `"better-auth/react"` pour React).

```ts
// lib/auth-client.ts
import { createAuthClient } from "better-auth/react"

export const authClient = createAuthClient({
  baseURL: "http://localhost:3000" // optionnel si même domaine
});
```

👉 Vous pouvez aussi **exporter directement les méthodes** :

```ts
export const { signIn, signUp, useSession } = createAuthClient();
```

***

## 🎉 C’est tout !

Votre application est maintenant prête à utiliser **BetterAuth** 🚀.\
➡️ Passez à la section **Basic Usage** pour apprendre à connecter vos utilisateurs avec `signIn`, `signUp`, etc.
