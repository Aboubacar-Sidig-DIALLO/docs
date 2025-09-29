# 🔑 OAuth avec Better Auth

### 1. Built-in OAuth

Better Auth inclut déjà des intégrations prêtes pour les principaux fournisseurs :

* Google
* GitHub
* Apple
* Discord
* Facebook
* etc.

👉 Tu n’as qu’à activer le provider dans `auth.ts` :

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    },
    github: {
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
    }
  }
});
```

***

### 2. Sign-in avec un provider

Côté **client**, tu utilises :

```ts
import { authClient } from "@/lib/auth-client";

await authClient.signIn.social({
  provider: "google",        // ou "github", "facebook", "apple"
  callbackURL: "/dashboard", // redirection après login
  errorCallbackURL: "/error",
  newUserCallbackURL: "/welcome",
  disableRedirect: false     // si true => pas de redirection auto
});
```

***

### 3. OAuth Tokens

Si tu ne veux pas rediriger l’utilisateur, tu peux directement passer un **idToken** ou un **accessToken** obtenu depuis ton provider :

```ts
await authClient.signIn.social({
  provider: "google",
  idToken: "xxx.yyy.zzz", // jeton JWT renvoyé par Google
});
```

***

### 4. Generic OAuth Plugin

Si ton provider n’est **pas supporté nativement**, tu peux utiliser le **Generic OAuth Plugin** pour définir :

* L’URL d’authentification
* L’URL de token
* L’URL du profil utilisateur
* Les scopes nécessaires

Cela permet d’intégrer n’importe quel service OAuth 2.0 / OIDC.

***

## ⚡ En résumé

* **Built-in providers** (Google, GitHub, Apple…) → simples à activer.
* **signIn.social** → la méthode clé pour démarrer l’auth OAuth.
* **Generic OAuth Plugin** → pour tout provider personnalisé.

## ⚙️ **Configurer un Social Provider (ex: Google)**

### 1. Pourquoi ?

Les **social providers** (Google, GitHub, Apple, etc.) permettent à tes utilisateurs de se connecter sans créer de mot de passe → en utilisant leur compte existant.

Better Auth s’appuie sur **OAuth 2.0 / OIDC** pour gérer cela.

***

### 2. Exemple avec **Google**

📌 `auth.ts`

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  // ... autres configurations (db, plugins, etc.)
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    },
  },
});
```

***

### 3. Où trouver le `clientId` et `clientSecret` ?

👉 Tu dois créer un projet dans la **Google Cloud Console** :

1. Va sur Google Cloud Console.
2. Crée un projet ou utilise un existant.
3. Active **"OAuth consent screen"**.
4. Crée des **identifiants OAuth 2.0**.
   * Type : **Web application**
   * URL de redirection : `http://localhost:3000/api/auth/callback/google` (en local)
   * En prod : `https://ton-domaine.com/api/auth/callback/google`
5. Copie le **Client ID** et le **Client Secret** dans ton `.env` :

```env
GOOGLE_CLIENT_ID=xxxxxxxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=xxxxxxxx
```

***

### 4. Côté client : se connecter via Google

📌 `sign-in.ts`

```ts
import { authClient } from "@/lib/auth-client";

await authClient.signIn.social({
  provider: "google",        // identifiant du provider
  callbackURL: "/dashboard", // où rediriger après login
  errorCallbackURL: "/error",
  newUserCallbackURL: "/welcome",
});
```

***

✅ **En résumé :**

* Déclare ton provider dans `auth.ts` avec `clientId` et `clientSecret`.
* Configure tes URLs de redirection dans la console du provider (Google, GitHub, etc.).
* Utilise `authClient.signIn.social({ provider: "xxx" })` côté client pour initier la connexion.

## 🌍 **Usage des Social Providers**

Better Auth te permet de gérer deux cas :

1. **Connexion avec un compte social** (Google, GitHub, etc.)
2. **Lier un compte social à un utilisateur déjà existant**

***

### 🔑 1. **Connexion avec un Social Provider**

#### 📌 Côté Client

```ts
// sign-in.ts
import { authClient } from "@/lib/auth-client";

await authClient.signIn.social({
  provider: "google", // ou "github", "apple", "discord", etc.
  callbackURL: "/dashboard",   // redirection après connexion
  errorCallbackURL: "/error",  // en cas d’erreur
  newUserCallbackURL: "/welcome", // si c’est un nouvel utilisateur
});
```

