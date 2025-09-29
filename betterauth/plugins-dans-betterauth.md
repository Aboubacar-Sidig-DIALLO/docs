# 🔌 Plugins dans BetterAuth

Les **plugins** sont une **brique fondamentale** de BetterAuth.\
👉 Ils permettent d’**étendre les fonctionnalités de base** sans réécrire le cœur de la lib.

***

### ✨ À quoi servent les plugins ?

* ➕ Ajouter de **nouvelles méthodes d’authentification** (ex: magic link, passkey, OTP).
* 🔒 Ajouter des **couches de sécurité** (ex: authentification à deux facteurs).
* ⚙️ Introduire des **fonctionnalités avancées** (multi-tenant, audit, rôles & permissions).
* 🎨 **Personnaliser les comportements** (adaptation du login flow, hooks globaux, cookies custom…).

***

### 🛠️ Plugins inclus (built-in)

BetterAuth propose déjà plusieurs plugins prêts à l’emploi.\
Exemples :

* **Two Factor (2FA)** 🔑 → sécurité renforcée.
* **Magic Link** ✉️ → connexion sans mot de passe.
* **Passkey / WebAuthn** 🖐 → authentification biométrique ou clé physique.
* **Organisation & Access Control** 👥 → gestion des organisations et permissions.
* **Social Providers** 🌍 → Google, GitHub, Apple, etc.

👉 Tu peux les activer en quelques lignes seulement.

***

### 🧑‍💻 Créer ses propres plugins

Si les plugins intégrés ne suffisent pas, tu peux développer les tiens :

* Ajouter des endpoints custom.
* Introduire de nouveaux hooks.
* Étendre le `authClient` côté frontend.
* Connecter BetterAuth à d’autres services (analytics, billing, monitoring…).

***

### 🎯 En résumé

Les **plugins** = la **colonne vertébrale de l’extensibilité** de BetterAuth.

* ⚡ Tu profites des plugins intégrés pour les cas les plus courants.
* 🛠️ Tu peux créer tes propres plugins si tu as des besoins spécifiques.

👉 Résultat : une **authentification 100% adaptée** à ton application, sans surcharger ton code.

## 🔌 Utiliser un Plugin dans BetterAuth

Les plugins peuvent être :

* 🖥️ **Serveur uniquement** (logique côté backend : ex. gestion des sessions, DB, sécurité).
* 🌐 **Client uniquement** (logique côté frontend : ex. hooks supplémentaires, gestion UI).
* 🔄 **Serveur + Client** (la majorité des plugins : ex. 2FA, magic link, social login).

***

### 🖥️ Côté serveur : activer un plugin

Pour ajouter un plugin côté **backend**, il suffit de l’inclure dans l’option `plugins` de ta configuration `betterAuth`.

```ts
// server.ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  plugins: [
    // Exemple : plugin d’auth à deux facteurs
    // twoFactor()
  ],
});
```

➡️ Ici, chaque plugin peut être initialisé avec ses propres **options de configuration** (selon la doc du plugin).

***

### 🌐 Côté client : activer un plugin

Les plugins qui ont une partie **frontend** doivent être ajoutés lors de la création de ton `authClient`.

```ts
// auth-client.ts
import { createAuthClient } from "better-auth/client";

export const authClient = createAuthClient({
  plugins: [
    // Exemple : plugin 2FA côté client
    // twoFactorClient({ twoFactorPage: "/two-factor" })
  ],
});
```

👉 Cela permet :

* d’exposer de **nouvelles méthodes** côté client (`authClient.magicLink.signIn()`, `authClient.twoFactor.verifyTOTP()`, etc.).
* d’ajouter des **hooks UI réactifs** liés au plugin.

***

### 📌 Bonne pratique : séparer les fichiers

Il est recommandé de **séparer ton instance serveur et ton instance client** :

* `auth.ts` ou `server.ts` → configuration BetterAuth côté **backend**.
* `auth-client.ts` → configuration BetterAuth côté **frontend**.

Cela évite les confusions et permet une meilleure maintenabilité.

***

### 🎯 En résumé

* Les **plugins serveur** se déclarent dans `betterAuth({ plugins: [...] })`.
* Les **plugins client** se déclarent dans `createAuthClient({ plugins: [...] })`.
* La plupart des plugins nécessitent une **double déclaration** (backend + frontend).
* Sépare bien ton `auth.ts` (serveur) et ton `auth-client.ts` (client).

## 🛠️ Créer un Plugin BetterAuth

👉 Les **plugins** sont ce qui rend BetterAuth vraiment **extensible**.\
Ils permettent d’ajouter de nouvelles fonctionnalités sans modifier le noyau.

***

### 🖥️ Étape 1 : Créer un **plugin serveur**

Les **plugins serveur** sont la **colonne vertébrale** des plugins.\
Ils :

* définissent les **endpoints** (API personnalisées),
* étendent le **schéma de base de données**,
* ajoutent des **middlewares** ou **hooks**,
* appliquent des règles globales (rate limiting, sécurité, cookies…).

💡 Les **plugins client** sont juste une **couche d’interface** → ils se connectent à ton plugin serveur pour simplifier l’usage côté frontend.

***

### 📌 Que peut faire un plugin ?

Un plugin peut :

