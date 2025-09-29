# 🪝 Hooks dans BetterAuth

Les **Hooks** permettent de s’**accrocher** au cycle de vie de BetterAuth pour exécuter du **code personnalisé** à des moments précis.\
👉 Ils offrent un moyen de **modifier le comportement** de BetterAuth **sans avoir à écrire un plugin complet**.

***

### 📌 Pourquoi utiliser les hooks ?

* 🎯 **Personnalisation fine** : ajouter des actions spécifiques sur un endpoint existant.
* ⚡ **Simplicité** : pas besoin de créer une API séparée pour chaque ajustement.
* 🔄 **Intégration native** : votre logique s’exécute directement dans le flux BetterAuth (après sign-in, avant sign-up, etc.).
* 🔒 **Centralisation** : vous gardez toute la logique liée à l’auth dans BetterAuth, au lieu de la disperser dans votre code.

***

### 🧐 Quand utiliser un hook ?

* Après qu’un utilisateur se soit inscrit → lui attribuer un rôle par défaut.
* Lors d’une connexion → logger l’événement ou vérifier une condition business.
* Après une vérification d’email → envoyer un bonus de bienvenue.
* Avant une action sensible → bloquer ou rediriger selon vos règles.

***

### 🚀 Bonnes pratiques

* ✅ **Utilisez un hook si vous voulez étendre un endpoint existant**.
* ❌ **N’écrivez pas un nouvel endpoint externe** juste pour ça → vous perdriez l’intégration native de BetterAuth.
* ✅ Combinez les hooks avec vos outils internes (DB, analytics, monitoring, emails, etc.).

***

👉 En résumé :\
Les **Hooks** sont un outil **léger et puissant** pour personnaliser BetterAuth.\
Ils sont parfaits si vous voulez **adapter un comportement spécifique sans créer un plugin complet** 🎉.

## ⏳ Before Hooks (Avant exécution d’un endpoint)

Les **Before Hooks** s’exécutent **avant qu’un endpoint BetterAuth ne soit exécuté**.\
👉 Ils sont utiles pour :

* 🔍 **Modifier la requête** (ex: compléter les données envoyées).
* ✅ **Valider des règles métier** avant d’autoriser l’action.
* ⛔ **Stopper l’exécution** si une condition échoue (retour d’erreur).

***

### 📌 Exemple 1 : Restreindre un domaine email

Ici, on empêche les utilisateurs de s’inscrire avec une adresse qui ne se termine pas par `@example.com`.

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { createAuthMiddleware, APIError } from "better-auth/api";

export const auth = betterAuth({
  hooks: {
    before: createAuthMiddleware(async (ctx) => {
      if (ctx.path !== "/sign-up/email") {
        return; // ✅ On laisse passer les autres endpoints
      }

      if (!ctx.body?.email.endsWith("@example.com")) {
        throw new APIError("BAD_REQUEST", {
          message: "Email must end with @example.com",
        });
      }
    }),
  },
});
```

🛠 Fonctionnement :

1. On intercepte le flux **avant** l’appel du endpoint `/sign-up/email`.
2. On vérifie le domaine de l’email.
3. ❌ Si ce n’est pas conforme → on renvoie une **erreur 400** avec un message explicite.

***

### 📌 Exemple 2 : Modifier le contexte de requête

On peut aussi enrichir/modifier les données avant de laisser continuer l’exécution.

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { createAuthMiddleware } from "better-auth/api";

export const auth = betterAuth({
  hooks: {
    before: createAuthMiddleware(async (ctx) => {
      if (ctx.path === "/sign-up/email") {
        return {
          context: {
            ...ctx,
            body: {
              ...ctx.body,
              name: "John Doe", // 👤 Valeur par défaut pour "name"
            },
          },
        };
      }
    }),
  },
});
```

⚡ Ici :

* On cible uniquement le **sign-up par email**.
* On injecte automatiquement `name: "John Doe"` si la valeur est manquante.

***

### 🎯 Cas d’usage typiques des _Before Hooks_

* 🚫 Restreindre l’inscription à certains emails (ex: entreprise).
* 🔐 Ajouter une **validation supplémentaire** (ex: mot de passe conforme à une règle interne).
* 🛠 Modifier ou enrichir le `body` (ajouter des métadonnées, pré-remplir un champ).
* ⚡ Appliquer des contrôles business (ex: limiter l’accès à une fonctionnalité en bêta).

***

👉 En résumé :\
Les **Before Hooks** vous permettent d’**agir avant qu’un endpoint ne soit exécuté**, que ce soit pour **bloquer, enrichir ou transformer** la requête.\
Ils sont parfaits pour **appliquer vos propres règles métier** directement dans le flux BetterAuth 🔥.

