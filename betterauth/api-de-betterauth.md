# 🌐 API de BetterAuth

Lorsque vous créez une nouvelle instance **BetterAuth**, celle-ci vous fournit automatiquement un objet **`api`**.

👉 Cet objet expose **tous les endpoints** disponibles dans votre instance BetterAuth :

* 🔹 Ceux du **noyau (core)**
* 🔹 Ceux ajoutés par des **plugins**

Ainsi, vous pouvez facilement interagir avec votre serveur BetterAuth **côté serveur** 🖥️, sans avoir à redéfinir manuellement chaque route.

***

### 📌 Exemple simple

```ts
import { auth } from "./auth"; // votre instance BetterAuth

// Utilisation d’un endpoint exposé par BetterAuth
const session = await auth.api.getSession({
  headers: request.headers,
});
```

***

### 🧐 Explications

* **`auth.api`** → contient toutes les routes disponibles (connexion, déconnexion, gestion de session, etc.).
* **Endpoints dynamiques** → si vous ajoutez un **plugin** (par ex. 2FA, passkey, magic link), ses endpoints seront aussi accessibles via **`auth.api`**.
* **Utilisation côté serveur uniquement** → idéal pour :
  * 🔐 vérifier la session d’un utilisateur,
  * 🏢 appliquer des contrôles d’accès,
  * 📡 déclencher des actions d’auth sans passer par le client.

***

### 🎯 Avantages

* 🧩 **Extensible** : tout plugin enrichit automatiquement l’API disponible.
* 🔒 **Sécurisé** : endpoints centralisés, gestion des cookies et sessions intégrée.
* ⚡ **Simplicité** : pas besoin de réécrire vos routes d’auth → elles sont déjà exposées.
* 🌍 **Universalité** : utilisable avec tous les frameworks backend modernes.

***

👉 En résumé :\
L’objet **`api`** est la **porte d’entrée côté serveur** pour exploiter toutes les fonctionnalités de BetterAuth.\
Que ce soit le **core** ou des **plugins**, chaque endpoint devient automatiquement accessible via **`auth.api`** 🎉.

## 🖥️ Appeler les endpoints API côté serveur

Pour interagir avec BetterAuth **depuis votre serveur**, vous pouvez appeler directement les **endpoints exposés par l’objet `api`**.

👉 Il suffit d’importer votre instance **`auth`** et d’appeler la méthode correspondant à l’endpoint dont vous avez besoin.

***

### 📌 Exemple : récupérer une session côté serveur

```ts
// server.ts
import { betterAuth } from "better-auth";
import { headers } from "next/headers";

export const auth = betterAuth({
  // ... vos options
});

// Exemple : appel de getSession côté serveur
await auth.api.getSession({
  headers: await headers() // ⚠️ certains endpoints nécessitent les headers
});
```

***

### 🧐 Explications

* **`auth.api.getSession()`** → permet de récupérer les informations de session d’un utilisateur.
* **`headers`** → transmis pour :
  * lire les cookies d’authentification,
  * valider la session côté serveur.
* **Endpoints dynamiques** → tout plugin ou nouvelle fonctionnalité que vous ajoutez à BetterAuth sera aussi disponible via **`auth.api`**.

***

### 🎯 Cas d’usage côté serveur

* 🔐 **Protéger une route API** (ex : `/api/admin`) en vérifiant que l’utilisateur est bien connecté.
* 🏢 **Contrôle des permissions** → exiger un rôle particulier avant d’exécuter une action.
* 🧑‍💻 **Accéder aux infos utilisateur** pour du rendu serveur (SSR).
* 🌍 **Interopérabilité** → appeler d’autres endpoints ajoutés par vos plugins (ex : 2FA, passkeys, magic link).

***

### 🚀 Avantages

* ✅ **Simplicité** → pas besoin de réécrire vos endpoints : ils sont déjà exposés.
* 🔒 **Sécurité** → gestion automatique des cookies et sessions côté serveur.
* 🧩 **Extensible** → chaque plugin ajoute ses propres endpoints, accessibles directement via `auth.api`.

***

👉 En résumé :\
Avec **`auth.api`**, vous pouvez **appeler directement tous les endpoints BetterAuth côté serveur**.\
C’est la méthode la plus simple et la plus sécurisée pour gérer vos sessions et vos règles d’accès 🔐.