1. **Créer des endpoints custom**\
   👉 Tu peux définir des routes supplémentaires pour exécuter n’importe quelle logique métier.
2. **Étendre les tables de la base**\
   👉 Ajoute tes propres colonnes ou tables pour stocker des infos spécifiques à ton app.
3. **Utiliser un middleware avec route matcher**\
   👉 Cible un **groupe de routes** (par motif `/sign-in/*`) et exécute du code seulement quand elles sont appelées.
4. **Ajouter des hooks ciblés**\
   👉 Exécute du code avant/après un endpoint spécifique, ou même si le endpoint est invoqué directement.
5. **Intercepter toutes les requêtes/réponses**\
   👉 Avec `onRequest` ou `onResponse`, tu peux injecter une logique qui touche **toutes les requêtes** (ex: logs, monitoring, audit).
6. **Créer des règles de rate-limit personnalisées**\
   👉 Exemple : limiter le nombre de tentatives de connexion par IP ou par utilisateur.

***

### 🚀 Étape 2 : Ajouter un **plugin client** (si nécessaire)

Si ton plugin serveur expose des **endpoints** à appeler depuis le frontend → tu devras aussi créer un **plugin client**.

💡 Le plugin client permet d’appeler simplement tes endpoints avec :

```ts
authClient.myPlugin.doSomething()
```

Sans lui, tu devrais appeler l’API manuellement avec `fetch()`.

***

### 🎯 En résumé

Créer un plugin BetterAuth te permet de :

* Étendre la **base de données** 🗄️
* Créer des **endpoints custom** 🌍
* Ajouter des **middlewares** 🛡️
* Définir des **hooks ciblés** 🎯
* Intercepter toutes les requêtes (`onRequest` / `onResponse`) ⚡
* Appliquer du **rate limiting sur mesure** ⏳

👉 Et si besoin, tu ajoutes un **plugin client** pour exposer facilement ces nouvelles fonctionnalités côté frontend.

## ⚙️ Créer un Plugin Serveur dans BetterAuth

Pour créer un **plugin serveur**, il suffit de retourner un objet qui respecte l’interface **`BetterAuthPlugin`**.

👉 **Propriété obligatoire :**

* `id` → un identifiant **unique** du plugin (chaîne de caractères).
  * Il sert à distinguer ton plugin des autres (ex: `"two-factor"`, `"audit-logger"`, etc.).
  * 💡 Si tu crées aussi un plugin client, il doit **partager le même `id`** pour rester synchronisé.

***

### 📌 Exemple minimal de plugin serveur

```ts
// plugin.ts
import type { BetterAuthPlugin } from "better-auth";

export const myPlugin = () => {
  return {
    id: "my-plugin", // identifiant unique du plugin
  } satisfies BetterAuthPlugin;
};
```

➡️ Ici, c’est le **squelette minimal** d’un plugin.

***

### 🤔 Fonction ou objet ?

* Tu pourrais directement écrire :

```ts
export const myPlugin: BetterAuthPlugin = {
  id: "my-plugin",
};
```

* **Mais** il est recommandé d’utiliser une **fonction** :

```ts
export const myPlugin = (options?: { debug?: boolean }) => {
  return {
    id: "my-plugin",
    // … autres propriétés ici
  } satisfies BetterAuthPlugin;
};
```

👉 Pourquoi ?

* Tu peux **passer des options** au plugin (`debug`, `config`, etc.).
* C’est **plus cohérent** avec la manière dont sont construits les **plugins officiels BetterAuth**.

***

### 🎯 En résumé

* Un plugin serveur **doit avoir un `id` unique**.
* Il peut être écrit en **objet simple**, mais la bonne pratique = **fonction qui retourne un objet**.
* Cela permet d’**ajouter des options configurables** et d’être aligné avec la structure des **plugins intégrés**.

## 🌍 Créer des Endpoints dans un Plugin BetterAuth

Les **endpoints** permettent à ton plugin serveur d’exposer de **nouvelles routes API** accessibles par ton frontend ou par d’autres services.

👉 Dans BetterAuth, les endpoints reposent sur **Better Call**, un micro framework TypeScript conçu par l’équipe de BetterAuth.\
Et pour créer des endpoints auth-ready, on utilise **`createAuthEndpoint`**.

***

### 📌 Structure d’un plugin avec endpoint

```ts
// plugin.ts
import { createAuthEndpoint } from "better-auth/api";
import type { BetterAuthPlugin } from "better-auth";

export const myPlugin = () => {
  return {
    id: "my-plugin",
    endpoints: {
      // 🔑 clé = nom interne de l’endpoint
      getHelloWorld: createAuthEndpoint(
        "/my-plugin/hello-world", // 📌 chemin URL exposé
        {
          method: "GET", // Méthode HTTP
        },
        async (ctx) => {
          // 💡 handler de la route
          return ctx.json({
            message: "Hello World",
          });
        }
      ),
    },
  } satisfies BetterAuthPlugin;
};
```

***

### 🔍 Explications détaillées

* **`id: "my-plugin"`** → identifiant unique du plugin.
* **`endpoints`** → objet regroupant tes routes custom.
* **`createAuthEndpoint(path, options, handler)`** → crée un endpoint sécurisé BetterAuth.
  * `path` → URL publique (doit être unique dans ton app).
  * `options` → config (ex: méthode `GET`/`POST`, validation, etc.).
  * `handler(ctx)` → fonction exécutée quand l’endpoint est appelé.

