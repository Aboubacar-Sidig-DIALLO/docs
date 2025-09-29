# 📝 Utilisation de base (Basic Usage)

BetterAuth propose nativement un support d’authentification pour :

* 📧 **Email + mot de passe**
* 🌍 **Fournisseurs sociaux** (Google, GitHub, Apple, et bien d’autres)

Mais il peut aussi être **facilement étendu** grâce aux **plugins** ✨, afin d’ajouter d’autres méthodes telles que :

* 👤 **Nom d’utilisateur (username)**
* ✉️ **Lien magique (magic link)**
* 🔑 **Passkey**
* 📮 **OTP par email (email-otp)**
* … et bien plus encore !

***

👉 En résumé : **BetterAuth vous donne les bases solides tout en restant flexible**. Vous pouvez démarrer rapidement avec email ou social login ✅, puis enrichir avec des méthodes modernes via les plugins 🔌.

## 📧 Email & Mot de passe

BetterAuth supporte **nativement** l’authentification avec **email + mot de passe** 🔐.\
Pour l’activer, il suffit de modifier la configuration de votre instance :

***

### ✨ Exemple de configuration

```ts
// auth.ts
import { betterAuth } from "better-auth"

export const auth = betterAuth({
    emailAndPassword: {    
        enabled: true
    } 
})
```

***

👉 Et voilà 🎉 !\
Avec seulement ces quelques lignes, votre application prend déjà en charge l’authentification par **email et mot de passe sécurisé**.

⚡ Vous pourrez ensuite utiliser les méthodes **`signUp`** et **`signIn`** côté client pour enregistrer ou connecter vos utilisateurs.

## 📝 Inscription d’un utilisateur (**Sign Up**)

Avec BetterAuth, l’inscription via **email + mot de passe** est simple et directe ✅.\
Il suffit d’appeler la méthode côté client **`signUp.email`** fournie par `authClient`.

***

### 📌 Exemple d’inscription

```ts
// sign-up.ts
import { authClient } from "@/lib/auth-client"; // Import du client d'authentification

const { data, error } = await authClient.signUp.email(
  {
    email,       // 📧 Adresse email de l’utilisateur
    password,    // 🔐 Mot de passe (min. 8 caractères par défaut)
    name,        // 🧑 Nom ou pseudonyme affiché
    image,       // 🖼️ (Optionnel) URL de l’image de profil
    callbackURL: "/dashboard" // 🌍 (Optionnel) URL de redirection après vérification email
  },
  {
    onRequest: (ctx) => {
      // ⚡ Déclenché lorsque la requête commence
      // 👉 Exemple : afficher un "loader" ou désactiver le bouton
    },
    onSuccess: (ctx) => {
      // ✅ Déclenché quand l’inscription réussit
      // 👉 Exemple : rediriger vers le tableau de bord ou la page de connexion
    },
    onError: (ctx) => {
      // ❌ Déclenché en cas d’erreur
      // 👉 Exemple : afficher un message d’erreur à l’utilisateur
      alert(ctx.error.message);
    },
  }
);
```

***

### 🔄 Comportement par défaut : auto connexion

Par défaut, BetterAuth **connecte automatiquement** l’utilisateur après une inscription réussie 🎉.\
C’est pratique pour la majorité des applications → gain de fluidité UX, pas besoin de refaire un `signIn`.

👉 Mais si tu veux **désactiver cette auto-connexion** et forcer l’utilisateur à se connecter manuellement après son inscription :

```ts
// auth.ts
import { betterAuth } from "better-auth"

export const auth = betterAuth({
  emailAndPassword: {
    enabled: true,
    autoSignIn: false // 🔴 Désactive la connexion auto après inscription
  },
})
```

***

### 🧐 Explications détaillées