## 📦 Body, Headers & Query dans l’API serveur

Contrairement au **client** (où vous passez directement les valeurs aux méthodes), côté **serveur**, vous devez transmettre les données dans un objet avec des clés spécifiques :

* **`body`** → pour les données du corps de la requête (ex: email, mot de passe).
* **`headers`** → pour les en-têtes (cookies, IP, user-agent, etc.).
* **`query`** → pour les paramètres de requête (ex: token de vérification).

***

### 📌 Exemple d’utilisation

```ts
// Récupérer une session (headers requis)
await auth.api.getSession({
  headers: await headers()
})

// Connexion par email/mot de passe (body + headers)
await auth.api.signInEmail({
  body: {
    email: "john@doe.com",
    password: "password"
  },
  headers: await headers() // facultatif mais utile pour IP, user-agent, etc.
})

// Vérifier un email (query)
await auth.api.verifyEmail({
  query: {
    token: "my_token"
  }
})
```

***

### 🧐 Explications

* **`headers`** :
  * utilisés pour transporter les **cookies d’authentification** 🍪,
  * utiles pour enregistrer des métadonnées comme **l’adresse IP** 🌍 ou le **user-agent** 🖥️.
* **`body`** :
  * contient les données sensibles envoyées lors d’un login, signup, ou activation de fonctionnalités (2FA, etc.).
* **`query`** :
  * utilisé pour des paramètres transmis dans l’URL (par exemple un **token de vérification d’email**).

***

### ⚡ Technologie sous-jacente : Better Call