***

### 🧰 Le `ctx` dans un endpoint

Dans `handler(ctx)`, tu accèdes à tout ce que fournit BetterAuth :

* `ctx.json(data)` → renvoyer une réponse JSON.
* `ctx.redirect(url)` → rediriger.
* `ctx.body` → corps de la requête (POST).
* `ctx.query` → query params.
* `ctx.headers` → headers HTTP.
* `ctx.context` → contexte auth BetterAuth :
  * `db` → ton instance de DB / ORM.
  * `options` → la config BetterAuth.
  * `baseURL` → URL de base de l’auth.
  * `generateId()` → générer des IDs uniques.

***

### ⚡ Exemple avancé : récupérer les sessions actives

```ts
import { createAuthEndpoint } from "better-auth/api";
import type { BetterAuthPlugin } from "better-auth";

export const sessionsPlugin = () => {
  return {
    id: "sessions-plugin",
    endpoints: {
      listSessions: createAuthEndpoint(
        "/sessions/list",
        { method: "GET" },
        async (ctx) => {
          const sessions = await ctx.context.adapter.findMany("session", {
            where: {},
          });

          return ctx.json({
            count: sessions.length,
            sessions,
          });
        }
      ),
    },
  } satisfies BetterAuthPlugin;
};
```

***

### 🎯 En résumé

* Les **endpoints** permettent de créer des **APIs custom** dans BetterAuth.
* Utilise `createAuthEndpoint()` pour déclarer une nouvelle route.
* Dans ton `ctx`, tu peux accéder à la **DB, la config, les sessions, les cookies…**.
* Les plugins deviennent ainsi de **mini-modules complets** avec leur propre logique serveur.

## 🧩 Context Object dans un Endpoint BetterAuth

Quand tu crées un endpoint avec `createAuthEndpoint`, tu reçois un objet **`ctx`**.\
À l’intérieur, tu as **`ctx.context`** → un objet riche qui te donne accès à l’écosystème BetterAuth.

#### 🎛️ Propriétés principales de `ctx.context`

| Propriété              | Description                                                            | Exemple d’utilisation                                     |
| ---------------------- | ---------------------------------------------------------------------- | --------------------------------------------------------- |
| **`appName`**          | Nom de l’app (par défaut `"Better Auth"`)                              | `console.log(ctx.context.appName)`                        |
| **`options`**          | Options passées à l’instance BetterAuth (`betterAuth({ ... })`)        | Lire si une feature est activée                           |
| **`tables`**           | Définitions des tables core (users, sessions…)                         | `ctx.context.tables.user`                                 |
| **`baseURL`**          | URL de base de ton auth server (`http://localhost:3000/api/auth`)      | Générer des URLs custom                                   |
| **`session`**          | Config session (ex: `updateAge`, `expiresIn`)                          | `ctx.context.session.expiresIn`                           |
| **`secret`**           | Clé secrète de ton instance (pour cookies, hash…)                      | Hasher un cookie custom                                   |
| **`authCookie`**       | Config cookies par défaut                                              | `ctx.context.authCookie.sessionToken`                     |
| **`logger`**           | Logger interne BetterAuth                                              | `ctx.context.logger.info("Nouvelle session")`             |
| **`db`**               | Instance Kysely brute (requêtes SQL)                                   | `await ctx.context.db.selectFrom("user").execute()`       |
| **`adapter`**          | API ORM-like (préférée à `db`)                                         | `await ctx.context.adapter.findOne("user", { id })`       |
| **`internalAdapter`**  | Méthodes internes BetterAuth (sessions, users…)                        | `await ctx.context.internalAdapter.createSession(userId)` |
| **`createAuthCookie`** | Helper pour noms + options de cookies sécurisés (`__Secure`, `__Host`) | Générer cookie conforme automatiquement                   |

👉 Résumé :

* **Utilise `adapter`** dans 90% des cas (ORM-like, safe).
* **Utilise `internalAdapter`** si tu veux reproduire une logique auth officielle (ex: `createSession`, `updateSession`).
* **Utilise `db`** uniquement pour des requêtes SQL très custom.

***

## 📜 Règles Officielles pour Créer un Endpoint

✅ **Convention de nommage :**

* Utilise **kebab-case** pour tes paths (ex: `/my-plugin/hello-world`, pas `/helloWorld`).

✅ **Méthodes HTTP :**

* `GET` → lecture de données.
* `POST` → modifications (création, mise à jour, suppression).

✅ **Isolation des chemins :**

* Préfixe toujours ton path avec le **nom du plugin** → évite les conflits.
  * Mauvais : `/hello-world`
  * Bon : `/my-plugin/hello-world`

✅ **Utilisation obligatoire de `createAuthEndpoint` :**

* Ne fais pas de `fetch` custom → tout doit passer par `createAuthEndpoint`.

***

## 🚀 Exemple complet avec règles appliquées