* **`email`** → obligatoire. Sert d’identifiant unique.
* **`password`** → obligatoire. Par défaut, BetterAuth impose un **minimum de 8 caractères** (bonne pratique sécurité 🔒). Tu peux renforcer cette règle avec des plugins ou une validation custom.
* **`name`** → optionnel, utile pour l’affichage (ex: tableau de bord, profil).
* **`image`** → optionnel, permet d’associer une image de profil dès l’inscription.
* **`callbackURL`** → optionnel. Très utile quand tu veux que l’utilisateur confirme son adresse email avant d’accéder à certaines parties (redirection après vérification).

***

### 🎯 Exemple de flow utilisateur

1. 👤 L’utilisateur remplit un formulaire d’inscription (email, mot de passe, nom).
2. 📡 `authClient.signUp.email()` est appelé.
3. ⏳ Pendant la requête → `onRequest` s’exécute (ex : afficher un "spinner").
4. ✅ Si l’inscription réussit → `onSuccess` est déclenché (rediriger vers `/dashboard`).
5. ❌ Si une erreur survient (email déjà utilisé, mot de passe trop court, etc.) → `onError` affiche un message clair.

***

### 🚀 Avantages de ce système

* 🔐 **Sécurité intégrée** (validation email/mot de passe, hash côté serveur).
* 🎯 **Flexibilité** (callback, autoSignIn activable/désactivable).
* 🌍 **UX fluide** (l’utilisateur est directement connecté après inscription si tu le souhaites).
* 🧩 **Extensible** (tu peux combiner avec d’autres méthodes : social login, passkey, etc.).

***

👉 En résumé :\
`signUp.email` est la **porte d’entrée standard** pour créer un utilisateur avec email + mot de passe dans BetterAuth.\
Tu peux **contrôler l’expérience utilisateur** (auto connexion, redirection, messages d’erreur) tout en bénéficiant de **bonnes pratiques de sécurité intégrées**.

## 🔑 Connexion d’un utilisateur (**Sign In**)

Pour connecter un utilisateur avec **email + mot de passe**, BetterAuth fournit la méthode **`signIn.email`** côté client.

***

### 📌 Exemple de connexion

```ts
// sign-in.ts
const { data, error } = await authClient.signIn.email(
  {
    /**
     * 📧 Email de l’utilisateur
     */
    email,
    /**
     * 🔐 Mot de passe de l’utilisateur
     */
    password,
    /**
     * 🌍 (Optionnel) URL de redirection après
     * vérification email ou connexion
     */
    callbackURL: "/dashboard",
    /**
     * 🕒 Se souvenir de la session après fermeture du navigateur
     * Valeur par défaut : true
     */
    rememberMe: false
  },
  {
    // Callbacks utiles
    onRequest: (ctx) => {
      // ⚡ Pendant l’envoi de la requête
      // Exemple : afficher un spinner
    },
    onSuccess: (ctx) => {
      // ✅ Si la connexion réussit
      // Exemple : rediriger vers le dashboard
    },
    onError: (ctx) => {
      // ❌ Si une erreur survient
      // Exemple : afficher un message d’erreur
      alert(ctx.error.message);
    }
  }
);
```

***

### ⚠️ Bonnes pratiques

👉 Toujours appeler **`authClient.signIn.email` côté client**.\
🚫 Ne jamais l’invoquer depuis le serveur : ces méthodes sont pensées pour interagir avec l’utilisateur via son navigateur ou application.

***

### 🧐 Explications détaillées des options

* **`email`** → obligatoire. Identifiant unique de l’utilisateur.
* **`password`** → obligatoire. Vérifié côté serveur et comparé au hash stocké en base.
* **`callbackURL`** (optionnel) → redirection après vérification email ou connexion réussie.
* **`rememberMe`** (optionnel, par défaut `true`) :
  * `true` → session persistante (cookie conservé même après fermeture du navigateur).
  * `false` → session supprimée à la fermeture du navigateur (cookie de session).

***

### 🎯 Exemple de flow utilisateur

1. 👤 L’utilisateur saisit son **email et mot de passe**.
2. 📡 `authClient.signIn.email()` est appelé.
3. ⏳ `onRequest` → affichage d’un loader.
4. ✅ `onSuccess` → redirection / tableau de bord / message de bienvenue.
5. ❌ `onError` → gestion et affichage d’un message d’erreur (ex: email inexistant, mot de passe incorrect).