Les endpoints API de BetterAuth reposent sur [**better-call**](https://www.npmjs.com/package/better-call) ⚡ :

* 🧑‍💻 un micro-framework web minimaliste,
* permet d’appeler des endpoints REST **comme s’il s’agissait de simples fonctions JS/TS**,
* facilite l’**inférence automatique des types** côté client → vous bénéficiez de l’autocomplétion TypeScript 🎉.

***

### 🚀 Avantages

* 🔒 **Cohérence** : toutes les requêtes serveur suivent la même structure (`body`, `headers`, `query`).
* 🧑‍💻 **DX améliorée** : grâce à _better-call_, vous appelez vos endpoints comme de vraies fonctions.
* ⚡ **Sécurité & flexibilité** : headers exploitables pour traçabilité (IP, user-agent), gestion des cookies intégrée.
* 📦 **Interopérabilité** : compatible avec les endpoints core et ceux ajoutés via plugins.

***

👉 En résumé :\
Côté serveur, chaque appel à l’API BetterAuth se fait via un objet structuré (`body`, `headers`, `query`).\
Grâce à **better-call**, ces endpoints s’utilisent comme de simples fonctions TypeScript → un gain énorme en **simplicité** et en **productivité** 🎉.

## 📑 Récupérer les **headers** et l’objet **Response**

Par défaut, lorsqu’on appelle un endpoint de l’API BetterAuth **côté serveur**, celui-ci renvoie simplement un **objet JavaScript** ou un **tableau**.\
👉 En effet, chaque endpoint est en réalité une **fonction TypeScript classique**.

Mais dans certains cas, vous aurez besoin d’aller plus loin :

* 🍪 **Récupérer les cookies** envoyés dans les headers,
* 📑 Lire ou modifier les **headers HTTP**,
* 📦 Accéder directement à l’**objet `Response`**.

BetterAuth vous donne cette flexibilité via deux options : **`returnHeaders`** et **`asResponse`**.

***

### 📌 1. Récupérer les **headers**

Vous pouvez obtenir les headers en ajoutant l’option **`returnHeaders: true`**.

```ts
const { headers, response } = await auth.api.signUpEmail({
  returnHeaders: true, // 👉 active la récupération des headers
  body: {
    email: "john@doe.com",
    password: "password",
    name: "John Doe",
  },
});
```

#### 🔎 Utilisation des headers

```ts
// Récupérer les cookies envoyés par BetterAuth
const cookies = headers.get("set-cookie");

// Récupérer un header personnalisé
const customHeader = headers.get("x-custom-header");
```

👉 Très utile si vous devez gérer les cookies manuellement (par ex. dans Express, Hono ou tout autre serveur qui ne gère pas automatiquement la réponse).

***

### 📌 2. Récupérer l’objet **Response**

Si vous voulez directement l’**objet `Response`** (comme dans les API modernes type Fetch/Next.js), utilisez l’option **`asResponse: true`**.

```ts
// server.ts
const response = await auth.api.signInEmail({
  body: {
    email: "john@doe.com",
    password: "password",
  },
  asResponse: true, // 👉 renvoie un objet Response complet
});
```

Avec `response`, vous pouvez manipuler :

* **status** → le code HTTP (`200`, `401`, `403`, etc.)
* **headers** → via `response.headers.get("set-cookie")`
* **body** → via `await response.json()` ou `await response.text()`

***

### 🧐 Quand utiliser quoi ?

* ✅ **Retour par défaut (objet JS)** → si vous avez juste besoin des **données** utilisateur/session.
* 🍪 **`returnHeaders`** → si vous voulez inspecter ou définir vous-même les **cookies/headers**.
* 📦 **`asResponse`** → si vous préférez manipuler directement une **réponse HTTP complète** (idéal dans **Next.js**, **Hono**, **Remix**, etc.).

***

### 🚀 Points forts

* 🔒 Contrôle total des **cookies et headers** quand nécessaire.
* ⚡ Flexibilité → fonction standard ou réponse HTTP, selon votre contexte.
* 🌍 Compatibilité avec tous les frameworks backend modernes.

***

👉 En résumé :\
BetterAuth vous laisse le choix entre la **simplicité** (objet JS) et le **contrôle avancé** (headers + Response).\
Cela permet de l’utiliser aussi bien dans des frameworks modernes (Next.js, SvelteKit) que dans des serveurs Node.js classiques 🎉.

## ❌ Gestion des erreurs (**Error Handling**)

Lorsque vous appelez un **endpoint API** de BetterAuth côté serveur, une **erreur sera levée** si la requête échoue.\
👉 Cette erreur est une instance de **`APIError`**, ce qui vous permet de la gérer proprement en TypeScript.

***

### 📌 Exemple d’utilisation

```ts
// server.ts
import { APIError } from "better-auth/api";

try {
  await auth.api.signInEmail({
    body: {
      email: "",
      password: ""
    }
  })
} catch (error) {
  if (error instanceof APIError) {
    console.log(error.message, error.status)
  }
}
```

***

### 🧐 Explications

* **`APIError`** est une classe d’erreurs spécifique à BetterAuth.
* Elle contient des propriétés utiles pour le **debug** et la **gestion d’erreurs côté serveur** :
  * **`message`** → description claire de l’erreur (ex: _"Invalid credentials"_)
  * **`status`** → code HTTP correspondant (ex: `400`, `401`, `403`, `500`...)

👉 Cela vous permet de **réagir différemment selon le type d’erreur**.

***

### 🎯 Cas d’usage concrets

#### 🔑 Mauvais identifiants (401)

```ts
if (error.status === 401) {
  return { error: "Email ou mot de passe incorrect" }
}
```

#### 📩 Email non vérifié (403)

```ts
if (error.status === 403) {
  return { error: "Veuillez vérifier votre adresse email avant de vous connecter" }
}
```

#### ⚡ Erreur serveur (500)

```ts
if (error.status === 500) {
  console.error("Erreur serveur BetterAuth :", error.message)
}
```

***

### 🚀 Bonnes pratiques

* ✅ Toujours **catcher vos erreurs** lors des appels `auth.api.*`.
* ✅ Utiliser **`instanceof APIError`** pour distinguer les erreurs BetterAuth des autres erreurs JS/TS.
* ✅ Logger les erreurs critiques (`500`) pour le suivi serveur.
* ✅ Fournir des messages clairs à l’utilisateur côté client (sans exposer d’infos sensibles).

***

### 🧩 Avantages de `APIError`

* 🔒 **Sécurité** : messages standards, évite de divulguer trop d’infos techniques.
* ⚡ **Simplicité** : gestion uniforme pour tous les endpoints BetterAuth.
* 🧑‍💻 **Clarté** : distinction facile entre erreurs BetterAuth et autres erreurs JS.

***

👉 En résumé :\
BetterAuth centralise la **gestion des erreurs serveur** avec `APIError`, vous offrant une manière **claire, typée et sécurisée** de traiter les erreurs dans vos applications 🎉.