```ts
// plugin.ts
import { createAuthEndpoint } from "better-auth/api";
import type { BetterAuthPlugin } from "better-auth";

export const myPlugin = () => {
  return {
    id: "my-plugin",
    endpoints: {
      // Lecture des profils → GET
      getProfiles: createAuthEndpoint(
        "/my-plugin/profiles",
        { method: "GET" },
        async (ctx) => {
          const users = await ctx.context.adapter.findMany("user", {
            where: {},
          });
          return ctx.json({ count: users.length, users });
        }
      ),

      // Création d’un profil → POST
      createProfile: createAuthEndpoint(
        "/my-plugin/profiles",
        { method: "POST" },
        async (ctx) => {
          const { email, name } = ctx.body;
          const user = await ctx.context.adapter.create("user", {
            data: { email, name },
          });
          return ctx.json({ message: "User created", user });
        }
      ),
    },
  } satisfies BetterAuthPlugin;
};
```

***

## 🎯 En résumé

* `ctx.context` = **boîte à outils complète** de BetterAuth.
* Respecte les **conventions d’endpoint** (kebab-case, GET pour lecture, POST pour modifications).
* Utilise toujours **`createAuthEndpoint`**.
* Préfixe ton path avec le **nom du plugin** pour éviter les collisions.

## 🗄️ Définir un **Schema** dans un Plugin BetterAuth

Un plugin peut inclure un objet `schema` → c’est ce qui décrit **les tables et colonnes** que ton plugin doit ajouter à la base.

#### 📌 Exemple simple : créer une nouvelle table

```ts
// plugin.ts
import type { BetterAuthPlugin } from "better-auth/plugins";

export const myPlugin = () => {
  return {
    id: "my-plugin",
    schema: {
      myTable: {
        fields: {
          name: {
            type: "string", // type = string | number | boolean | date
            required: true, // valeur obligatoire à l’insertion
            unique: true,   // doit être unique en DB
          },
        },
        modelName: "myTable", // optionnel (sinon prend la clé comme nom)
      },
    },
  } satisfies BetterAuthPlugin;
};
```

***

## 🔑 Champs et Propriétés disponibles

Chaque champ (`field`) a une définition sous forme d’objet :

| Propriété       | Description                                                | Exemple                                                             |
| --------------- | ---------------------------------------------------------- | ------------------------------------------------------------------- |
| **`type`**      | Type de la colonne (`string`, `number`, `boolean`, `date`) | `{ type: "string" }`                                                |
| **`required`**  | Champ obligatoire (par défaut `false`)                     | `{ type: "string", required: true }`                                |
| **`unique`**    | Valeur unique en DB (par défaut `false`)                   | `{ type: "string", unique: true }`                                  |
| **`reference`** | Relation avec une autre table                              | `ts reference: { model: "user", field: "id", onDelete: "CASCADE" }` |

***

## 🔗 Exemple avec Référence

```ts
const myPlugin = () => {
  return {
    id: "my-plugin",
    schema: {
      posts: {
        fields: {
          title: { type: "string", required: true },
          userId: {
            type: "string",
            reference: {
              model: "user", // relation avec la table user
              field: "id",
              onDelete: "CASCADE", // si l’utilisateur est supprimé → supprime ses posts
            },
          },
        },
      },
    },
  } satisfies BetterAuthPlugin;
};
```

***

## ⚙️ Autres Propriétés du Schema

| Propriété              | Description                                                          | Exemple                                                  |
| ---------------------- | -------------------------------------------------------------------- | -------------------------------------------------------- |
| **`disableMigration`** | Empêche la migration automatique de cette table (par défaut `false`) | `disableMigration: opts.storage.provider !== "database"` |
| **`modelName`**        | Alias du nom de la table en DB                                       | `"myTable"`                                              |

***

## 🧑‍💻 Étendre la Table `user` ou `session`

Tu peux ajouter des champs **directement à la table utilisateur** sans recréer une nouvelle table.\
BetterAuth **mettra à jour les types automatiquement** → super pratique avec TypeScript.

```ts
const myPlugin = () => {
  return {
    id: "my-plugin",
    schema: {
      user: {
        fields: {
          age: { type: "number" },
          isPremium: { type: "boolean", required: true },
        },
      },
    },
  } satisfies BetterAuthPlugin;
};
```

👉 Ensuite, quand tu appelles :

```ts
const session = await auth.api.getSession({ headers });
console.log(session.user.age);       // number
console.log(session.user.isPremium); // boolean
```

***

## ⚠️ Bonnes pratiques

* ❌ Ne stocke **pas d’infos sensibles** (tokens, secrets, clés privées) dans `user` ou `session`.\
  → Crée une nouvelle table dédiée (ex: `user_secrets`).
* ✅ Utilise `reference` pour gérer les relations proprement.
* ✅ Ajoute `disableMigration` si tu veux gérer la migration toi-même (ex: avec Prisma/Drizzle).
* ✅ Laisse BetterAuth générer les types pour toi → tu auras l’autocomplétion TypeScript.

## 🪝 Hooks dans un Plugin BetterAuth

Un plugin peut embarquer des **hooks serveur** via la clé `hooks`.\
Ces hooks permettent d’exécuter du code **avant** (`before`) ou **après** (`after`) une requête.

### 📌 Structure d’un plugin avec hooks