***

### 🔧 Cas pratiques supplémentaires

#### 1. ✅ Connexion persistante (rememberMe: true)

Utile pour les applis **grand public** (réseaux sociaux, e-commerce).

```ts
await authClient.signIn.email({ email, password, rememberMe: true });
```

#### 2. 🔒 Connexion sans persistance (rememberMe: false)

Idéal pour les applis **sensibles** (banque, santé, admin).

```ts
await authClient.signIn.email({ email, password, rememberMe: false });
```

#### 3. 🔀 Redirection après connexion (callbackURL)

```ts
await authClient.signIn.email({
  email,
  password,
  callbackURL: "/profile"
});
```

***

### 🚀 Points forts de BetterAuth pour la connexion

* 🔒 **Sécurité intégrée** (hashing, validation, gestion des sessions).
* 🧑‍💻 **Callbacks flexibles** pour une UX personnalisée.
* 🕒 **Gestion simple des sessions persistantes** (`rememberMe`).
* 🌍 **Support natif des redirections** (après vérification ou social login).

***

👉 **En résumé** :\
`signIn.email` est la méthode standard pour connecter vos utilisateurs via **email + mot de passe**.\
Elle est **sécurisée**, **facile à utiliser** et **hautement personnalisable** 🎉.

## 🖥️ Authentification côté serveur (**Server-Side Authentication**)

BetterAuth ne se limite pas au **côté client** 🌍.\
Vous pouvez aussi gérer l’**authentification côté serveur** grâce aux méthodes fournies par **`auth.api`**.

***

### 📌 Exemple d’utilisation côté serveur

```ts
// server.ts
import { auth } from "./auth"; // chemin vers votre instance BetterAuth serveur

const response = await auth.api.signInEmail({
  body: {
    email,    // 📧 Email de l’utilisateur
    password, // 🔐 Mot de passe de l’utilisateur
  },
  asResponse: true // retourne un objet Response plutôt que les seules données
});
```

***

### 🧐 Explications

* **`auth.api.signInEmail`** → méthode permettant de gérer une connexion **email + mot de passe** côté serveur.
* **`body`** → contient les informations envoyées par le client (ici `email` + `password`).
* **`asResponse: true`** →
  * si défini à `true`, retourne directement un **objet `Response`** (idéal pour frameworks modernes qui exploitent les API Response).
  * sinon, retourne uniquement les **données utilisateur/session**.

***

### ⚠️ Gestion des cookies

L’authentification via BetterAuth repose sur des **cookies sécurisés** 🍪.

* Si votre serveur **ne peut pas retourner directement un objet `Response`**, vous devrez :
  1. **parser manuellement** la réponse,
  2. puis **définir les cookies** d’authentification dans votre serveur.

👉 Exemple : Express ou serveurs Node.js classiques.

***

### ⚡ Cas particulier : Next.js

Pour **Next.js** (et d’autres frameworks modernes similaires), BetterAuth fournit **un plugin intégré** 🎉 qui gère automatiquement :

* ✅ la création et gestion des cookies,
* ✅ la compatibilité avec l’API Route Handlers (`app/api/`),
* ✅ une intégration fluide avec le système d’auth.

***

### 🚀 Points forts côté serveur

* 🔒 Authentification **sécurisée et centralisée** côté backend.
* 🍪 Gestion automatique ou manuelle des **cookies d’authentification**.
* 🌍 Plugins disponibles pour simplifier l’intégration avec les **frameworks populaires** (Next.js, etc.).

***

👉 En résumé :\
BetterAuth permet une **authentification serveur robuste et flexible**.\
Selon votre framework, vous pouvez soit **utiliser la réponse standard**, soit **gérer les cookies manuellement**, soit profiter d’un **plugin officiel** qui fait tout pour vous.

## 🌍 Connexion sociale (**Social Sign-On**)

BetterAuth prend en charge **plusieurs fournisseurs de connexion sociale** 🔑 :