➡️ Cela redirige l’utilisateur vers la page de login du provider choisi (Google par ex.), puis revient à ton app.

***

#### 📌 Côté Serveur

```ts
import { auth } from "./auth";

await auth.api.signInSocial({
  body: {
    provider: "google", // identifiant du provider
  },
});
```

👉 Utile pour les frameworks server-side (Next.js, Hono, NestJS, etc.), où tu veux gérer la connexion sans passer par `authClient`.

***

### 🔗 2. **Lier un Compte Social à un Utilisateur**

#### 📌 Côté Client

```ts
// Exemple: dans la page profil, l’utilisateur veut lier son compte Google
await authClient.linkSocial({
  provider: "google",
});
```

➡️ Ça associe le compte social à l’utilisateur déjà connecté (par email/mot de passe ou autre).

***

#### 📌 Côté Serveur

```ts
import { auth } from "./auth";

await auth.api.linkSocialAccount({
  body: {
    provider: "google", 
  },
  headers: { 
    // ⚠️ Nécessaire : inclure les headers avec le token/session utilisateur
    cookie: req.headers.cookie, 
    authorization: req.headers.authorization,
  }
});
```

➡️ Ici tu dois passer les **headers d’authentification** pour identifier quel utilisateur doit être lié au provider.

***

### ✅ En résumé

* `signIn.social` → se connecter avec un provider externe.
* `linkSocial` → lier un provider à un compte existant.
* Client (`authClient`) = usage côté navigateur/app.
* Serveur (`auth.api`) = usage côté backend, avec `headers` obligatoires pour identifier l’utilisateur.

## 🔑 **Récupérer un Access Token (Social Provider)**

Better Auth te fournit la méthode **`getAccessToken`** qui fonctionne :

* **côté client** via `authClient`
* **côté serveur** via `auth.api`

👉 Et en plus, **si le token est expiré, Better Auth le rafraîchit automatiquement** pour toi 🎉.

***

### 🖥️ **Côté Client**

```ts
// Exemple : récupérer l’access token Google depuis le navigateur
const { accessToken } = await authClient.getAccessToken({
  providerId: "google",   // ID du provider ("google", "github", "apple", etc.)
  accountId: "accountId", // (optionnel) si l’utilisateur a plusieurs comptes liés
})
```

➡️ Retourne un objet avec la clé **`accessToken`** prête à être utilisée pour des appels API (par ex. requête à Google Calendar API).

***

### ⚙️ **Côté Serveur**

```ts
import { auth } from "./auth";

await auth.api.getAccessToken({
  body: {
    providerId: "google",   // ID du provider
    accountId: "accountId", // (optionnel) identifiant d’un compte spécifique
    userId: "userId",       // (optionnel) si tu n’utilises pas les headers de session
  },
  headers: {
    // ⚠️ Nécessaire si tu veux récupérer le token de l’utilisateur connecté
    cookie: req.headers.cookie,
    authorization: req.headers.authorization,
  }
})
```

➡️ Deux cas possibles :

1. **Avec `headers` d’authentification** → Better Auth identifie automatiquement l’utilisateur connecté.
2. **Avec `userId` explicite** → utile pour des jobs serveur (ex : CRON qui appelle l’API Google pour tous tes utilisateurs).

***

### ✅ Points clés à retenir

* **`providerId` obligatoire** → correspond à ton provider configuré (ex: `"google"`).
* **`accountId` optionnel** → utile si l’utilisateur a lié plusieurs comptes Google, GitHub, etc.
* **Auto-refresh intégré** → pas besoin de gérer toi-même l’expiration du token.
* **Usage client ou serveur** → flexible selon tes besoins.

***

⚡ Exemple concret d’usage :

* L’utilisateur se connecte avec Google → tu récupères un **`accessToken`**.
* Tu utilises ce token pour appeler l’API Google (par ex. récupérer son profil, ses contacts ou son calendrier).
* Si le token est expiré → Better Auth le régénère pour toi ✅.

## 👤 **Récupérer les infos d’un compte social (Account Info)**

Better Auth fournit la méthode **`accountInfo`** :

* **Côté client** → `authClient.accountInfo()`
* **Côté serveur** → `auth.api.accountInfo()`

Cette méthode permet de demander au provider (Google, GitHub, Apple, …) les données du compte **associé à l’`accountId`** (ID fourni par le provider lui-même).