```ts
// plugin.ts
import { createAuthMiddleware } from "better-auth/plugins";

const myPlugin = () => {
  return {
    id: "my-plugin",
    hooks: {
      before: [
        {
          matcher: (context) => {
            // condition d’activation du hook
            return context.headers.get("x-my-header") === "my-value";
          },
          handler: createAuthMiddleware(async (ctx) => {
            // ✅ Code exécuté avant l’action
            console.log("Avant la requête :", ctx.path);

            // Tu peux modifier le contexte si besoin
            return {
              context: ctx,
            };
          }),
        },
      ],

      after: [
        {
          matcher: (context) => {
            // ce hook ne s’applique que sur la route /sign-up/email
            return context.path === "/sign-up/email";
          },
          handler: createAuthMiddleware(async (ctx) => {
            // ✅ Code exécuté après l’action
            return ctx.json({
              message: "Hello World 👋",
            });
          }),
        },
      ],
    },
  } satisfies BetterAuthPlugin;
};
```

***

### 🧩 Les deux parties essentielles

#### 🔹 `matcher`

C’est une fonction `(context) => boolean` → détermine si le hook doit s’exécuter.\
Exemples :

* Vérifier un header custom
* Filtrer sur une route (`/sign-up/email`)
* Limiter aux requêtes venant d’un domaine précis

#### 🔹 `handler`

C’est la fonction **middleware** créée avec `createAuthMiddleware`.\
Elle reçoit `ctx` et te permet de :

* ✅ lire/modifier le contexte
* ✅ retourner une réponse custom
* ✅ interrompre le flux avec une erreur

***

### 🚦 Exemple concret : restriction d’accès

👉 Bloquer l’accès aux endpoints d’un plugin si un header `x-api-key` n’est pas présent.

```ts
import { createAuthMiddleware, APIError } from "better-auth/plugins";

const securePlugin = () => {
  return {
    id: "secure-plugin",
    hooks: {
      before: [
        {
          matcher: () => true, // s’applique partout
          handler: createAuthMiddleware(async (ctx) => {
            const apiKey = ctx.headers.get("x-api-key");
            if (apiKey !== process.env.MY_API_KEY) {
              throw new APIError("UNAUTHORIZED", {
                message: "API Key invalide 🚫",
              });
            }
          }),
        },
      ],
    },
  };
};
```

***

### 🚀 Points forts des hooks dans plugins

* 🎯 Ciblage précis avec `matcher`
* 🔄 Personnalisation avant/après l’action
* 🔒 Sécurité (validation, filtrage, audit)
* ⚡ Rapidité → éviter de réécrire des endpoints séparés

## 🛡️ Middleware dans un Plugin BetterAuth

Un **middleware** est un bloc de logique qui s’exécute **avant un endpoint**, uniquement quand la requête passe par l’API (donc via `fetch`, `authClient`, etc.).\
👉 Contrairement aux **hooks**, si tu appelles la fonction directement depuis le serveur (`auth.api...`), le middleware **ne s’exécutera pas**.

***

### 📌 Exemple simple

```ts
// plugin.ts
import { createAuthMiddleware } from "better-auth/plugins";

const myPlugin = () => {
  return {
    id: "my-plugin",
    middlewares: [
      {
        // Ce middleware s’applique uniquement à la route /my-plugin/hello-world
        path: "/my-plugin/hello-world",
        middleware: createAuthMiddleware(async (ctx) => {
          console.log("Middleware exécuté ✅", ctx.path);

          // Exemple : vérifier un header
          const token = ctx.headers.get("x-custom-token");
          if (!token) {
            throw new Error("Token manquant ❌");
          }
        }),
      },
    ],
  } satisfies BetterAuthPlugin;
};
```

***

### ⚙️ Propriétés importantes

* **`path`** → la route ciblée
  * peut être une **chaîne exacte** (`"/sign-up/email"`)
  * ou un **matcher avancé** (wildcards avec Better Call → ex: `"/api/*"`)
* **`middleware`** → une fonction créée avec `createAuthMiddleware`
  * reçoit un `ctx` (contenant `headers`, `body`, `query`, etc.)
  * peut retourner :
    * **rien** → la requête continue normalement
    * **`APIError`** → stoppe la requête avec une erreur
    * **`Response`** → stoppe la requête et renvoie une réponse custom

***

### 🚦 Exemple : authentification via clé API

```ts
import { createAuthMiddleware, APIError } from "better-auth/plugins";

const securePlugin = () => {
  return {
    id: "secure-plugin",
    middlewares: [
      {
        path: "/secure/*", // s’applique à toutes les routes de ce plugin
        middleware: createAuthMiddleware(async (ctx) => {
          const apiKey = ctx.headers.get("x-api-key");
          if (apiKey !== process.env.MY_API_KEY) {
            throw new APIError("UNAUTHORIZED", {
              message: "Accès refusé 🚫",
            });
          }
        }),
      },
    ],
  } satisfies BetterAuthPlugin;
};
```

***

### 🔑 Différence **Middleware** vs **Hooks**

| **Aspect**                     | **Hooks** 🪝                                                  | **Middleware** 🛡️                               |
| ------------------------------ | ------------------------------------------------------------- | ------------------------------------------------ |
| **Quand ça s’exécute ?**       | Avant/après un endpoint, même en appel direct (`auth.api...`) | Uniquement sur les requêtes API venant du client |
| **Usage typique**              | Validation, enrichissement du contexte, side-effects          | Authentification, filtrage des requêtes, logs    |
| **Positionnement**             | Niveau global (server) ou plugin                              | Niveau route/plugin uniquement                   |
| **Peut modifier la réponse ?** | Oui (via `ctx.json`, `ctx.redirect`, etc.)                    | Oui (en retournant une `Response`)               |