* 🟢 **Google**
* 🐙 **GitHub**
* 🍏 **Apple**
* 🎮 **Discord**
* … et bien d’autres encore !

Ces options permettent à vos utilisateurs de **se connecter rapidement** sans avoir à créer un mot de passe classique.

***

### 📌 Configuration d’un fournisseur social

Pour activer un fournisseur social, configurez-le dans la clé **`socialProviders`** de votre objet `auth`.

#### Exemple avec GitHub 👇

```ts
// auth.ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: { 
    github: { 
      clientId: process.env.GITHUB_CLIENT_ID!, 
      clientSecret: process.env.GITHUB_CLIENT_SECRET!, 
    } 
  }, 
});
```

***

### 🧐 Explications

* **`clientId`** → fourni par le fournisseur (Google, GitHub, etc.) lors de la configuration de l’application OAuth.
* **`clientSecret`** → clé secrète associée à votre application OAuth.
* Vous devez définir ces valeurs via des **variables d’environnement** (`.env`) pour des raisons de sécurité 🔐.

👉 Exemple `.env` :

```env
GITHUB_CLIENT_ID=xxxxxx
GITHUB_CLIENT_SECRET=yyyyyy
```

***

### 🎯 Flow utilisateur typique

1. 👤 L’utilisateur clique sur “Se connecter avec GitHub” (ou Google, Apple, etc.).
2. 🌍 Il est redirigé vers la page de connexion du fournisseur.
3. ✅ Après validation, l’utilisateur est redirigé vers votre app avec un **jeton d’accès**.
4. 🍪 BetterAuth gère automatiquement la **création de session sécurisée** dans votre base de données.

***

### 🚀 Avantages de l’auth sociale avec BetterAuth

* ✨ Support intégré pour plusieurs fournisseurs (OAuth2/OIDC).
* 🔒 Sécurité gérée automatiquement (tokens, sessions, cookies).
* 🎯 Expérience utilisateur fluide (connexion en 1 clic).
* 🧩 Extensible → vous pouvez ajouter d’autres fournisseurs si besoin.

***

👉 En résumé :\
BetterAuth rend la **connexion sociale simple et sécurisée**.\
Vous n’avez qu’à **ajouter vos credentials OAuth** dans la config, et vos utilisateurs peuvent se connecter via leurs comptes existants 🎉.

## 🌍 Connexion avec un fournisseur social (**Sign In with Social Providers**)

Pour connecter un utilisateur via un **fournisseur social** (Google, GitHub, Apple, etc.), BetterAuth fournit la méthode **`signIn.social`** côté client.

***

### 📌 Exemple de connexion avec GitHub

```ts
// sign-in.ts
import { authClient } from "@/lib/auth-client"; // import du client d'auth

await authClient.signIn.social({
  /**
   * 🆔 Identifiant du fournisseur social
   * @example "github", "google", "apple"
   */
  provider: "github",

  /**
   * 🌍 URL de redirection après connexion réussie
   * @default "/"
   */
  callbackURL: "/dashboard",

  /**
   * ❌ URL de redirection si une erreur survient
   */
  errorCallbackURL: "/error",

  /**
   * 👋 URL de redirection si l’utilisateur est nouvellement inscrit
   */
  newUserCallbackURL: "/welcome",

  /**
   * 🚫 Désactiver la redirection automatique vers le fournisseur
   * @default false
   */
  disableRedirect: true,
});
```

***

### 🧐 Explications des options

* **`provider`** → identifiant du fournisseur social (`"google"`, `"github"`, `"apple"`, `"discord"`, etc.).
* **`callbackURL`** → URL de redirection après une connexion réussie (ex : `/dashboard`).
* **`errorCallbackURL`** → URL en cas d’erreur pendant le processus (ex : `/error`).
* **`newUserCallbackURL`** → URL vers laquelle rediriger si l’utilisateur est créé pour la première fois (ex : `/welcome`).
* **`disableRedirect`** →
  * `false` (par défaut) → BetterAuth redirige automatiquement l’utilisateur vers le fournisseur (Google, GitHub, etc.).
  * `true` → désactive cette redirection, utile si vous voulez gérer le flow manuellement.