## ⏭️ After Hooks (Après exécution d’un endpoint)

Les **After Hooks** s’exécutent **juste après qu’un endpoint BetterAuth ait terminé son exécution**.\
👉 Leur rôle :

* ✨ Modifier la **réponse** avant qu’elle ne soit renvoyée au client.
* 📡 Déclencher des **actions secondaires** (notifications, logs, analytics, etc.).
* 🛠 Appliquer une **logique post-traitement** sur l’événement.

***

### 📌 Exemple 1 : Envoyer une notification quand un utilisateur s’inscrit

Ici, on intercepte le flux **après inscription** pour prévenir un canal externe (Slack, Discord, webhook, etc.).

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { createAuthMiddleware } from "better-auth/api";
import { sendMessage } from "@/lib/notification";

export const auth = betterAuth({
  hooks: {
    after: createAuthMiddleware(async (ctx) => {
      // ✅ On cible les endpoints d’inscription
      if (ctx.path.startsWith("/sign-up")) {
        const newSession = ctx.context.newSession;

        if (newSession) {
          // 📡 Envoi d’une notification externe
          sendMessage({
            type: "user-register",
            name: newSession.user.name,
          });
        }
      }
    }),
  },
});
```

***

### 📌 Exemple 2 : Modifier la réponse envoyée au client

On peut également **transformer ou enrichir la réponse** avant de la renvoyer.

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { createAuthMiddleware } from "better-auth/api";

export const auth = betterAuth({
  hooks: {
    after: createAuthMiddleware(async (ctx) => {
      if (ctx.path === "/sign-in/email") {
        return {
          response: {
            ...ctx.response,
            customMessage: "Bienvenue 🎉",
          },
        };
      }
    }),
  },
});
```

👉 Dans cet exemple :

* Lors d’une connexion réussie, la réponse renvoyée au client contient un champ supplémentaire `customMessage`.

***

### 🎯 Cas d’usage typiques des _After Hooks_

* 📩 **Envoyer un email** ou une notification après un événement (inscription, changement de mot de passe, etc.).
* 📊 **Logger l’activité** (connexion, déconnexion, échec d’authentification).
* 🛠 **Enrichir la réponse** avec des métadonnées ou un message personnalisé.
* 🔔 **Déclencher des workflows business** (ex: activer un abonnement, créer une entrée CRM après inscription).

***

👉 En résumé :\
Les **After Hooks** vous permettent de **réagir aux événements post-authentification**.\
Ils sont idéaux pour **connecter BetterAuth à vos outils externes** (monitoring, analytics, notifications, workflows internes) tout en **modifiant les réponses si besoin**.

## 🗂️ Le contexte (`ctx`) dans BetterAuth

Quand tu utilises **`createAuthMiddleware`**, la fonction reçoit un objet **`ctx`**.\
👉 Cet objet est une **boîte à outils riche** qui contient toutes les infos utiles sur :

* 📌 la requête en cours,
* 📌 les données transmises (body, query, headers…),
* 📌 le contexte d’authentification de BetterAuth (sessions, cookies, config, etc.).

Tu peux donc **lire, modifier ou utiliser** ces infos pour personnaliser ton flux d’authentification.

***

### 🛠️ Propriétés principales de `ctx`

#### 🔗 1. `ctx.path`

Chemin de l’endpoint actuellement appelé.

* Exemples :
  * `/sign-in/email`
  * `/sign-up/email`
  * `/verify-email`

👉 Très utile pour **cibler un hook sur un endpoint spécifique**.

***

#### 📦 2. `ctx.body`

Contient le **corps de la requête** (déjà parsé).

* Disponible uniquement pour les requêtes **POST**.
* Exemple : lors d’un sign-up, `ctx.body.email` et `ctx.body.password`.

👉 Utile pour appliquer des **validations custom** ou enrichir les données avant traitement.

***

#### 📑 3. `ctx.headers`

Accès aux **headers de la requête HTTP**.

* Exemples :
  * `ctx.headers["user-agent"]`
  * `ctx.headers["x-forwarded-for"]` (IP du client si proxys).

👉 Très pratique pour :

* détecter le **navigateur / device** de l’utilisateur,
* appliquer une sécurité supplémentaire (origine, API keys…).

***

#### 🌐 4. `ctx.request`

L’objet **Request** brut.

* Peut ne pas exister dans certains endpoints _server-only_.
* Contient tout le payload HTTP original.

👉 À utiliser si tu veux un **contrôle bas niveau** (rare).

