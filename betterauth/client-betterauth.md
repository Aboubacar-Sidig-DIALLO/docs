# 🖥️ Client BetterAuth

BetterAuth fournit une **librairie client** compatible avec les frameworks frontend modernes ✨ :

* ⚛️ **React**
* 🖖 **Vue**
* 🔥 **Svelte**
* 🧊 **Solid**
* 🌍 … et bien d’autres encore !

***

### 📌 Rôle de la librairie client

La librairie client inclut un ensemble de **fonctions prêtes à l’emploi** pour interagir avec le **serveur BetterAuth**.\
👉 Grâce à elle, vous pouvez facilement :

* 🔑 connecter et déconnecter vos utilisateurs,
* 📨 gérer l’inscription, la récupération de mot de passe, la vérification email,
* 🔐 accéder à la **session utilisateur en temps réel**,
* 🌍 utiliser les **fournisseurs sociaux** (Google, GitHub, etc.).

***

### 🧩 Une base commune (core client)

Chaque librairie spécifique à un framework est construite **au-dessus d’un client “core” universel** 🌐.

* Ce **core client** est **framework-agnostic** → il n’est lié à aucun framework particulier.
* Toutes les méthodes et hooks sont **uniformes** → vous aurez la même API, que vous utilisiez React, Vue, ou Svelte.

✅ Cela garantit une expérience **cohérente** et facilite la migration ou l’utilisation multi-frameworks.

***

### 🚀 Avantages côté client

* 🎯 **Cohérence** → la même API disponible sur tous les frameworks.
* ⚡ **Simplicité** → fonctions prêtes à l’emploi (`signIn`, `signOut`, `useSession`, etc.).
* 🔒 **Sécurité** → gestion automatique des sessions et cookies.
* 🧑‍💻 **Flexibilité** → possibilité d’étendre le client avec des plugins (ex: 2FA).

***

👉 En résumé :\
La **librairie client BetterAuth** est votre passerelle entre l’UI et le serveur d’auth.\
Elle fournit des **fonctions universelles et cohérentes**, adaptées à tous les frameworks modernes 🎉.

## ⚡ Installation & Configuration du Client BetterAuth

***

### 📥 Installation du package

Si vous ne l’avez pas encore fait, installez **BetterAuth** dans votre projet frontend :

```bash
# npm
npm i better-auth

# pnpm
pnpm add better-auth

# yarn
yarn add better-auth

# bun
bun add better-auth
```

***

### 🛠️ Création d’une instance client

BetterAuth fournit une fonction **`createAuthClient`** pour initialiser le client dans votre application.\
👉 Importez la version correspondant à votre framework (par ex. **`better-auth/react`** pour React).

#### Exemple (React)

```ts
// lib/auth-client.ts
import { createAuthClient } from "better-auth/react"

export const authClient = createAuthClient({
  baseURL: "http://localhost:3000" // 🌍 URL du serveur d’auth
})
```

***

### 🧐 Explications

* **`baseURL`** → l’URL de votre serveur d’auth.
  * Si le serveur tourne sur **le même domaine que votre client**, vous pouvez **ignorer cette option**.
  *   Si vous utilisez un chemin personnalisé (ex: `/custom-path/auth`), il faut fournir l’URL complète :

      ```ts
      baseURL: "http://localhost:3000/custom-path/auth"
      ```

👉 Cela garantit que toutes les requêtes client sont envoyées vers le bon endpoint du serveur BetterAuth.

***

### 🚀 Utilisation du client

Une fois votre **client initialisé**, vous pouvez l’utiliser pour **interagir avec le serveur BetterAuth**.

Le client fournit déjà un ensemble de fonctions par défaut (inscription, connexion, gestion de session, déconnexion, etc.) et peut être **étendu avec des plugins** (ex: 2FA, passkeys, magic link).

***

#### 📌 Exemple : Connexion avec email + mot de passe

```ts
// auth-client.ts
import { createAuthClient } from "better-auth/client"

const authClient = createAuthClient()

await authClient.signIn.email({
  email: "test@user.com",
  password: "password1234"
})
```

***

### 🎯 Points forts