***

### 🖥️ **Côté Client**

```ts
// Exemple : récupérer les infos d’un compte social côté client
const info = await authClient.accountInfo({
  accountId: "accountId", 
  // 👉 L’ID donné par le provider (par ex. sub de Google, id de GitHub, etc.)
})

console.log(info)
/**
 * Exemple possible de retour (selon provider) :
 * {
 *   id: "1234567890",
 *   email: "user@gmail.com",
 *   name: "John Doe",
 *   picture: "https://lh3.googleusercontent.com/a-/AOh14Gh...",
 *   provider: "google"
 * }
 */
```

***

### ⚙️ **Côté Serveur**

```ts
import { auth } from "./auth";

const info = await auth.api.accountInfo({
  body: { accountId: "accountId" }, // ID spécifique renvoyé par le provider
  headers: {
    // ⚠️ Important : inclure les headers de l’utilisateur connecté
    cookie: req.headers.cookie,
    authorization: req.headers.authorization,
  }
})

console.log(info)
```

***

### ✅ Points clés à retenir

* **`accountId` obligatoire** → il identifie le compte social auprès du provider.
* **Provider auto-détecté** → Better Auth sait quel provider utiliser à partir de l’`accountId`.
* **Headers côté serveur** → nécessaires pour authentifier l’appel et associer la requête à l’utilisateur.
* **Utilité pratique** :
  * Récupérer le profil utilisateur complet depuis Google, GitHub, etc.
  * Rafraîchir les infos (photo, nom, email).
  * Vérifier les permissions ou claims d’un compte social.

## 🔐 **Demander des Scopes OAuth Supplémentaires**

### 📝 Pourquoi ?

👉 Lorsqu’un utilisateur se connecte via Google, GitHub ou autre, il n’est pas toujours souhaitable de demander **trop de permissions dès le départ**.\
Exemple :

* Au début → simple connexion avec **email** et **profil**.
* Plus tard → demander un accès **Drive**, **GitHub Repositories**, ou **calendrier**.

Better Auth permet de gérer ça **sans casser la connexion existante** 💡.

***

### 🚀 Comment faire ?

Tu utilises la méthode **`linkSocial`** avec :

* le même `provider` déjà lié (ex. `"google"`, `"github"`)
* un tableau `scopes` contenant les nouvelles permissions demandées

#### Exemple pratique : Google Drive

```ts
const requestAdditionalScopes = async () => {
  await authClient.linkSocial({
    provider: "google",
    scopes: ["https://www.googleapis.com/auth/drive.file"],
  });
};
```

💡 Ici, on redemande à Google l’autorisation d’accéder aux fichiers Drive de l’utilisateur.

***

### ⚠️ Conditions importantes

* ✔️ **Version requise : 1.2.7 ou plus**
  * Les versions antérieures (ex: `1.2.2`) peuvent retourner l’erreur :\
    `"Social account already linked"` 🚫.
* ✔️ L’utilisateur garde **le même compte lié** → pas besoin de recréer un nouveau compte.
* ✔️ Le flux OAuth est relancé, mais cette fois avec les **nouvelles permissions** demandées.

***

### ✅ En résumé

* Tu peux commencer avec **scopes minimum** pour simplifier l’onboarding utilisateur.
* Ensuite, au moment opportun (ex: l’utilisateur veut importer depuis Google Drive), tu appelles **`linkSocial` avec de nouveaux scopes**.
* L’expérience est fluide et garde la **connexion existante** intacte.

## ⚙️ **Options de Configuration d’un Provider Social**

Quand tu ajoutes un provider (Google, GitHub, etc.), tu peux préciser des **options supplémentaires** comme :

***

### 🔎 **1. scope** (permissions demandées)

👉 Le **scope** définit **ce que ton application est autorisée à récupérer** auprès du provider.\
Exemples fréquents :

* `email` → accès à l’adresse email
* `profile` → accès au nom, avatar, etc.
* `openid` → pour OpenID Connect (souvent obligatoire avec certains providers)

