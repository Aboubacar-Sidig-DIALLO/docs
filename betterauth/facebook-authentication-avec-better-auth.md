# 📘 Facebook Authentication avec Better Auth

### 🔑 1. Obtenir vos credentials Facebook

Pour utiliser **Facebook Sign In**, vous devez créer une application sur le **Facebook Developer Portal** :

1. Rendez-vous sur https://developers.facebook.com.
2. Sélectionnez ou créez une application.
3. Naviguez vers **App Settings > Basic**.
4. Récupérez :
   * **App ID** → `clientId`
   * **App Secret** → `clientSecret`

⚠️ **Ne jamais exposer `clientSecret` côté client (frontend)**. Gardez-le uniquement côté serveur.

➡️ Configurez l’URL de redirection :

* En **local** : `http://localhost:3000/api/auth/callback/facebook`
* En **production** : `https://votre-domaine.com/api/auth/callback/facebook`

Si vous avez modifié le chemin de vos routes d’auth (`basePath`), adaptez l’URL de redirection.

***

### ⚙️ 2. Configurer Facebook dans Better Auth

```ts
// auth.ts
import { betterAuth } from "better-auth"

export const auth = betterAuth({
  socialProviders: {
    facebook: {
      clientId: process.env.FACEBOOK_CLIENT_ID as string,
      clientSecret: process.env.FACEBOOK_CLIENT_SECRET as string,
    },
  },
})
```

***

### 🏢 3. Facebook Login for Business (optionnel)

Better Auth supporte également **Facebook Login for Business**.

* Ajoutez un **configId** dans la configuration :

```ts
facebook: {
  clientId: process.env.FACEBOOK_CLIENT_ID as string,
  clientSecret: process.env.FACEBOOK_CLIENT_SECRET as string,
  configId: process.env.FACEBOOK_CONFIG_ID as string, // requis pour Facebook Business
}
```

⚠️ Votre app doit être une **Business App**, et la configuration doit être de type **User access token** (les **System-user access tokens** ne sont pas supportés).

***

### 🚀 4. Sign In avec Facebook

👉 Côté client, utilisez `signIn.social` :

```ts
// auth-client.ts
import { createAuthClient } from "better-auth/client"

const authClient = createAuthClient()

const signIn = async () => {
  const data = await authClient.signIn.social({
    provider: "facebook"
  })
}
```

## ⚙️ Facebook – Configuration Avancée avec Better Auth

### 🔐 1. Ajouter des **scopes** personnalisés

Par défaut, Facebook fournit uniquement les infos de base (`id`, `name`, `email`, `picture`).\
Si vous voulez plus de permissions (ex: récupérer les amis, publications, etc.), ajoutez l’option `scopes`.

👉 Exemple :

```ts
// auth.ts
import { betterAuth } from "better-auth"

export const auth = betterAuth({
  socialProviders: {
    facebook: {
      clientId: process.env.FACEBOOK_CLIENT_ID as string,
      clientSecret: process.env.FACEBOOK_CLIENT_SECRET as string,
      scopes: ["email", "public_profile", "user_friends"], // remplace la liste par défaut
      fields: ["user_friends"], // ajoute des champs supplémentaires
    },
  },
})
```

***

### 📌 2. Détails des options

* **scopes**\
  → Définit les **permissions** demandées lors du login.
  * ⚡ Valeur par défaut : `["email", "public_profile"]`
  * Exemple d’extensions possibles :
    * `"user_friends"` → accès aux amis Facebook
    * `"user_posts"` → accès aux publications
    * `"pages_show_list"` → accès aux pages gérées
* **fields**\
  → Définit les **données du profil utilisateur** que vous voulez récupérer dans la réponse.
  * ⚡ Valeur par défaut : `["id", "name", "email", "picture"]`
  * Exemple d’extensions possibles :
    * `"user_friends"` → récupère la liste des amis
    * `"birthday"` → date de naissance
    * `"gender"` → genre

***

### ✅ Exemple Complet

```ts
// auth.ts
export const auth = betterAuth({
  socialProviders: {
    facebook: {
      clientId: process.env.FACEBOOK_CLIENT_ID as string,
      clientSecret: process.env.FACEBOOK_CLIENT_SECRET as string,
      scopes: ["email", "public_profile", "user_friends", "user_posts"],
      fields: ["id", "name", "email", "picture", "user_friends", "birthday"],
    },
  },
})
```