## 🔄 On Request & On Response dans un Plugin BetterAuth

Ces deux hooks sont très pratiques pour ajouter une logique **transversale** (comme du logging, de la sécurité ou du monitoring).

***

### 📌 On Request

➡️ S’exécute **juste avant** qu’une requête ne parte.\
Il reçoit :

* `request` → l’objet `Request` complet (headers, body, etc.)
* `context` → le contexte BetterAuth (options, cookies, session, etc.)

#### ✅ Ce que tu peux faire :

* **Continuer normalement** → si tu ne retournes rien.
* **Stopper la requête** → retourne un objet `{ response }` avec une `Response`.
* **Modifier la requête** → retourne un nouvel objet `request`.

#### Exemple : Ajouter un header de sécurité

```ts
const myPlugin = () => {
  return {
    id: "my-plugin",
    onRequest: async (request, context) => {
      console.log("📡 Nouvelle requête vers :", request.url);

      // Ajouter un header custom
      const newRequest = new Request(request, {
        headers: {
          ...Object.fromEntries(request.headers),
          "x-powered-by": "BetterAuth",
        },
      });

      return { request: newRequest };
    },
  } satisfies BetterAuthPlugin;
};
```

***

### 📌 On Response

➡️ S’exécute **juste après** qu’une réponse a été produite, mais avant qu’elle ne soit renvoyée au client.\
Il reçoit :

* `response` → l’objet `Response`
* `context` → le contexte BetterAuth

#### ✅ Ce que tu peux faire :

* **Modifier la réponse** → retourner un `Response` modifié.
* **Laisser passer tel quel** → si tu ne retournes rien.

#### Exemple : Ajouter un header global

```ts
const myPlugin = () => {
  return {
    id: "my-plugin",
    onResponse: async (response, context) => {
      console.log("📦 Réponse générée avec status :", response.status);

      // Ajouter un header à toutes les réponses
      const newHeaders = new Headers(response.headers);
      newHeaders.set("x-response-timestamp", Date.now().toString());

      return new Response(response.body, {
        ...response,
        headers: newHeaders,
      });
    },
  } satisfies BetterAuthPlugin;
};
```

***

## 🚦 Différences avec Hooks & Middleware

| **Mécanisme**            | **Quand ?**             | **Cible**                             | **Cas d’usage**                                    |
| ------------------------ | ----------------------- | ------------------------------------- | -------------------------------------------------- |
| **Hook Before/After** 🪝 | Avant/après un endpoint | Endpoints spécifiques (via `matcher`) | Validation, enrichir le contexte, logs ciblés      |
| **Middleware** 🛡️       | Avant un endpoint       | Routes API uniquement                 | Auth API key, rate limit, filtrage                 |
| **On Request** 📡        | Avant toute requête     | Global (tout plugin)                  | Ajouter headers, bloquer certaines requêtes        |
| **On Response** 📦       | Après la réponse        | Global (tout plugin)                  | Ajouter headers, modifier la réponse, logs globaux |

## 🚦 Rate Limit dans un Plugin BetterAuth

BetterAuth te permet de définir des **règles de limitation de débit** directement au niveau d’un plugin.\
👉 Tu peux cibler des routes précises et limiter le nombre de requêtes dans une **fenêtre temporelle donnée**.

#### 📌 Exemple simple : limiter une route

```ts
// plugin.ts
const myPlugin = () => {
  return {
    id: "my-plugin",
    rateLimit: [
      {
        pathMatcher: (path) => path === "/my-plugin/hello-world", // cible
        limit: 10,   // max 10 requêtes
        window: 60,  // par 60 secondes
      },
    ],
  } satisfies BetterAuthPlugin;
};
```

✅ Ici, un client ne pourra pas appeler `/my-plugin/hello-world` plus de **10 fois par minute**.\
Au-delà → une **erreur 429 (Too Many Requests)** est renvoyée automatiquement.

***

## 🛠️ Helpers Serveur pour Plugins

BetterAuth inclut des fonctions utilitaires pour simplifier la vie côté serveur 👇

***

### 🔍 `getSessionFromCtx`

Permet de **récupérer la session utilisateur** directement depuis le `ctx` du middleware.\
Pratique si tu veux exécuter une logique conditionnée à l’utilisateur connecté.

```ts
import { createAuthMiddleware } from "better-auth/plugins";
import { getSessionFromCtx } from "better-auth/api";

const myPlugin = {
  id: "my-plugin",
  hooks: {
    before: [
      {
        matcher: (context) => context.headers.get("x-my-header") === "my-value",
        handler: createAuthMiddleware(async (ctx) => {
          const session = await getSessionFromCtx(ctx);

          if (!session) {
            console.log("🚫 Pas de session valide");
          } else {
            console.log("✅ Session active pour :", session.user.email);
          }

          return { context: ctx };
        }),
      },
    ],
  },
} satisfies BetterAuthPlugin;
```

***

### 🔑 `sessionMiddleware`