#### Exemple Google avec email + profil

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET",
      scope: ["email", "profile"], // ✅ accès email + profil
    },
  },
});
```

💡 Tu peux ajouter d’autres scopes plus avancés (Drive, Calendar, etc.)\
Ex:

```ts
scope: ["email", "profile", "https://www.googleapis.com/auth/drive.file"]
```

***

### 🔎 **2. redirectURI** (callback personnalisé)

👉 Par défaut, Better Auth redirige vers :

```
/api/auth/callback/${providerName}
```

Mais tu peux **forcer une autre URL de callback**, par exemple si tu veux gérer le retour d’authentification sur une page dédiée.

#### Exemple Google avec redirect personnalisé

```ts
export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET",
      redirectURI: "https://your-app.com/auth/callback", // ✅ callback custom
    },
  },
});
```

***

### ✅ Points clés à retenir

* **scope** → définit les permissions demandées dès la connexion (email, profil, accès APIs spécifiques).
* **redirectURI** → permet de personnaliser le chemin de retour après l’authentification.
* Si tu ne définis rien → Better Auth utilise ses valeurs **par défaut** (`email + profile` pour la plupart des providers, et `/api/auth/callback/...` pour la redirection).
* Tu peux combiner **scope minimal** à l’inscription, puis demander des scopes supplémentaires plus tard avec **`linkSocial`** (comme on a vu précédemment).

## ⚙️ **Options avancées des Providers Sociaux**

Lorsque tu configures un provider (Google, Apple, GitHub, etc.), Better Auth te donne des **options de sécurité et de personnalisation supplémentaires** :

***

### 🚫 **1. `disableSignUp`**

* **But** : empêcher les **nouvelles inscriptions** via ce provider.
* **Utilité** : si tu veux autoriser uniquement les **utilisateurs déjà existants** dans ta base de données, mais bloquer la création de nouveaux comptes via ce provider.

#### Exemple

```ts
export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET",
      disableSignUp: true, // 🚫 interdit la création de nouveaux comptes Google
    },
  },
});
```

👉 Cas pratique :

* Tu ouvres Google Sign-In uniquement pour des employés déjà enregistrés.
* Un utilisateur externe ne pourra pas créer de compte via Google.

***

### 🔒 **2. `disableIdTokenSignIn`**

* **But** : empêche la connexion via un **ID Token** (JWT fourni par le provider).
* **Par défaut** : activé chez certains providers (Google, Apple).
* **Utilité** : renforcer la sécurité si tu veux éviter que des tokens soient utilisés directement pour s’authentifier (et forcer le flux OAuth standard).

#### Exemple

```ts
export const auth = betterAuth({
  socialProviders: {
    apple: {
      clientId: "YOUR_APPLE_CLIENT_ID",
      clientSecret: "YOUR_APPLE_CLIENT_SECRET",
      disableIdTokenSignIn: true, // 🚫 interdit l’utilisation de l’ID Token pour login
    },
  },
});
```

👉 Cela oblige l’utilisateur à passer **par le flux complet OAuth** au lieu d’un simple échange de token.

***

### ✅ **3. `verifyIdToken`**

* **But** : te permet de fournir ta **propre logique de vérification** de l’ID Token.
* **Quand utile** :
  * Tu veux ajouter une couche de sécurité (ex : vérifier un claim spécifique comme `hd` pour restreindre à un domaine Google d’entreprise).
  * Tu veux logger, tracer ou appliquer une validation personnalisée.

#### Exemple

```ts
export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET",
      verifyIdToken: async (idToken) => {
        // Exemple : décoder et vérifier un claim particulier
        const payload = decodeJwt(idToken);
        if (payload.hd !== "mycompany.com") {
          throw new Error("Seuls les emails @mycompany.com sont autorisés");
        }
        return true;
      },
    },
  },
});
```

***

### ✅ Résumé visuel

| Option                     | Rôle                                             | Cas d’usage                                               |
| -------------------------- | ------------------------------------------------ | --------------------------------------------------------- |
| **`disableSignUp`**        | Bloque la création de nouveaux comptes           | Limiter l’accès à des utilisateurs déjà existants         |
| **`disableIdTokenSignIn`** | Interdit l’authentification directe via ID Token | Forcer le flux OAuth complet pour + de sécurité           |
| **`verifyIdToken`**        | Permet une vérification personnalisée du token   | Restreindre à un domaine, vérifier des claims spécifiques |

## 👤 **Personnalisation du Profil Utilisateur avec Better Auth**

Better Auth te permet de **contrôler la manière dont les données du provider sont stockées ou mises à jour** dans ta base.\
Deux options importantes :

***

### 🔄 **1. `overrideUserInfoOnSignIn`**

* **Type** : `boolean`
* **Valeur par défaut** : `false`

👉 Détermine si les informations utilisateur doivent être **mises à jour à chaque connexion**.

* `false` → les données stockées en base restent inchangées, même si le provider a modifié le profil (nom, avatar, etc.).
* `true` → à chaque connexion, Better Auth **rafraîchit les données utilisateur** avec les informations les plus récentes du provider.

#### Exemple

```ts
export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET",
      overrideUserInfoOnSignIn: true, // 🔄 met à jour les infos à chaque login
    },
  },
});
```

💡 Cas d’usage :

* Tu veux que l’avatar, le nom ou l’email d’un utilisateur soient **toujours synchronisés** avec Google/GitHub.
* Évite les données obsolètes (ex: un utilisateur qui change d’adresse Gmail ou de photo).

***

### 🛠️ **2. `mapProfileToUser`**

* **Type** : `(profile) => Partial<User>`
* **But** : personnaliser le mapping entre le **profil du provider** et ton **modèle utilisateur interne**.

👉 Très utile si :

* Tu as ajouté des **champs custom** (ex: `firstName`, `lastName`, `locale`).
* Tu veux **changer la structure par défaut** utilisée par Better Auth.

#### Exemple Google

```ts
export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET",
      mapProfileToUser: (profile) => {
        return {
          firstName: profile.given_name,  // 👤 prénom
          lastName: profile.family_name,  // 👤 nom de famille
          locale: profile.locale,         // 🌍 langue préférée
        };
      },
    },
  },
});
```

#### Exemple GitHub

```ts
mapProfileToUser: (profile) => {
  return {
    username: profile.login,      // identifiant GitHub
    bio: profile.bio,             // biographie
    avatarUrl: profile.avatar_url // image de profil
  };
}
```

***

### ✅ Résumé visuel

| Option                         | Rôle                                                      | Cas d’usage                                                       |
| ------------------------------ | --------------------------------------------------------- | ----------------------------------------------------------------- |
| **`overrideUserInfoOnSignIn`** | Mettre à jour ou non les infos utilisateur à chaque login | Garder les infos **synchronisées** avec le provider               |
| **`mapProfileToUser`**         | Mapper le profil provider → modèle utilisateur interne    | Ajouter des **champs custom** (nom, prénom, langue, avatar, etc.) |

## 🔄 **`refreshAccessToken` – Explication**

* **Rôle** : gérer comment obtenir un **nouveau `accessToken`** et éventuellement un **nouveau `refreshToken`** quand l’ancien est expiré.
* **Par défaut** : Better Auth sait déjà rafraîchir les tokens pour les providers supportés.
* **Pourquoi l’utiliser ?**
  * Si tu veux **personnaliser** la manière dont le rafraîchissement est fait.
  * Si tu veux **ajouter une logique métier** (par exemple : logguer chaque refresh, ajouter une métrique, stocker l’historique des tokens).
  * Si le comportement natif ne couvre pas ton besoin spécifique (Google Workspace, API tierce qui demande un format particulier, etc.).

***

## ⚙️ **Structure attendue**

La fonction reçoit en paramètre le **token actuel** et doit renvoyer un objet contenant :

```ts
{
  accessToken: string;   // le nouveau token d’accès
  refreshToken?: string; // le refresh token, si mis à jour
  expiresAt?: number;    // timestamp (optionnel) pour indiquer la nouvelle expiration
}
```

***

## ✅ **Exemple basique avec Google**

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET",

      // Fonction custom de refresh
      refreshAccessToken: async (token) => {
        // Exemple : appel direct à Google OAuth endpoint
        const res = await fetch("https://oauth2.googleapis.com/token", {
          method: "POST",
          headers: { "Content-Type": "application/x-www-form-urlencoded" },
          body: new URLSearchParams({
            client_id: "YOUR_GOOGLE_CLIENT_ID",
            client_secret: "YOUR_GOOGLE_CLIENT_SECRET",
            grant_type: "refresh_token",
            refresh_token: token.refreshToken,
          }),
        });

        const data = await res.json();

        if (!res.ok) {
          throw new Error(data.error || "Failed to refresh token");
        }

        return {
          accessToken: data.access_token,
          refreshToken: data.refresh_token ?? token.refreshToken, // conserve si Google n'en renvoie pas
          expiresAt: Date.now() + data.expires_in * 1000, // nouvelle date d’expiration
        };
      },
    },
  },
});
```