Avec cette config :

* L’app demandera explicitement à l’utilisateur d’accorder l’accès aux **amis** et aux **publications**.
* Le profil récupéré inclura `id`, `name`, `email`, `picture`, `user_friends` et `birthday`.

## 🔑 Facebook – Sign In avec **ID Token** ou **Access Token**

Better Auth permet de se connecter à Facebook **sans redirection** si vous avez déjà un **ID Token** ou un **Access Token** (utile pour les applis mobiles ou des SDK natifs).

***

### 🚀 Exemple – Connexion avec ID Token ou Access Token

```ts
// auth-client.ts
import { authClient } from "./auth-client"

const data = await authClient.signIn.social({
  provider: "facebook",
  idToken: {
    ...(platform === "ios"
      ? { token: idToken } // iOS → utiliser le vrai ID Token
      : { token: accessToken, accessToken: accessToken }), // Android / autres → fournir accessToken aussi
  },
})
```

***

### 📌 Explications

* `provider: "facebook"` → précise qu’on utilise Facebook comme fournisseur.
* `idToken.token` → requis pour **Facebook Limited Login (iOS)**.
* `idToken.token + idToken.accessToken` → requis si vous n’avez que l’**accessToken** (⚠️ voir issue [#1183](https://github.com/better-auth/better-auth/issues/1183)).
* Aucun **redirect** → l’utilisateur est directement signé côté serveur.

***

### ✅ Cas d’usage typiques

* 🔹 **App mobile (iOS / Android)** utilisant le **Facebook SDK natif**.
* 🔹 **Auth hybride** : vous récupérez l’`idToken` ou `accessToken` via une API externe et vous validez l’identité via Better Auth.
* 🔹 **Connexion silencieuse** : l’utilisateur est déjà connecté via Facebook (native login) et vous synchronisez l’accès avec votre backend.

## 📖 Permissions Reference – **Meta Technologies APIs (Facebook, Instagram, etc.)**

Les **permissions** chez Meta (Facebook / Instagram / Business Manager) déterminent quelles données et quelles actions votre app peut réaliser via l’API Graph. Elles sont **granulaires** et doivent être demandées **uniquement si nécessaires**.

***

### ✅ Règles Générales

* 🎯 **Principe du minimum** : demandez seulement les permissions nécessaires → éviter les rejets en App Review.
* 📝 **App Review** : obligatoire si vous accédez à des données **que vous ne possédez pas** (amis, pages, groupes, etc.).
* 🏢 **Business Verification** : nécessaire pour **Advanced Access** (pubs, audiences, insights business).
* 🔄 **Expiration** : si un utilisateur n’utilise pas votre app pendant **90 jours**, il doit **revalider** les permissions.
* 📊 **Usage autorisé** : vous pouvez utiliser les permissions pour :
  * fournir les fonctionnalités promises,
  * analyser et améliorer votre app,
  * marketing et pub, **uniquement via données anonymisées et agrégées**.

***

### 🚪 Manières de demander une permission

1. **Facebook Login** → classique, email + profil + permissions.
2. **Facebook Login for Business** → accès aux données business (pages, ads, etc.).
3. **Instagram API avec Facebook Login for Business** → pour gérer Instagram Business Accounts.
4. **Business Login for Instagram** → similaire mais orienté business/entreprise.
5. **Meta Business Manager** → permissions de gestion des assets business.

***

### 🔑 Gestion des Permissions

* **Accorder** : lors du login, l’utilisateur choisit d’accepter ou non.
* **Révoquer** : via **Meta App Dashboard** ou par l’utilisateur dans ses paramètres Facebook.
* **Supprimer** : si une permission est dépréciée ou inutile → supprimez-la dans le **Dashboard**.

***

### ⚠️ Points Importants pour Better Auth

* Quand vous configurez **Facebook OAuth dans Better Auth**, vous pouvez définir :

```ts
socialProviders: {
  facebook: {
    clientId: process.env.FACEBOOK_CLIENT_ID!,
    clientSecret: process.env.FACEBOOK_CLIENT_SECRET!,
    scopes: ["email", "public_profile", "user_friends"], // permissions demandées
    fields: ["id", "name", "email", "picture"], // infos récupérées
  }
}
```

👉 Les **scopes** correspondent aux **permissions Meta**.\
👉 Si vous demandez des permissions sensibles (ex: `user_friends`, `pages_show_list`, `ads_read`), vous devrez **passer en App Review**.

## 📌 **Permissions Facebook / Meta – Classées par cas d’usage**

***

### 1️⃣ **Authentification / Profil utilisateur (base)**

Ces permissions sont **souvent incluses par défaut** et ne nécessitent pas de review.

| Permission       | Description                                | Review |
| ---------------- | ------------------------------------------ | ------ |
| `email`          | Accès à l’email principal de l’utilisateur | ✅ Non  |
| `public_profile` | Infos publiques (nom, photo, ID Facebook)  | ✅ Non  |
| `user_link`      | Lien vers le profil Facebook               | ✅ Non  |

💡 Avec **Better Auth**, ces scopes sont ajoutés automatiquement si tu ne les désactives pas.

***

### 2️⃣ **Permissions Sociales**

⚠️ Ces permissions sont sensibles, souvent soumises à **App Review**.

| Permission      | Description                                                   | Review |
| --------------- | ------------------------------------------------------------- | ------ |
| `user_friends`  | Liste des amis de l’utilisateur **qui utilisent aussi l’app** | 📝 Oui |
| `user_birthday` | Date de naissance                                             | 📝 Oui |
| `user_gender`   | Genre                                                         | 📝 Oui |
| `user_likes`    | Pages aimées par l’utilisateur                                | 📝 Oui |

***

### 3️⃣ **Gestion de Pages Facebook**

Pour les apps qui interagissent avec des pages (publier, lire les insights, etc.).

| Permission                | Description                            | Review                         |
| ------------------------- | -------------------------------------- | ------------------------------ |
| `pages_show_list`         | Liste des pages que l’utilisateur gère | 📝 Oui                         |
| `pages_read_engagement`   | Lire les posts, commentaires, insights | 📝 Oui                         |
| `pages_manage_posts`      | Créer / modifier des posts             | 📝 Oui                         |
| `pages_manage_engagement` | Gérer commentaires et likes            | 📝 Oui                         |
| `pages_manage_ads`        | Créer et gérer publicités sur pages    | 🏢 Oui (Business Verification) |

***

### 4️⃣ **Instagram (avec Business Account)**

Requiert que l’utilisateur ait un **Instagram Business Account relié à une Page Facebook**.

| Permission                  | Description                                | Review |
| --------------------------- | ------------------------------------------ | ------ |
| `instagram_basic`           | Accès aux infos de base du compte IG       | 📝 Oui |
| `instagram_manage_comments` | Lire et répondre aux commentaires IG       | 📝 Oui |
| `instagram_manage_insights` | Lire analytics (followers, posts, stories) | 📝 Oui |
| `instagram_content_publish` | Publier du contenu sur IG                  | 🏢 Oui |

***

### 5️⃣ **Publicité et Business Manager**

Ces permissions demandent **Business Verification + App Review**, car elles donnent accès aux données business sensibles.

| Permission            | Description                                 | Review |
| --------------------- | ------------------------------------------- | ------ |
| `ads_read`            | Lire infos sur les campagnes publicitaires  | 🏢 Oui |
| `ads_management`      | Créer et gérer des campagnes                | 🏢 Oui |
| `business_management` | Gérer les assets business (BM, pages, apps) | 🏢 Oui |
| `leads_retrieval`     | Accéder aux leads générés par Facebook Ads  | 🏢 Oui |

***

## 📊 Exemple **Better Auth Config**

👉 Exemple : une app qui gère **login + accès aux pages Facebook**

```ts
export const auth = betterAuth({
  socialProviders: {
    facebook: {
      clientId: process.env.FACEBOOK_CLIENT_ID!,
      clientSecret: process.env.FACEBOOK_CLIENT_SECRET!,
      scopes: [
        "email",
        "public_profile",
        "pages_show_list",
        "pages_read_engagement"
      ],
      fields: ["id", "name", "email", "picture"],
    },
  },
});
```

***

⚠️ **Important** :

* Si tu demandes seulement `email` + `public_profile` → ✅ pas de review.
* Si tu ajoutes `pages_*`, `instagram_*` ou `ads_*` → 📝 App Review obligatoire.
* Si tu demandes accès aux **Ads / Business** → 🏢 Business Verification aussi obligatoire.