***

### 🔄 Autres méthodes d’authentification sociale

Au lieu de rediriger l’utilisateur vers le site du fournisseur, vous pouvez aussi l’authentifier directement avec :

* **`idToken`** → jeton d’identité renvoyé par le fournisseur.
* **`accessToken`** → jeton d’accès OAuth2 renvoyé par le fournisseur.

👉 Cela permet par exemple de gérer des intégrations **mobile-first** (React Native, Expo) où la redirection OAuth est plus complexe.

***

### 🎯 Flow utilisateur typique (avec redirection)

1. 👤 L’utilisateur clique sur "Se connecter avec GitHub".
2. 🌍 Redirection vers la page OAuth de GitHub.
3. ✅ Après autorisation, retour dans votre app via `callbackURL`.
4. 🆕 Si c’est un nouvel utilisateur → redirection vers `newUserCallbackURL`.
5. ❌ Si une erreur survient (refus OAuth, credentials invalides) → redirection vers `errorCallbackURL`.

***

### 🚀 Avantages

* 🔒 Gestion sécurisée de l’OAuth (tokens, sessions).
* 🌍 Compatible avec plusieurs providers en même temps.
* 👋 Support natif pour les nouveaux utilisateurs.
* 📱 Flexibilité : redirection classique ou login par **tokens**.

***

👉 En résumé :\
Avec **`signIn.social`**, BetterAuth rend la connexion via **fournisseurs sociaux** (Google, GitHub, Apple, etc.) **simple, sécurisée et personnalisable** 🎉.

## 🚪 Déconnexion d’un utilisateur (**Sign Out**)

BetterAuth fournit une méthode **simple et sécurisée** pour déconnecter un utilisateur : **`signOut`**.

***

### 📌 Exemple de déconnexion simple

```tsx
// user-card.tsx
await authClient.signOut();
```

👉 Cette commande supprime la **session utilisateur active** et nettoie les **cookies d’authentification** 🍪.

***

### 📌 Exemple avec redirection après déconnexion

Vous pouvez personnaliser le comportement après la déconnexion, par exemple **rediriger l’utilisateur vers la page de connexion** :

```tsx
// user-card.tsx
await authClient.signOut({
  fetchOptions: {
    onSuccess: () => {
      router.push("/login"); // redirige vers la page login
    },
  },
});
```

***

### 🧐 Explications

* **`signOut()`** → déconnecte l’utilisateur actuel.
* **`fetchOptions`** → objet permettant de personnaliser le comportement :
  * **`onSuccess`** → callback déclenché après une déconnexion réussie.
  * Vous pouvez y mettre une **redirection**, un **toast de notification**, ou un **refresh du state**.

***

### 🎯 Exemple de flow utilisateur

1. 👤 L’utilisateur clique sur "Déconnexion".
2. 📡 `authClient.signOut()` est exécuté.
3. 🍪 La session et les cookies sont invalidés côté serveur.
4. ✅ `onSuccess` est déclenché (rediriger, afficher un message, etc.).

***

### 🚀 Points forts

* 🔒 **Sécurité intégrée** : les cookies de session sont invalidés proprement.
* 🧑‍💻 **Simplicité** : une seule fonction suffit pour déconnecter l’utilisateur.
* 🎨 **Personnalisable** : possibilité d’ajouter des callbacks (`onSuccess`, `onError`) pour adapter l’expérience utilisateur.

***

👉 En résumé :\
Avec **`signOut`**, la déconnexion est **facile, sécurisée et flexible** 🎉.\
Que ce soit pour un simple logout ou une redirection vers `/login`, tout est déjà prévu ✅.

## 🔐 Gestion de la session (**Session Management**)

Une fois qu’un utilisateur est **connecté**, il est essentiel de pouvoir accéder à sa **session**.\
BetterAuth facilite cet accès aussi bien **côté client** que **côté serveur** ✅.