***

## 📌 Points importants

* ⚠️ **Supporté uniquement pour les providers intégrés** → pas dispo pour le plugin OAuth générique.
* ⚡ Si tu ne fournis pas `refreshAccessToken`, Better Auth utilise sa propre implémentation par défaut.
* 🔒 Bonne pratique : toujours prévoir un `try/catch` et retourner le `refreshToken` précédent si le provider n’en renvoie pas de nouveau.
* 🗂️ Tu peux aussi **logger les erreurs** pour suivre les échecs de refresh (utile pour debug ou métriques).

## 🔑 **Résumé `clientKey` pour TikTok**

* **`clientKey`** = identifiant public de ton app (équivalent du `clientId` ailleurs).
* **`clientSecret`** = secret privé de ton app (pareil que pour les autres).
* Utilisés ensemble lors du **flow OAuth 2.0** avec TikTok.

***

## ✅ Exemple configuration TikTok

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    tiktok: {
      clientKey: "YOUR_TIKTOK_CLIENT_KEY",     // ⚡ clé publique (au lieu de clientId)
      clientSecret: "YOUR_TIKTOK_CLIENT_SECRET", // 🔒 secret privé
      scope: ["user.info.basic"], // par ex. accès aux infos basiques du profil
      redirectURI: "https://your-app.com/auth/callback/tiktok",
    },
  },
});
```

***

## 📌 Points à retenir

* TikTok utilise **`clientKey`** au lieu de **`clientId`** → bien mettre la bonne propriété dans ta config.
* Tu récupères ces credentials depuis ton **TikTok Developer Dashboard**.
* Tu peux définir un **`scope`** comme avec les autres providers (ex. `user.info.basic` ou `video.list`).
* `redirectURI` est optionnelle si tu utilises le chemin par défaut `/api/auth/callback/tiktok`.

## 🚀 **Explication de `getUserInfo`**

* Better Auth, par défaut, sait déjà comment appeler les endpoints _user info_ des providers connus (Google, GitHub, Facebook, etc.).
* Mais parfois :
  * tu veux **récupérer plus de champs** que ceux proposés par défaut,
  * ou tu veux **mapper différemment les données** pour coller à ton schéma utilisateur,
  * ou tu utilises un **provider custom** (ou un provider qui change ses API).

Dans ces cas → tu peux **surcharger la logique** avec `getUserInfo`.

***

## ✅ Exemple Google (personnalisé)

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET",

      // 🔑 Custom getUserInfo
      getUserInfo: async (token) => {
        const response = await fetch("https://www.googleapis.com/oauth2/v2/userinfo", {
          headers: {
            Authorization: `Bearer ${token.accessToken}`,
          },
        });

        const profile = await response.json();

        return {
          user: {
            id: profile.id,                // ID unique du user
            name: profile.name,            // Nom complet
            email: profile.email,          // Email
            image: profile.picture,        // Avatar
            emailVerified: profile.verified_email, // Vérif email
          },
          data: profile, // ⚡ données brutes si tu veux les stocker/logguer
        };
      },
    },
  },
});
```