* ⚡ **Installation rapide** → une seule commande.
* 🌍 **Compatible multi-frameworks** → React, Vue, Svelte, Solid, Vanilla.
* 🔑 **Client extensible** → support natif des plugins (2FA, passkey, etc.).
* 🎨 **API cohérente** → les mêmes méthodes (`signIn`, `signUp`, `signOut`, `useSession`, etc.) quel que soit le framework.

***

👉 En résumé :\
Avec **`createAuthClient`**, vous obtenez une **passerelle simple et universelle** pour connecter votre frontend au serveur BetterAuth.\
Vous pouvez dès lors implémenter **l’inscription, la connexion et la gestion de session en quelques lignes** 🎉.

## 🎣 Hooks du Client BetterAuth

En plus des méthodes classiques (`signIn`, `signOut`, `getSession`…), le client BetterAuth fournit des **hooks réactifs** pour accéder facilement aux données d’authentification.

👉 Tous les hooks sont disponibles depuis l’objet client et commencent par **`use`** (ex: `useSession`).

***

### 📌 Exemple : `useSession`

Ce hook permet d’accéder **en temps réel** à la session utilisateur.

#### Exemple (React)

```tsx
// user.tsx
// Assurez-vous d’utiliser le client React
import { createAuthClient } from "better-auth/react"

const { useSession } = createAuthClient() 

export function User() {
  const {
    data: session,   // 🔐 données de session (utilisateur connecté)
    isPending,       // ⏳ état de chargement
    error,           // ❌ objet erreur (si session invalide ou expirée)
    refetch          // 🔄 permet de recharger manuellement la session
  } = useSession()

  return (
    <div>
      {isPending && <p>Chargement...</p>}
      {error && <p>Erreur : {error.message}</p>}
      {session ? (
        <p>Bienvenue {session.user.name} 🎉</p>
      ) : (
        <p>Utilisateur non connecté</p>
      )}
    </div>
  )
}
```

***

### 🧐 Explications

* **`data: session`** → contient l’objet session (infos utilisateur, rôle, etc.).
* **`isPending`** → vrai tant que la récupération de session est en cours.
* **`error`** → utile pour gérer les cas de session invalide ou expirée.
* **`refetch`** → permet de recharger la session à la demande (ex: après mise à jour profil).

👉 Avantage clé : grâce à la réactivité, toute modification de la session (connexion, déconnexion, expiration) est **automatiquement reflétée dans votre UI** ⚡.

***

### 🌍 Compatibilité

BetterAuth fournit les hooks adaptés à chaque framework :

* ⚛️ **React**
* 🖖 **Vue**
* 🔥 **Svelte**
* 🧊 **Solid**

Peu importe le framework → les hooks gardent la même **API et logique d’utilisation** ✅.

***

### 🚀 Points forts des hooks BetterAuth

* 🎯 **Réactivité intégrée** → session toujours synchronisée avec le serveur.
* ⚡ **API uniforme** → mêmes noms et comportements dans tous les frameworks.
* 🔄 **Refetch manuel possible** → contrôle total quand nécessaire.
* 🧑‍💻 **DX optimale** → gestion simple de l’état `pending`, `error`, `data`.

***

👉 En résumé :\
Les **hooks BetterAuth** facilitent l’accès aux **données d’authentification en temps réel**.\
Parmi eux, **`useSession`** est le plus utilisé pour afficher et surveiller l’état de connexion utilisateur 🎉.

## 🌐 Fetch Options dans le Client BetterAuth

BetterAuth utilise en interne une librairie appelée **Better Fetch** pour effectuer les requêtes vers le serveur d’authentification.

👉 **Better Fetch** est un **wrapper autour de l’API `fetch` native**, mais amélioré :

* 📦 il apporte des options supplémentaires,
* 🔄 il simplifie la gestion des requêtes,
* ⚡ il s’intègre parfaitement avec BetterAuth.

***

### 📌 Définir des options par défaut

Vous pouvez passer des **options fetch par défaut** à votre client BetterAuth lors de sa création via la propriété **`fetchOptions`**.

```ts
// auth-client.ts
import { createAuthClient } from "better-auth/client"

const authClient = createAuthClient({
  fetchOptions: {
    // ✨ toutes les options better-fetch
    credentials: "include",  // exemple
    onRequest: (ctx) => {
      console.log("⏳ Requête envoyée :", ctx)
    },
    onSuccess: (ctx) => {
      console.log("✅ Succès :", ctx.response)
    },
    onError: (ctx) => {
      console.error("❌ Erreur :", ctx.error)
    }
  },
})
```