***

#### ❓ 5. `ctx.query`

Contient les **paramètres de query** de l’URL.

* Exemple : `/verify-email?token=12345`
  * `ctx.query.token === "12345"`

👉 Indispensable pour valider des **tokens envoyés en query** (ex: vérification email, reset password).

***

#### 🔐 6. `ctx.context`

C’est le **contexte auth BetterAuth**. C’est ici que tu retrouves :

* ✅ `newSession` → la session fraichement créée (après sign-up, sign-in…).
* ✅ `authCookies` → configuration des cookies (nom, durée, etc.).
* ✅ `hashPassword` → méthode utilitaire pour hacher un mot de passe.
* ✅ `config` → la config complète de BetterAuth (tes options).

👉 Très puissant pour **interagir directement avec l’écosystème auth**.

***

### 🎯 Exemple concret : journaliser un sign-in

```ts
import { betterAuth } from "better-auth"
import { createAuthMiddleware } from "better-auth/api"

export const auth = betterAuth({
  hooks: {
    after: createAuthMiddleware(async (ctx) => {
      if (ctx.path === "/sign-in/email") {
        console.log("Nouvelle connexion !");
        console.log("📧 Email :", ctx.body?.email);
        console.log("🌍 IP :", ctx.headers["x-forwarded-for"]);
        console.log("🆔 Session ID :", ctx.context.newSession?.id);
      }
    }),
  },
});
```

***

### 🚀 En résumé

Le **`ctx`** est le **couteau suisse** des hooks BetterAuth :

* `path` → savoir **quel endpoint** est appelé.
* `body` → accéder/modifier les **données envoyées**.
* `headers` → lire des infos réseau (IP, user-agent).
* `query` → lire les **tokens en URL**.
* `context` → interagir avec la **session, cookies, config et utilitaires BetterAuth**.

## 📬 Request & Response dans les Hooks

Avec BetterAuth, tu peux **contrôler les requêtes et réponses HTTP directement depuis un hook** via l’objet `ctx`.\
👉 Cela te permet de personnaliser **comment BetterAuth répond à une requête** avant même que la réponse finale ne parte.

***

### 📦 1. Réponses JSON

Utilise `ctx.json` pour envoyer une réponse JSON personnalisée :

```ts
const hook = createAuthMiddleware(async (ctx) => {
  return ctx.json({
    message: "Hello World",
  });
});
```

⚡ Exemple concret : renvoyer une API customisée directement depuis un hook.

***

### 🔀 2. Redirections

Utilise `ctx.redirect` pour rediriger un utilisateur vers une autre page :

```ts
import { createAuthMiddleware } from "better-auth/api";

const hook = createAuthMiddleware(async (ctx) => {
  throw ctx.redirect("/sign-up/name");
});
```

👉 Très utile après une étape incomplète (par ex : forcer l’utilisateur à compléter son profil).

***

### 🍪 3. Gestion des Cookies

BetterAuth fournit des méthodes pratiques :

* `ctx.setCookies(name, value)` → définit un cookie standard.
* `ctx.setSignedCookie(name, value, secret, options)` → définit un cookie **signé** pour garantir son intégrité.
* `ctx.getCookies(name)` → récupère un cookie.
* `ctx.getSignedCookies(name)` → récupère et **vérifie** un cookie signé.

```ts
import { createAuthMiddleware } from "better-auth/api";

const hook = createAuthMiddleware(async (ctx) => {
  ctx.setCookies("my-cookie", "value");

  await ctx.setSignedCookie("my-signed-cookie", "value", ctx.context.secret, {
    maxAge: 1000,
  });

  const cookie = ctx.getCookies("my-cookie");
  const signedCookie = await ctx.getSignedCookies("my-signed-cookie");
});
```

***

### ⚠️ 4. Gestion des Erreurs

Pour lancer une erreur propre avec un code HTTP spécifique → utilise `APIError`.

```ts
import { createAuthMiddleware, APIError } from "better-auth/api";

const hook = createAuthMiddleware(async (ctx) => {
  throw new APIError("BAD_REQUEST", {
    message: "Invalid request",
  });
});
```

***

### 🔐 5. Contexte (`ctx.context`)

L’objet `ctx.context` contient **toutes les infos auth internes**.

#### ✨ Nouvel utilisateur ou session

Dans un `after hook`, tu peux récupérer la **nouvelle session créée** :

```ts
createAuthMiddleware(async (ctx) => {
  const newSession = ctx.context.newSession;
});
```

***

#### 🔄 Valeur retournée par le hook précédent