***

## 📌 Structure du retour attendu

Ton `getUserInfo` doit retourner :

```ts
{
  user: {
    id: string;          // identifiant unique côté provider
    name?: string;       // nom complet
    email?: string;      // email de l'utilisateur
    image?: string;      // URL de l'avatar
    emailVerified?: boolean; // true si validé par provider
  },
  data?: any; // (optionnel) données supplémentaires si tu veux stocker/logguer
}
```

***

## 🔥 Use cases intéressants

1. **Récupérer plus de scopes**\
   Exemple : Google Drive → récupérer les fichiers de l’utilisateur après login.
2. **Mapper sur un schéma custom**\
   Exemple : ton modèle `User` contient `firstName` et `lastName` → tu sépares `profile.name`.
3. **Provider pas encore supporté**\
   Exemple : TikTok, Notion, Slack → tu codes ton propre `getUserInfo`.

## 🚀 **`disableImplicitSignUp`**

#### 🔎 Par défaut

Quand un utilisateur se connecte avec Google (ou un autre provider OAuth) **et qu’il n’existe pas encore dans ta base**, Better Auth :

* crée automatiquement un **nouvel utilisateur** (c’est ce qu’on appelle **implicit sign-up**).
* puis il crée une session et le connecte.

***

#### ⚡ Avec `disableImplicitSignUp: true`