Un middleware prêt à l’emploi pour **forcer la présence d’une session valide**.\
➡️ Si la session est valide, elle est injectée dans `ctx.context.session`.\
➡️ Sinon → erreur **401 Unauthorized** automatique.

```ts
import { createAuthEndpoint } from "better-auth/api";
import { sessionMiddleware } from "better-auth/api";

const myPlugin = () => {
  return {
    id: "my-plugin",
    endpoints: {
      getHelloWorld: createAuthEndpoint(
        "/my-plugin/hello-world",
        {
          method: "GET",
          use: [sessionMiddleware], // ✅ vérifie la session
        },
        async (ctx) => {
          const session = ctx.context.session; // dispo ici !
          return ctx.json({
            message: `Hello ${session.user.name} 👋`,
          });
        }
      ),
    },
  } satisfies BetterAuthPlugin;
};
```

***

## 🚀 Résumé

* **Rate Limit** → protège tes endpoints contre l’abus (DoS, spam, etc.).
* **`getSessionFromCtx`** → récupère facilement la session d’un utilisateur dans un hook/middleware.
* **`sessionMiddleware`** → middleware tout fait pour **protéger tes routes** et injecter la session directement dans le contexte.

## 🖥️ Créer un **Client Plugin** dans BetterAuth

### 📌 1. Plugin minimal

Tu définis ton plugin client en exportant un objet qui satisfait l’interface `BetterAuthClientPlugin` :

```ts
// client-plugin.ts
import type { BetterAuthClientPlugin } from "better-auth/client";

export const myPluginClient = () => {
  return {
    id: "my-plugin", // 🔑 identifiant unique, doit matcher le plugin serveur
  } satisfies BetterAuthClientPlugin;
};
```

***

### 📌 2. Inférer les Endpoints du Plugin Serveur

👉 Pour que ton client sache appeler les endpoints créés dans ton plugin serveur (`createAuthEndpoint`), tu dois ajouter la clé **`$InferServerPlugin`**.

BetterAuth convertit automatiquement les **paths kebab-case** en **camelCase**.\
Exemple :

* `/my-plugin/hello-world` → `myPlugin.helloWorld()`

```ts
// client-plugin.ts
import type { BetterAuthClientPlugin } from "better-auth/client";
import type { myPlugin } from "./plugin"; // ton plugin serveur

export const myPluginClient = () => {
  return {
    id: "my-plugin",
    $InferServerPlugin: {} as ReturnType<typeof myPlugin>, // 🔗 lien avec plugin serveur
  } satisfies BetterAuthClientPlugin;
};
```

***

### 📌 3. Ajouter des Actions Custom (getActions)

Tu peux aussi enrichir ton plugin client avec des méthodes spécifiques via **`getActions`**.\
BetterAuth utilise **Better Fetch** (wrapper amélioré de `fetch`) → donc tu as accès à `$fetch`.

```ts
// client-plugin.ts
import type { BetterAuthClientPlugin } from "better-auth/client";
import type { myPlugin } from "./plugin";
import type { BetterFetchOption } from "@better-fetch/fetch";

export const myPluginClient = {
  id: "my-plugin",
  $InferServerPlugin: {} as ReturnType<typeof myPlugin>,

  getActions: ($fetch) => {
    return {
      // 🎯 Action personnalisée
      myCustomAction: async (
        data: { foo: string },
        fetchOptions?: BetterFetchOption
      ) => {
        const res = await $fetch("/custom/action", {
          method: "POST",
          body: { foo: data.foo },
          ...fetchOptions, // permet d’ajouter retry, timeout, callbacks...
        });

        return res; // toujours retourner { data, error }
      },
    };
  },
} satisfies BetterAuthClientPlugin;
```

***

## 🧩 Bonnes pratiques

✅ Les fonctions du client doivent idéalement :

* prendre **un seul argument obligatoire** (objet `data`)
* et un **second argument optionnel** (`fetchOptions`)

✅ Toujours retourner `{ data, error }` → cohérence avec le reste de BetterAuth.

✅ Si ton use case **ne concerne pas un appel API** (ex: logique pure client), tu peux déroger à cette règle.

***

## 🚀 Exemple complet d’utilisation

#### 📌 Côté serveur (`plugin.ts`)

```ts
import { createAuthEndpoint } from "better-auth/api";
import type { BetterAuthPlugin } from "better-auth";

export const myPlugin = () => {
  return {
    id: "my-plugin",
    endpoints: {
      helloWorld: createAuthEndpoint(
        "/my-plugin/hello-world",
        { method: "GET" },
        async (ctx) => {
          return ctx.json({ message: "Hello World 🌍" });
        }
      ),
    },
  } satisfies BetterAuthPlugin;
};
```

#### 📌 Côté client (`client-plugin.ts`)

```ts
import type { BetterAuthClientPlugin } from "better-auth/client";
import type { myPlugin } from "./plugin";

export const myPluginClient = () => {
  return {
    id: "my-plugin",
    $InferServerPlugin: {} as ReturnType<typeof myPlugin>,

    getActions: ($fetch) => ({
      greetUser: async (name: string) => {
        return await $fetch("/my-plugin/hello-world", {
          method: "GET",
        });
      },
    }),
  } satisfies BetterAuthClientPlugin;
};
```