***

### 🖥️ Côté client

#### 🎣 Hook `useSession`

BetterAuth fournit un hook pratique : **`useSession`**, qui permet d’accéder facilement aux données de session dans vos composants.

👉 Ce hook est basé sur **Nanostores** (une librairie de state management ultra-légère ⚡), ce qui garantit que **toute modification de la session** (comme une déconnexion) est **immédiatement reflétée dans l’UI**.

BetterAuth propose ce hook pour :

* ⚛️ **React**
* 🖖 **Vue**
* 🔥 **Svelte**
* 🧊 **Solid**
* 🌍 **Vanilla JS**

***

#### 📌 Exemple avec React

```tsx
// user.tsx
import { authClient } from "@/lib/auth-client"; // import du client d’auth

export function User() {
  const { 
    data: session,   // 🔐 Les données de session (user, token, etc.)
    isPending,       // ⏳ État de chargement
    error,           // ❌ Erreur éventuelle
    refetch          // 🔄 Permet de refetch la session manuellement
  } = authClient.useSession();

  return (
    <div>
      {isPending && <p>Chargement...</p>}
      {error && <p>Erreur : {error.message}</p>}
      {session ? (
        <p>Bienvenue {session.user.name} 🎉</p>
      ) : (
        <p>Non connecté</p>
      )}
    </div>
  );
}
```

***

### 🧐 Explications

* **`data: session`** → contient toutes les informations liées à l’utilisateur connecté (`user`, `id`, `email`, `roles`, etc.).
* **`isPending`** → indique si la session est en cours de récupération (chargement).
* **`error`** → permet de gérer les erreurs éventuelles (ex: cookie invalide, session expirée).
* **`refetch`** → utile pour recharger la session manuellement (par ex. après mise à jour du profil).

***

### 🎯 Exemple de flow typique côté client

1. 👤 L’utilisateur se connecte (`signIn.email` ou `signIn.social`).
2. 🔐 La session est créée côté serveur et stockée via cookie sécurisé.
3. ⚛️ `useSession` met automatiquement à jour le **state client**.
4. 🎨 L’UI réagit immédiatement → affichage du nom, avatar, boutons spécifiques aux utilisateurs connectés.
5. 🚪 Si l’utilisateur se déconnecte (`signOut`), la session est détruite → `useSession` passe à `null`.

***

### 🚀 Points forts

* ⚡ **Temps réel** : toute modification de session est répercutée automatiquement.
* 🎨 **Expérience utilisateur fluide** : pas besoin de recharger la page pour voir l’état de connexion.
* 🧑‍💻 **API simple** : un seul hook pour gérer toutes les situations (login, logout, session expirée).
* 🌍 **Multi-frameworks** : React, Vue, Svelte, Solid, Vanilla.

***

👉 En résumé :\
Avec **`useSession`**, BetterAuth offre une **gestion de session simple, réactive et universelle**.\
Vous pouvez facilement afficher l’état de connexion de l’utilisateur et réagir en temps réel aux changements 🎉.

## 🔐 Récupération de la session (**Get Session**)

En plus du hook **`useSession`**, BetterAuth propose la méthode **`getSession`** pour récupérer directement la session côté client.\
C’est une alternative pratique si vous ne souhaitez pas utiliser de hook.

***

### 📌 Exemple d’utilisation simple

```tsx
// user.tsx
import { authClient } from "@/lib/auth-client"; // import du client

const { data: session, error } = await authClient.getSession();

if (error) {
  console.error("Erreur lors de la récupération de la session :", error);
}

if (session) {
  console.log("Utilisateur connecté :", session.user.name);
} else {
  console.log("Aucun utilisateur connecté.");
}
```

***

### 🧐 Explications

* **`getSession()`** → méthode asynchrone qui renvoie :
  * **`data: session`** → l’objet session (utilisateur, token, etc.) si connecté.
  * **`error`** → une erreur si la session est invalide ou expirée.

👉 Contrairement à **`useSession`** :