* Le comportement **automatique est bloqué**.
* Si l’utilisateur n’existe pas déjà dans la base, la connexion échoue.
* Pour autoriser la création du compte, il faut appeler **explicitement** l’API de sign-in **avec `requestSignUp: true`**.

👉 Ça donne le contrôle complet :

* tu peux exiger un **processus d’onboarding** avant de créer l’utilisateur,
* tu peux restreindre les inscriptions aux personnes **préalablement invitées** ou qui remplissent certaines conditions.

***

## ✅ Exemple d’implémentation

#### `auth.ts`

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET",
      disableImplicitSignUp: true, // 🚫 blocage du sign-up auto
    },
  },
});
```

***

#### Côté **client**

Si tu veux autoriser explicitement l’inscription d’un nouveau user :

```ts
await authClient.signIn.social({
  provider: "google",
  requestSignUp: true, // ✅ forcer la création du compte
});
```

Si tu oublies `requestSignUp: true` et que l’utilisateur n’existe pas →\
➡️ l’appel échoue avec une erreur (`USER_NOT_FOUND` ou similaire).

***

## 📌 Cas d’usage typiques

1. **App entreprise** → seuls les employés invités peuvent créer un compte.
2. **App avec whitelisting** → tu ajoutes une vérification avant d’autoriser la création du user.
3. **Flow custom** → tu obliges l’utilisateur à passer par une étape "Complétez votre profil" avant de réellement créer son compte.

## 🚀 **`prompt` dans OAuth (Google, etc.)**

Le paramètre `prompt` est passé à l’URL d’autorisation OAuth et influence **comment le fournisseur (Google, Facebook, etc.) demande l’authentification ou le consentement à l’utilisateur**.

***

### 🔑 Valeurs possibles

* **`login`**\
  ➝ Force l’utilisateur à se reconnecter même s’il est déjà connecté.\
  (utile pour forcer une authentification fraîche, ex. action sensible comme changer son email).
* **`consent`**\
  ➝ Force la réapparition de l’écran de consentement (re-demande des scopes).\
  (utile si tu as ajouté de nouveaux scopes et veux t’assurer que l’utilisateur les accepte).
* **`select_account`**\
  ➝ Permet à l’utilisateur de **choisir un compte** Google parmi ceux connectés (au lieu de prendre automatiquement le compte actif).\
  (très utile si tes utilisateurs ont plusieurs comptes Google).
* **`none`**\
  ➝ Essaye d’utiliser le compte déjà connecté **sans afficher d’écran**.\
  ⚠️ échoue si une action utilisateur est nécessaire (par exemple pas de compte actif).\
  (utile pour du **silent login**).
* **Combinaisons** (séparées par `+`) :
  * `"select_account+consent"` ➝ oblige à choisir un compte **et** re-demander le consentement.

***

### ✅ Exemple pratique (Google)

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET",
      prompt: "select_account", // 👈 oblige le choix d’un compte
    },
  },
});
```

***

### 📌 Cas d’usage réels

* 🔐 **Sécurité renforcée** :\
  utiliser `login` pour forcer la reconnexion avant une action critique (ex: supprimer le compte).
* 🌐 **Multi-comptes** :\
  utiliser `select_account` pour éviter que Google connecte automatiquement le "mauvais" compte.
* 🔄 **Scopes additionnels** :\
  utiliser `consent` pour forcer le consentement aux nouveaux scopes.
* ⚡ **Silent login** (SSO fluide) :\
  utiliser `none` si tu veux éviter d’afficher un popup si l’utilisateur est déjà connecté.

## 🚀 **`responseMode` en OAuth**

Quand ton app fait une requête d’autorisation, le provider doit renvoyer le **code d’autorisation** (ou le token) à ton application.\
Le paramètre **`responseMode`** détermine **par quel mécanisme** ce code est renvoyé.

***

### 🔑 Valeurs possibles

#### 1. **`query`** (par défaut)

* Le **code** est renvoyé dans l’URL (query string).
*   Exemple :

    ```
    https://your-app.com/callback?code=abc123&state=xyz
    ```
* ✅ Simple, lisible.
* ⚠️ Moins sécurisé car le code apparaît dans l’historique / logs / referrer.