Chaque hook peut transmettre un retour au suivant :

```ts
createAuthMiddleware(async (ctx) => {
  const returned = ctx.context.returned; 
  // Peut être une réponse ou une APIError
});
```

***

#### 📑 Headers de réponse

Tu peux accéder aux headers générés par BetterAuth ou d’autres hooks :

```ts
createAuthMiddleware(async (ctx) => {
  const responseHeaders = ctx.context.responseHeaders;
});
```

***

#### 🍪 Cookies pré-définis par BetterAuth

Accès direct aux cookies internes comme `sessionToken` :

```ts
createAuthMiddleware(async (ctx) => {
  const cookieName = ctx.context.authCookies.sessionToken.name;
});
```

***

#### 🔑 Secret d’instance

Le **secret d’auth** utilisé par BetterAuth :

```ts
const secret = ctx.context.secret;
```

***

#### 🔐 Gestion des mots de passe

BetterAuth fournit directement des helpers :

```ts
// Hacher un mot de passe
await ctx.context.password.hash("plainPassword");

// Vérifier un mot de passe
await ctx.context.password.verify("plainPassword", "hashedPassword");
```

***

#### 🗄️ Accès aux Adapters (DB)

BetterAuth expose les méthodes DB via `adapter` :

* `findOne`, `findMany`, `create`, `update`, `delete`, `updateMany`

👉 Mais ⚠️ il est recommandé d’utiliser directement ton **ORM** (Prisma, Drizzle, etc.) sauf si tu veux profiter des **databaseHooks** et du support **secondaryStorage**.

***

#### 🔧 Internal Adapter

Méthodes spécialisées comme :

* `createUser`
* `createSession`
* `updateSession`

👉 Pratique si tu veux **bénéficier de la logique interne BetterAuth** au lieu de recréer des requêtes similaires côté ORM.

***

#### 🆔 Génération d’ID

Utilise `ctx.context.generateId()` pour générer des IDs uniques.

```ts
const newId = ctx.context.generateId();
```

***

### 🎯 En résumé

Avec `ctx`, tu peux :

* 📤 Créer des réponses JSON (`ctx.json`)
* 🔀 Rediriger un utilisateur (`ctx.redirect`)
* 🍪 Gérer cookies (set/get, signés ou non)
* ⚠️ Gérer les erreurs (`APIError`)
* 🔐 Accéder au **contexte auth complet** (sessions, cookies, hashing, config, DB)
* 🆔 Générer des IDs uniques

## 🔁 Hooks Réutilisables

Les **hooks** sont très puissants pour personnaliser le comportement de BetterAuth.\
👉 Mais si tu commences à les dupliquer sur plusieurs endpoints, ton code devient vite difficile à maintenir.

***

### 📌 Bonne pratique

➡️ Si tu veux **réutiliser un hook sur plusieurs endpoints**, il est recommandé de créer un **plugin** plutôt que de copier-coller le hook.

* ✅ **Centralisation** → ta logique custom est regroupée au même endroit.
* ✅ **Réutilisation** → tu peux l’activer/désactiver facilement sur plusieurs projets.
* ✅ **Lisibilité** → tu sépares la logique métier de la configuration auth.
* ✅ **Évolutivité** → plus simple à maintenir et à enrichir dans le temps.

***

### 🚀 Exemple d’approche

1. **Avec un simple hook inline** (bien pour un besoin unique) :

```ts
auth: betterAuth({
  hooks: {
    before: createAuthMiddleware(async (ctx) => {
      if (ctx.path === "/sign-up/email") {
        // Vérifications custom ici
      }
    }),
  },
});
```

2. **Avec un plugin** (mieux si besoin récurrent) :

```ts
// plugins/restrict-domain.ts
import { createAuthMiddleware } from "better-auth/api";

export function restrictDomainPlugin(allowedDomain: string) {
  return {
    hooks: {
      before: createAuthMiddleware(async (ctx) => {
        if (ctx.path === "/sign-up/email") {
          if (!ctx.body?.email.endsWith(allowedDomain)) {
            throw new Error(`Email must end with ${allowedDomain}`);
          }
        }
      }),
    },
  };
}
```

Puis, dans ta config :

```ts
import { betterAuth } from "better-auth";
import { restrictDomainPlugin } from "./plugins/restrict-domain";

export const auth = betterAuth({
  plugins: [
    restrictDomainPlugin("@example.com")
  ],
});
```

***

### 🎯 En résumé

* Utilise des **hooks inline** pour un besoin ponctuel.
* Si tu veux un hook **réutilisable, portable et maintenable** → transforme-le en **plugin**.