***

### 📌 Passer des options à une fonction spécifique

Vous pouvez aussi passer des options **directement à une fonction du client**.\
👉 Deux façons possibles :

#### 1. En second argument :

```ts
await authClient.signIn.email(
  {
    email: "email@email.com",
    password: "password1234",
  },
  {
    onSuccess(ctx) {
      console.log("Connexion réussie ✅", ctx.data)
    }
  }
)
```

***

#### 2. Dans l’objet principal via `fetchOptions` :

```ts
await authClient.signIn.email({
  email: "email@email.com",
  password: "password1234",
  fetchOptions: {
    onSuccess(ctx) {
      console.log("Bienvenue 🎉", ctx.data.user.name)
    },
    onError(ctx) {
      alert("Erreur de connexion ❌ : " + ctx.error.message)
    }
  }
})
```

***

### 🧐 Différence entre les deux

* **`createAuthClient({ fetchOptions })`** → définit des **options globales** appliquées à toutes les requêtes du client.
* **`fetchOptions` dans une fonction** → définit des **options locales**, uniquement pour cet appel.

👉 Vous pouvez donc définir un comportement par défaut (global), puis **le surcharger au besoin** pour un appel spécifique.

***

### 🚀 Avantages des Fetch Options

* 🔄 **Callbacks intégrés** : `onRequest`, `onSuccess`, `onError`.
* 🎯 **Flexibilité** : configuration globale ou spécifique.
* 🔒 **Cohérence** : mêmes options disponibles partout.
* ⚡ **DX optimisée** : simplifie la gestion des états de requêtes.

***

👉 En résumé :\
Les **Fetch Options** vous permettent de personnaliser et centraliser la façon dont votre client BetterAuth gère les requêtes, aussi bien au **niveau global** qu’au **niveau d’un appel précis** 🎉.

## ❌ Gestion des erreurs (Handling Errors)

Lorsqu’on utilise les fonctions du **client BetterAuth**, celles-ci renvoient généralement un **objet réponse** structuré avec :

* **`data`** → les données renvoyées en cas de succès ✅
* **`error`** → l’objet erreur si quelque chose a échoué ❌

***

### 📌 Structure de l’objet `error`

Un objet `error` contient toujours :

* **`message`** → description lisible (ex: `"Invalid email or password"`)
* **`status`** → code HTTP (ex: `400`, `401`, `403`, `500`)
* **`statusText`** → texte HTTP associé (ex: `"Unauthorized"`)

***

### 📌 Exemple simple de gestion d’erreur

```ts
// auth-client.ts
const { data, error } = await authClient.signIn.email({
  email: "email@email.com",
  password: "password1234"
})

if (error) {
  console.error("Erreur :", error.message) // ❌ Gérer l’erreur
} else {
  console.log("Connexion réussie 🎉", data)
}
```

***

### 📌 Utiliser `fetchOptions.onError`

Si l’action supporte **`fetchOptions`**, vous pouvez passer un **callback `onError`** pour centraliser la gestion des erreurs.

#### 1️⃣ En second argument :

```ts
await authClient.signIn.email(
  {
    email: "email@email.com",
    password: "password1234",
  },
  {
    onError(ctx) {
      alert("Erreur de connexion ❌ : " + ctx.error.message)
    }
  }
)
```

#### 2️⃣ Dans `fetchOptions` directement :

```ts
await authClient.signIn.email({
  email: "email@email.com",
  password: "password1234",
  fetchOptions: {
    onError(ctx) {
      console.error("Erreur détectée ⚠️", ctx.error)
    }
  }
})
```

***

### 📌 Gestion des erreurs avec les Hooks

Les **hooks** (comme `useSession`) renvoient eux aussi :

* **`error`** → l’erreur s’il y en a une
* **`isPending`** → indique si la requête est encore en cours

```tsx
const { data: session, error, isPending } = useSession()

if (isPending) return <p>Chargement… ⏳</p>
if (error) return <p>Erreur : {error.message} ❌</p>
if (session) return <p>Bienvenue {session.user.name} 🎉</p>
```