***

#### 2. **`form_post`**

* Le **code** est renvoyé via un **POST** avec un formulaire caché auto-soumis.
*   Exemple :

    ```http
    POST /callback HTTP/1.1
    Content-Type: application/x-www-form-urlencoded

    code=abc123&state=xyz
    ```
* ✅ Plus sécurisé (le code ne transite pas dans l’URL).
* ✅ Recommandé pour éviter les fuites de données dans les logs / referer.
* ⚠️ Implémentation un peu plus complexe (ton endpoint doit accepter POST).

***

### ✅ Exemple avec Better Auth

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET",
      responseMode: "form_post", // 🔒 plus sécurisé que "query"
    },
  },
});
```

***

### 📌 Quand utiliser quoi ?

| Mode        | Description                | Avantages                                     | Inconvénients                      | Quand l’utiliser           |
| ----------- | -------------------------- | --------------------------------------------- | ---------------------------------- | -------------------------- |
| `query`     | Retourne `code` dans l’URL | Simple, compatible partout                    | Risque de fuite (logs, referrer)   | Dev local, cas simples     |
| `form_post` | Retourne `code` en POST    | Sécurité renforcée, pas d’exposition dans URL | Besoin de gérer POST côté callback | Production, apps sensibles |

## 🔑 **`disableDefaultScope`**

Par défaut, Better Auth ajoute automatiquement certains **scopes minimaux** pour que la connexion sociale fonctionne correctement.\
👉 Exemple avec **Google** : `["openid", "email", "profile"]`

Ces scopes permettent à Better Auth d’obtenir :

* l’email de l’utilisateur ✅
* son profil de base (nom, prénom, avatar) ✅
* son identifiant unique (`sub`) ✅

***

### ⚡ Avec `disableDefaultScope: true`

* Les scopes par défaut **ne sont plus inclus automatiquement**.
* Seuls les **scopes que tu définis manuellement** dans `scope: [...]` seront utilisés.
* Utile si tu veux :
  * **Limiter au maximum les permissions demandées** (principe du moindre privilège 🔒).
  * **Personnaliser ton flux** pour ne récupérer que certaines infos.

***

### ✅ Exemple avec Google

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET",
      disableDefaultScope: true, // 🚫 on enlève email/profile/openid
      scope: ["https://www.googleapis.com/auth/userinfo.email"], // 👉 tu choisis uniquement email
    },
  },
});
```

👉 Ici, **Better Auth ne demandera plus le profil complet de l’utilisateur**, seulement son email.\
Cela signifie que dans ton `mapProfileToUser`, tu ne recevras **pas forcément le nom ou l’avatar** (puisque Google ne les retournera pas sans scope `profile`).

***

### 📌 Bonnes pratiques

* 🚫 Évite de mettre `disableDefaultScope: true` sauf si tu sais **exactement** quels scopes tu veux récupérer.
* ✅ Utilise-le dans les cas où :
  * Tu veux une **application stricte** qui ne demande que l’email.
  * Tu gères toi-même le profil utilisateur (nom, avatar…) et tu n’as pas besoin de ceux du provider.

## 🔧 **Other Provider Configurations**

Better Auth fournit un socle commun, mais **chaque provider peut exposer ses propres paramètres** (en fonction de ses API OAuth).

### 🎯 Exemple concret

#### Google

* `scope` → ex: `["email", "profile"]`
* `prompt` → `"select_account" | "consent" | "login"`
* `responseMode` → `"query" | "form_post"`

#### GitHub

* `scope` → ex: `["read:user", "repo"]`
* Pas de `prompt` (propre à Google/Apple)

#### Apple

* `scope` → `["name", "email"]`
* `responseMode` souvent imposé à `"form_post"`

#### TikTok

* Utilise `clientKey` (au lieu de `clientId`)
* Scopes différents (`user.info.basic`, etc.)

***

### 📌 Bonnes pratiques

1. **Toujours vérifier la doc du provider** → car certains exigent des paramètres spécifiques (par ex. Apple impose certaines configurations de redirectURI et responseMode).
2. **Ne pas copier-coller un setup Google pour un autre provider** → leurs APIs OAuth ne sont pas identiques.
3. **Commencer simple** : activer `clientId` + `clientSecret` + `scope: ["email", "profile"]`, puis ajouter des options spécifiques si nécessaire.