#### 📌 Usage dans ton frontend

```ts
import { createAuthClient } from "better-auth/react";
import { myPluginClient } from "./client-plugin";

const authClient = createAuthClient({
  plugins: [myPluginClient()],
});

// Appel du endpoint serveur
const { data, error } = await authClient.myPlugin.helloWorld();

// Appel de l’action custom
const custom = await authClient.myPlugin.greetUser("Alice");
```

## 🧩 **Get Atoms dans un plugin client**

### 🔎 Rappel rapide

* BetterAuth côté client utilise **nanostores** pour stocker l’état.
* `getAtoms` permet à ton plugin de créer ses propres **atoms réactifs**.
* Chaque framework (React, Vue, Svelte, Solid, …) peut ensuite consommer ces atoms via son propre hook (`useStore`, `useAtom`, etc.).

***

### 📌 Exemple minimal

```ts
// client-plugin.ts
import { atom } from "nanostores";
import type { BetterAuthClientPlugin } from "better-auth/client";
import type { myPlugin } from "./plugin";

export const myPluginClient = {
  id: "my-plugin",
  $InferServerPlugin: {} as ReturnType<typeof myPlugin>,

  getAtoms: ($fetch) => {
    // Créer un atom nanostores (valeur initiale: null)
    const myAtom = atom<string | null>(null);

    // Tu peux initialiser ou mettre à jour ton atom ici via $fetch
    // Exemple: fetch une valeur côté serveur
    (async () => {
      const { data } = await $fetch("/my-plugin/hello-world", { method: "GET" });
      if (data?.message) {
        myAtom.set(data.message);
      }
    })();

    return {
      myAtom, // sera dispo comme authClient.myPlugin.myAtom
    };
  },
} satisfies BetterAuthClientPlugin;
```

***

### 📌 Utilisation dans React

```tsx
import { useStore } from "@nanostores/react";
import { createAuthClient } from "better-auth/react";
import { myPluginClient } from "./client-plugin";

const authClient = createAuthClient({
  plugins: [myPluginClient],
});

export function MyComponent() {
  const myValue = useStore(authClient.myPlugin.myAtom);

  return <div>Valeur de mon atom: {myValue ?? "..."}</div>;
}
```

***

### 🚀 Cas d’usage des Atoms

* **Stocker l’état utilisateur custom** (ex: préférences, niveau, points de fidélité).
* **Créer un hook réactif** : ton atom peut être exposé comme `useUserPoints()` par exemple.
* **Synchroniser automatiquement avec ton backend** : via `$fetch`, tu mets à jour l’atom dès qu’une action se produit.

## ⚙️ **Résumé des options supplémentaires pour les client plugins**

### 🔹 1. Path Methods

Par défaut :

* **GET** si ton endpoint n’a pas de body
* **POST** si ton endpoint attend un body

👉 Mais tu peux **forcer** le type de méthode :

```ts
import type { BetterAuthClientPlugin } from "better-auth/client";
import type { myPlugin } from "./plugin";

export const myPluginClient = {
  id: "my-plugin",
  $InferServerPlugin: {} as ReturnType<typeof myPlugin>,

  // 🔄 Forcer un chemin à utiliser POST
  pathMethods: {
    "/my-plugin/hello-world": "POST"
  }
} satisfies BetterAuthClientPlugin;
```

⚠️ Utile si ton endpoint serveur est atypique (par exemple un `GET` qui prend quand même un body, ou un `POST` qui ne prend rien).

***

### 🔹 2. Fetch Plugins

Better Auth utilise **Better Fetch** (un wrapper autour de `fetch`).\
Tu peux **brancher des plugins** pour :

* Ajouter un header automatiquement (ex. `Authorization`)
* Logger toutes les requêtes
* Ajouter un retry ou un timeout global

```ts
import { authFetchLogger } from "@better-fetch/logger";
import type { BetterAuthClientPlugin } from "better-auth/client";

export const myPluginClient = {
  id: "my-plugin",
  fetchPlugins: [
    authFetchLogger({ level: "debug" }) // plugin d’exemple
  ]
} satisfies BetterAuthClientPlugin;
```

***

### 🔹 3. Atom Listeners

👉 Seulement utile si tu crées des **atoms réactifs** (`useSession`, `useUserData`, etc.).

Tu peux **écouter des changements** sur un atom et recalculer un autre atom ou déclencher une action.

Exemple (pseudo-code inspiré des built-in plugins) :

```ts
import { atom } from "nanostores";
import type { BetterAuthClientPlugin } from "better-auth/client";

export const myPluginClient = {
  id: "my-plugin",
  getAtoms: ($fetch) => {
    const userAtom = atom<{ name: string } | null>(null);
    const premiumAtom = atom<boolean>(false);

    // 🎯 Listener : si userAtom change, recalculer premiumAtom
    userAtom.listen((user) => {
      premiumAtom.set(user?.name === "VIP User");
    });

    return { userAtom, premiumAtom };
  }
} satisfies BetterAuthClientPlugin;
```

***

## 🚀 En résumé

* **`pathMethods`** → corrige les inférences automatiques GET/POST.
* **`fetchPlugins`** → ajoute des middlewares sur `fetch`.
* **`atom listeners`** → permet de créer des hooks plus riches, synchronisés entre eux.