***

### 📌 Gestion avancée : `$ERROR_CODES`

Le client BetterAuth expose un objet **`$ERROR_CODES`** contenant **tous les codes d’erreurs renvoyés par le serveur**.\
👉 Utile pour traduire les messages ou personnaliser l’UX.

#### Exemple : gestion multilingue

```ts
const authClient = createAuthClient();

type ErrorTypes = Partial<
  Record<
    keyof typeof authClient.$ERROR_CODES,
    {
      en: string;
      es: string;
    }
  >
>

const errorCodes = {
  USER_ALREADY_EXISTS: {
    en: "User already registered",
    es: "Usuario ya registrado",
  },
} satisfies ErrorTypes;

const getErrorMessage = (code: string, lang: "en" | "es") => {
  if (code in errorCodes) {
    return errorCodes[code as keyof typeof errorCodes][lang];
  }
  return "Unknown error ❓";
};

// Exemple d’utilisation
const { error } = await authClient.signUp.email({
  email: "user@email.com",
  password: "password",
  name: "User",
})

if (error) {
  console.log(getErrorMessage(error.code, "en"))
}
```

***

### 🚀 Bonnes pratiques

* ✅ Toujours vérifier `error` avant d’utiliser `data`.
* ✅ Centraliser la gestion via `fetchOptions.onError` pour plus de cohérence.
* ✅ Exploiter `$ERROR_CODES` pour afficher des **messages clairs et localisés**.
* ✅ Sur le client → afficher des messages conviviaux 🥳 (ne pas exposer d’infos techniques).
* ✅ Sur le serveur → logger les erreurs critiques pour le suivi 🔍.

***

👉 En résumé :\
BetterAuth fournit une gestion d’erreurs **typée, claire et flexible**.\
Que ce soit via `error` dans la réponse, `fetchOptions.onError`, ou via `$ERROR_CODES`, vous gardez un **contrôle total sur l’expérience utilisateur** 🎉.

## 🔌 Plugins (Client Side)

BetterAuth est conçu pour être **extensible**.\
👉 Grâce aux **plugins**, vous pouvez :

* ➕ ajouter de **nouvelles fonctionnalités** au client,
* 🔄 modifier ou enrichir les méthodes existantes.

***

### 📌 Exemple : Plugin **Magic Link**

Le plugin **Magic Link** permet à un utilisateur de se connecter uniquement via son **adresse email** (sans mot de passe).

#### ⚙️ Configuration côté client

```ts
// auth-client.ts
import { createAuthClient } from "better-auth/client"
import { magicLinkClient } from "better-auth/client/plugins"

const authClient = createAuthClient({
  plugins: [
    magicLinkClient() // 👉 activation du plugin Magic Link
  ]
})
```

***

### 📌 Utilisation

Une fois le plugin ajouté, de **nouvelles méthodes** sont disponibles sur l’objet `authClient`.

```ts
// Exemple d’utilisation : connexion par Magic Link
await authClient.signIn.magicLink({
  email: "test@email.com"
})
```

👉 Ici, un email contenant un **lien magique** est envoyé à l’utilisateur.\
En cliquant dessus, il est automatiquement authentifié ✅.

***

### 🧐 Explications

* **Sans plugin** → seules les méthodes natives (`signIn.email`, `signUp.email`, `signOut`, etc.) sont disponibles.
* **Avec plugin** → vous enrichissez le client avec de nouvelles capacités (ex: Magic Link, Passkeys, OTP, 2FA, etc.).

***

### 🚀 Avantages des Plugins

* 🔒 **Auth avancée prête à l’emploi** → OTP, Passkey, Social Login, etc.
* ⚡ **Flexibilité** → activer uniquement les fonctionnalités dont vous avez besoin.
* 🧩 **Extensibilité** → ajouter plusieurs plugins sans casser le cœur de l’auth.
* 🧑‍💻 **DX améliorée** → API claire et homogène, même avec des plugins multiples.

***

👉 En résumé :\
Les **plugins BetterAuth côté client** permettent d’ajouter des **méthodes d’authentification supplémentaires** (ex: `signIn.magicLink`) sans effort.\
Vous partez d’un client de base, puis vous l’**étendez selon vos besoins** 🎉.