* `getSession()` ne réagit pas automatiquement aux changements de session.
* C’est un simple appel ponctuel (fetch).
* Idéal si vous ne voulez pas gérer un state réactif en continu.

***

### ⚡ Intégration avec des librairies de data-fetching

BetterAuth est compatible avec des solutions comme **TanStack Query** 🌀.\
Vous pouvez donc l’utiliser avec un **query hook** pour profiter du cache, du refetch et du state management.

#### Exemple avec TanStack Query

```tsx
import { useQuery } from "@tanstack/react-query";
import { authClient } from "@/lib/auth-client";

export function User() {
  const { data: session, error, isLoading } = useQuery({
    queryKey: ["session"],
    queryFn: () => authClient.getSession().then(res => res.data),
  });

  if (isLoading) return <p>Chargement...</p>;
  if (error) return <p>Erreur : {error.message}</p>;

  return session ? (
    <p>Bienvenue {session.user.name} 🎉</p>
  ) : (
    <p>Non connecté</p>
  );
}
```

***

### 🚀 Points forts

* 🧑‍💻 **Alternative flexible** → pas obligé d’utiliser `useSession`.
* 🔄 **Intégration simple avec TanStack Query** ou d’autres librairies de data fetching.
* 🎨 **Idéal pour SSR ou client-side fetch ponctuel**.
* ⚡ Moins “réactif” que `useSession`, mais plus léger si vous n’avez pas besoin de mise à jour en temps réel.

***

👉 En résumé :\
`getSession()` est parfait si vous voulez **récupérer la session utilisateur de manière ponctuelle**, ou l’intégrer dans des workflows de **data-fetching (TanStack Query, SWR, etc.)**.\
Pour une UI réactive en temps réel, préférez plutôt **`useSession`** 🎉.

## 🖥️ Gestion de la session côté serveur (**Server-Side Session**)

BetterAuth ne se limite pas au **côté client** : vous pouvez aussi accéder à la **session utilisateur côté serveur** 🔐.\
C’est indispensable pour protéger certaines routes, vérifier les permissions ou afficher des données sécurisées.

***

### 📌 Utilisation de base

Le serveur expose une méthode **`auth.api.getSession`** qui permet de récupérer la session en cours.\
⚠️ Elle nécessite que vous passiez l’objet **`headers`** de la requête en paramètre.

```ts
// server.ts
import { auth } from "./auth"; // chemin vers votre instance BetterAuth
import { headers } from "next/headers";

const session = await auth.api.getSession({
  headers: await headers(), // les headers doivent être fournis
});
```

***

### 🧐 Explications

* **`auth.api.getSession()`** → méthode qui retourne la **session active** si elle existe.
* **`headers`** → nécessaires pour :
  * récupérer les **cookies sécurisés** contenant l’ID de session,
  * valider l’authentification côté serveur.
* **`session`** → contient toutes les infos liées à l’utilisateur connecté (`user`, `roles`, `permissions`, etc.).

***

### 🌍 Exemple avec des frameworks populaires

BetterAuth supporte les environnements modernes :

* ⚛️ **Next.js** → via `next/headers`
* 🖖 **Nuxt**
* 🔥 **SvelteKit**
* 🌌 **Astro**
* ⚡ **Hono**
* 🌀 **TanStack Start**

Dans chacun de ces frameworks, vous devrez **extraire les headers** de la requête entrante et les transmettre à **`getSession`**.

***

### 🎯 Cas d’usage typiques côté serveur

* 🔒 **Protéger une API route** : vérifier que l’utilisateur est connecté avant de répondre.
* 🏢 **Contrôle d’accès** : s’assurer que l’utilisateur a les bons rôles/permissions.
* 🎨 **SSR** (server-side rendering) : injecter les infos de session dans vos pages générées côté serveur.

***

### 🚀 Points forts

* 🔐 **Sécurité** : validation de session côté serveur = impossible de falsifier côté client.
* 🌍 **Compatibilité multi-frameworks** : Next.js, Nuxt, Svelte, Astro, etc.
* 🎯 **Simplicité** : une seule méthode pour récupérer la session (`getSession`).

***

👉 En résumé :\
Avec **`auth.api.getSession`**, BetterAuth permet de **sécuriser vos routes et rendre vos pages dynamiques côté serveur**, en s’appuyant sur les **headers** de la requête 🎉.

## 🧩 Utiliser les plugins (**Using Plugins**)

L’une des grandes forces de **BetterAuth** est son **écosystème de plugins** ✨.\
Il permet d’ajouter des fonctionnalités avancées d’authentification avec seulement **quelques lignes de code** 🚀.

***

### 🔐 Exemple : ajouter l’authentification à deux facteurs (2FA)

#### ⚙️ Configuration côté serveur

Pour utiliser un plugin, vous devez :

1. L’importer,
2. L’ajouter dans la clé **`plugins`** de votre instance `auth`.

👉 Exemple avec le plugin **Two Factor Authentication** :

```ts
// auth.ts
import { betterAuth } from "better-auth"
import { twoFactor } from "better-auth/plugins"

export const auth = betterAuth({
  // ... vos autres options
  plugins: [ 
    twoFactor() 
  ] 
})
```

✅ Après ça, toutes les routes et méthodes liées au 2FA seront disponibles côté serveur.

***

#### 🗄️ Migration de la base de données

Une fois le plugin ajouté, il faut créer les **tables nécessaires** en base.

**Générer le schéma :**

```bash
npx @better-auth/cli generate
```

**Appliquer directement la migration :**

```bash
npx @better-auth/cli migrate
```

👉 Vous pouvez aussi ajouter le schéma manuellement en consultant la **documentation du plugin 2FA**.

***

### 🖥️ Configuration côté client

Après le serveur, on configure aussi le **client** pour gérer le plugin.

👉 Exemple avec le plugin **Two Factor Authentication** côté client :

```ts
// auth-client.ts
import { createAuthClient } from "better-auth/client";
import { twoFactorClient } from "better-auth/client/plugins"; 

const authClient = createAuthClient({
  plugins: [ 
    twoFactorClient({ 
      twoFactorPage: "/two-factor" // page où rediriger l’utilisateur pour vérifier son 2ème facteur
    }) 
  ] 
})
```

✅ Après ça, toutes les méthodes liées au 2FA seront disponibles côté client.

***

### 📌 Exemple d’utilisation côté client

```ts
// profile.ts
import { authClient } from "./auth-client"

// Activer le 2FA
const enableTwoFactor = async () => {
  const data = await authClient.twoFactor.enable({
    password // le mot de passe de l’utilisateur est requis
  })
}

// Désactiver le 2FA
const disableTwoFactor = async () => {
  const data = await authClient.twoFactor.disable({
    password // le mot de passe de l’utilisateur est requis
  })
}

// Se connecter avec email + mot de passe + 2FA
const signInWith2Factor = async () => {
  const data = await authClient.signIn.email({
    // ...
  })
  // 👉 Si l’utilisateur a activé le 2FA, il sera redirigé vers la page définie (ex: /two-factor)
}

// Vérifier le code TOTP
const verifyTOTP = async () => {
  const data = await authClient.twoFactor.verifyTOTP({
    code: "123456", // le code saisi par l’utilisateur
    /**
     * Si le device est marqué comme "de confiance",
     * l’utilisateur n’aura pas à refaire le 2FA sur ce même appareil.
     */
    trustDevice: true
  })
}
```

***

### 🎯 Points clés à retenir

* 🧩 Les **plugins** ajoutent facilement des fonctionnalités complexes (2FA, passkeys, magic link, etc.).
* 🔐 Le plugin **Two Factor** permet d’ajouter un niveau de sécurité supplémentaire.
* 🗄️ Une **migration de la base** est nécessaire après ajout du plugin.
* 🖥️ La configuration client est aussi essentielle (page de redirection 2FA, méthodes disponibles).
* 📱 Vous pouvez ensuite gérer activation, désactivation, vérification TOTP, et login avec 2FA.
