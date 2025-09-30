# 👤 User & Accounts — Better Auth

En plus de l’authentification, **Better Auth** fournit des méthodes intégrées pour **gérer les utilisateurs** :

* mise à jour des informations utilisateur,
* changement de mot de passe,
* suppression de compte, etc.

***

### 🗄️ Table `user`

La table **user** contient les données principales d’authentification.\
Par défaut, elle stocke :

* `id` → identifiant unique de l’utilisateur,
* `email` → email de connexion,
* `passwordHash` → mot de passe haché (si auth par email/mot de passe activé),
* `createdAt` et `updatedAt`,
* `emailVerified` → statut de vérification email.

👉 Tu peux **l’étendre** avec :

* `additionalFields` (dans ta config `betterAuth`),
* ou via un **plugin** pour ajouter des colonnes personnalisées.

***

### 🛠️ Gestion des utilisateurs (fonctions principales)

Better Auth expose des méthodes prêtes à l’emploi pour gérer les comptes :

#### 1. Mettre à jour les infos utilisateur

Permet de changer des champs comme `name`, `image`, ou même des `additionalFields`.

```ts
await authClient.updateUser({
  name: "Nouveau Nom",
  image: "https://example.com/avatar.png"
})
```

***

#### 2. Changer le mot de passe

L’utilisateur peut mettre à jour son mot de passe **s’il connaît l’ancien**.

```ts
await authClient.changePassword({
  currentPassword: "ancienMotDePasse",
  newPassword: "nouveauMotDePasse"
})
```

⚡ Option utile → révoquer les autres sessions après changement :

```ts
await authClient.changePassword({
  currentPassword: "ancienMotDePasse",
  newPassword: "nouveauMotDePasse",
  revokeOtherSessions: true
})
```

***

#### 3. Réinitialiser le mot de passe

Si l’utilisateur a oublié son mot de passe → **Password Reset Email**.\
Le backend envoie un lien sécurisé avec un `token`.

```ts
auth = betterAuth({
  emailAndPassword: {
    enabled: true,
    sendResetPassword: async ({ user, url }) => {
      await sendEmail({
        to: user.email,
        subject: "Réinitialisation de mot de passe",
        text: `Clique ici : ${url}`
      })
    }
  }
})
```

***

#### 4. Supprimer un compte

Un utilisateur peut supprimer son compte et toutes ses données associées.

```ts
await authClient.deleteUser()
```

***

#### 5. Lister / gérer les comptes liés

Si tu actives **OAuth / social login**, un utilisateur peut avoir **plusieurs comptes liés**.\
Better Auth gère :

* `linkSocial()` → lier un compte social (Google, GitHub, etc.)
* `unlinkSocial()` → délier un fournisseur.

***

### 🧩 Extension du schéma utilisateur

Tu peux enrichir le modèle `user` avec des champs supplémentaires :

```ts
export const auth = betterAuth({
  user: {
    additionalFields: {
      role: {
        type: "string",
        input: false // ⚠️ empêche l’utilisateur de définir ce champ lui-même
      },
      isPremium: {
        type: "boolean",
        input: false
      }
    }
  }
})
```

👉 Ces champs seront disponibles :

* côté **serveur** dans `auth.$Infer.Session`,
* côté **client** avec `authClient.$Infer.Session`.

***

## 🚀 En résumé

* La table **user** stocke les infos de base (email, mot de passe, etc.).
* Tu peux l’**étendre** avec des champs personnalisés (`role`, `status`, etc.).
* Better Auth fournit des **méthodes intégrées** pour :
  * `updateUser()` → mise à jour infos,
  * `changePassword()` et `resetPassword()`,
  * `deleteUser()` → suppression du compte,
  * `linkSocial()` et `unlinkSocial()` → gestion des connexions sociales.

## 🔄 Update User — Better Auth

La fonction **`updateUser`** permet de mettre à jour les informations de l’utilisateur connecté.\
Elle est disponible **côté client** (via `authClient`).

***

### 📌 Exemple d’utilisation

```ts
await authClient.updateUser({
  image: "https://example.com/image.jpg",
  name: "John Doe",
})
```

***

### 🛠️ Champs que tu peux mettre à jour

* **`name`** → le nom affiché de l’utilisateur
* **`image`** → l’avatar de l’utilisateur (URL)
* **champs personnalisés** (`additionalFields`) que tu as définis dans ton schéma utilisateur

👉 Exemple avec un champ supplémentaire `role` :

```ts
await authClient.updateUser({
  name: "Alice",
  role: "admin", // si défini dans additionalFields
})
```

***

### ⚡ Notes importantes

* L’utilisateur doit être **authentifié** (session valide).
* Tu ne peux pas modifier directement des champs sensibles comme `email` ou `password` via `updateUser`.
  * Pour l’**email** → passe par la **vérification email**.
  * Pour le **mot de passe** → passe par `changePassword`.
* Si tu as défini `input: false` sur un champ dans `additionalFields`, il ne pourra **pas** être mis à jour par l’utilisateur via `updateUser`.

## 📧 Change Email — Better Auth

Better Auth permet de laisser les utilisateurs **changer leur adresse email**, avec un mécanisme de vérification pour éviter les abus.

***

### ⚙️ Activer la fonctionnalité

Par défaut, le changement d’email est **désactivé**.\
Il faut l’activer dans la configuration :

```ts
export const auth = betterAuth({
  user: {
    changeEmail: {
      enabled: true,
    },
  },
})
```

***

### 📤 Vérification par email

Si l’utilisateur a déjà une **adresse email vérifiée**, tu dois définir une fonction `sendChangeEmailVerification`.\
Cette fonction envoie un lien de validation à **l’email actuel** pour confirmer la modification.

```ts
export const auth = betterAuth({
  user: {
    changeEmail: {
      enabled: true,
      sendChangeEmailVerification: async ({ user, newEmail, url, token }, request) => {
        await sendEmail({
          to: user.email, // l’email actuel (doit valider le changement)
          subject: "Approve email change",
          text: `Cliquez sur ce lien pour approuver le changement : ${url}`,
        })
      },
    },
  },
})
```

***

### 🖥️ Utilisation côté client

L’utilisateur appelle `changeEmail` avec son nouvel email et une URL de redirection après validation.

```ts
await authClient.changeEmail({
  newEmail: "new-email@email.com",
  callbackURL: "/dashboard", // redirection après vérification
})
```

***

### 🔑 Règles importantes

* ✅ Si l’**email actuel est vérifié** → un email de validation est envoyé (obligatoire).
* ⚡ Si l’**email actuel n’est pas vérifié** → le changement est appliqué immédiatement.
* 📩 Après validation → l’email est mis à jour dans la base, et une confirmation est envoyée à la **nouvelle adresse**.

## 🔑 Change Password — Better Auth

Le mot de passe **n’est pas stocké dans la table `user`** mais dans la table `account`.\
Better Auth fournit une méthode pour permettre aux utilisateurs de le modifier en toute sécurité.

***

### 🖥️ Côté client

On utilise la méthode `authClient.changePassword` :

```ts
const { data, error } = await authClient.changePassword({
  newPassword: "newpassword1234", // le nouveau mot de passe
  currentPassword: "oldpassword1234", // le mot de passe actuel
  revokeOtherSessions: true, // optionnel : révoque toutes les autres sessions actives
})
```

#### Paramètres disponibles

| Propriété             | Description                                                              | Type    | Requis |
| --------------------- | ------------------------------------------------------------------------ | ------- | ------ |
| `newPassword`         | Le nouveau mot de passe à définir                                        | string  | ✅      |
| `currentPassword`     | Le mot de passe actuel (vérification avant de changer)                   | string  | ✅      |
| `revokeOtherSessions` | Si `true`, invalide toutes les autres sessions actives (sécurité accrue) | boolean | ❌      |

***

### ⚙️ Côté serveur

Better Auth expose aussi une **route `/change-password`** utilisable côté serveur :

```ts
await auth.api.changePassword({
  body: {
    newPassword: "newpassword1234",
    currentPassword: "oldpassword1234",
    revokeOtherSessions: true,
  },
  headers: req.headers, // important pour l’authentification
})
```

***

### 🚨 Bonnes pratiques

* ✅ Toujours demander le **mot de passe actuel** avant de valider le changement.
* ✅ Activer `revokeOtherSessions: true` (par défaut recommandé) → ça empêche une session compromise de rester active après changement.
* ✅ Associer cette action à une **vérification d’email fraîche** ou une session **fresh** pour renforcer la sécurité.

## 👤 Set Password & Delete User — Better Auth

***

### 🔑 **Set Password**

👉 Cas d’usage :\
Si un utilisateur s’est inscrit via **OAuth (Google, GitHub, Facebook, etc.)**, il n’aura pas de mot de passe stocké dans la table `account`.\
Tu peux alors utiliser `setPassword` pour en définir un.

⚠️ **Important :**

* Cette action ne peut être exécutée **que côté serveur** pour des raisons de sécurité.
* La bonne pratique est plutôt de proposer un **"Mot de passe oublié"** (`forgot password`) pour initialiser un mot de passe.

#### Exemple (côté serveur) :

```ts
await auth.api.setPassword({
  body: {
    newPassword: "password1234",
  },
  headers: req.headers, // doit contenir le token de session utilisateur
})
```

***

### 🗑️ **Delete User**

👉 Par défaut, Better Auth ne permet pas la suppression d’utilisateur (sécurité).\
Mais tu peux l’activer via la config.

#### Activation :

```ts
export const auth = betterAuth({
  user: {
    deleteUser: {
      enabled: true,
    },
  },
})
```

#### Utilisation (côté client) :

```ts
await authClient.deleteUser()
```

👉 Cela supprime **définitivement** le compte utilisateur et toutes ses données associées dans la base.\
⚠️ À utiliser avec précaution (souvent protégé par une confirmation forte : email, TOTP, re-authentication).

***

### ✅ Bonnes pratiques

* **Set Password** :
  * Utiliser uniquement côté serveur.
  * Associer à un flux sécurisé (`forgot password`) pour éviter les abus.
* **Delete User** :
  * Toujours demander une **confirmation forte** (ex. re-saisie du mot de passe, validation par email, TOTP).
  * Documenter clairement que la suppression est **irréversible**.
  * Si tu veux proposer une suppression "logique" (désactivation au lieu de suppression physique), il vaut mieux gérer ça avec un **champ `isDeleted`** côté base.

## 🛡️ Ajouter une vérification avant suppression de compte

***

### ⚙️ **Configuration côté serveur**

👉 Tu actives la suppression **et** tu fournis une fonction `sendDeleteAccountVerification` pour envoyer un mail de confirmation avec un lien sécurisé.

```ts
import { betterAuth } from "better-auth";
import { sendEmail } from "./email"; // ta fonction d'envoi d'email

export const auth = betterAuth({
  user: {
    deleteUser: {
      enabled: true,
      sendDeleteAccountVerification: async ({ user, url, token }, request) => {
        // Exemple basique d'envoi d’email
        await sendEmail({
          to: user.email,
          subject: "Confirmez la suppression de votre compte",
          text: `Cliquez sur ce lien pour confirmer la suppression de votre compte : ${url}`,
        });

        // Si tu veux générer ton propre lien :
        // const customUrl = `https://your-app.com/delete-account?token=${token}`;
        // await sendEmail({ to: user.email, subject: "...", text: customUrl });
      },
    },
  },
});
```

***

### 🖥️ **Côté client**

#### 1. Envoi de la demande de suppression (avec callback) :

```ts
await authClient.deleteUser({
  callbackURL: "/goodbye" // après suppression réussie, redirige vers cette page
});
```

👉 Cela déclenche l’envoi d’un mail contenant l’URL de vérification.

***

#### 2. Utilisation du **token reçu par email** (si tu crées ton propre lien) :

```ts
await authClient.deleteUser({
  token: "<verification-token>"
});
```

***

### 🔒 Règles de sécurité intégrées

* ✅ L’utilisateur doit être **connecté** pour initier la suppression.
* ✅ Le lien envoyé contient un **token unique et signé**, utilisable une seule fois.
* ✅ Une fois le lien cliqué, la suppression est effective et redirige vers `callbackURL`.

***

### 🚀 Bonnes pratiques

1. **Toujours confirmer par email** (évite qu’un attaquant ayant accès temporairement à une session supprime le compte).
2. **Ajouter un second facteur** (OTP/TOTP) pour plus de sécurité (surtout si l’app gère des données sensibles).
3. **Loguer l’événement** (`afterDeleteUser` hook → trace de sécurité).
4. **Prévoir une période de "grâce"** → au lieu de supprimer immédiatement, tu peux marquer l’utilisateur comme `deleted=true` et purger réellement après X jours (conformité RGPD).

## 🔐 Conditions pour supprimer un compte

Pour protéger la suppression, **Better Auth** impose qu’un utilisateur remplisse **au moins une condition** :

***

### 1. **Avec mot de passe (classique)**

Si l’utilisateur a un mot de passe, il doit le fournir pour valider la suppression :

```ts
await authClient.deleteUser({
  password: "password123"
});
```

👉 Ici, Better Auth va **vérifier le mot de passe** avant de supprimer.

***

### 2. **Avec session fraîche**

Si aucun mot de passe n’est fourni, Better Auth vérifie que la session est **récente (fresh)**.\
Par défaut : `freshAge = 24h` → l’utilisateur doit s’être connecté ou réauthentifié dans les 24h.

```ts
await authClient.deleteUser();
```

⚙️ Tu peux configurer ça dans ton `auth.ts` :

```ts
export const auth = betterAuth({
  session: {
    freshAge: 60 * 10 // ex. 10 minutes de fraîcheur max
  }
});
```

👉 **Conseil sécurité** : ne mets pas `0` sauf si tu forces une vérification par email (voir cas 3).

***

### 3. **Avec vérification par email (OAuth users ou extra sécurité)**

Les utilisateurs OAuth (Google, GitHub, etc.) n’ont pas de mot de passe → la seule option est la **vérification par email**.

Tu dois avoir configuré :

```ts
user: {
  deleteUser: {
    enabled: true,
    sendDeleteAccountVerification: async ({ user, url, token }) => {
      await sendEmail({
        to: user.email,
        subject: "Confirmez la suppression de votre compte",
        text: `Cliquez ici : ${url}`
      })
    }
  }
}
```

Ensuite côté client, l’appel est simple :

```ts
await authClient.deleteUser();
```

👉 Si tu utilises un **lien personnalisé** dans le mail (ex. `/delete-account?token=xxx`), tu devras fournir le `token` :

```ts
await authClient.deleteUser({
  token: "token-reçu-par-email"
});
```

***

## ✅ Recommandations de sécurité

* 🔑 Si **mot de passe dispo** → demande toujours le mot de passe.
* ⏳ Si **pas de mot de passe** (OAuth) → utilise **vérification email** + `freshAge` court (genre 5-10 minutes).
* 📩 Même avec mot de passe, tu peux combiner **confirmation par email** pour éviter les suppressions accidentelles.
* 🗄️ Option RGPD → stocker `deletedAt` et supprimer définitivement après un délai (14-30 jours).

## 🔄 Callbacks `beforeDelete` & `afterDelete`

Better Auth te permet d’exécuter du code **avant** et **après** la suppression définitive d’un utilisateur.\
C’est utile pour ajouter des **contrôles, nettoyages, logs ou side-effects**.

***

### 1. `beforeDelete`

👉 Exécuté **juste avant** la suppression.\
Tu peux :

* Vérifier des conditions
* Nettoyer des données liées (fichiers, tokens…)
* **Empêcher la suppression** en levant une erreur (`APIError`).

#### Exemple : simple nettoyage

```ts
export const auth = betterAuth({
  user: {
    deleteUser: {
      enabled: true,
      beforeDelete: async (user) => {
        console.log(`Suppression en cours pour ${user.email}`);
        // Exemple : supprimer ses fichiers S3 ou notifier un service externe
      },
    },
  },
});
```

#### Exemple : bloquer les comptes admin

```ts
import { APIError } from "better-auth/api";

export const auth = betterAuth({
  user: {
    deleteUser: {
      enabled: true,
      beforeDelete: async (user, request) => {
        if (user.email.includes("admin")) {
          throw new APIError("BAD_REQUEST", {
            message: "Les comptes admin ne peuvent pas être supprimés",
          });
        }
      },
    },
  },
});
```

***

### 2. `afterDelete`

👉 Exécuté **après** la suppression en base.\
Idéal pour :

* **Envoyer un mail de confirmation**
* Loguer l’événement (audit / analytics)
* Supprimer des entrées secondaires dans d’autres bases (logs, commandes, etc.)

#### Exemple : log et notification

```ts
export const auth = betterAuth({
  user: {
    deleteUser: {
      enabled: true,
      afterDelete: async (user, request) => {
        console.log(`Utilisateur supprimé : ${user.email}`);
        await sendEmail({
          to: "support@myapp.com",
          subject: "Suppression d’un compte",
          text: `Le compte ${user.email} a été supprimé.`
        });
      },
    },
  },
});
```

***

## ⚖️ Bonnes pratiques

* ✅ Utiliser **`beforeDelete`** pour **vérifier ou bloquer** (ex. rôle `admin`, comptes protégés).
* ✅ Utiliser **`afterDelete`** pour **notifier ou nettoyer** (logs, analytics, emails, cache Redis…).
* ⚠️ Ne pas mettre de logique critique dans `afterDelete` qui pourrait échouer → car à ce stade, l’utilisateur est déjà supprimé.

## 🔑 Accounts dans Better Auth

### 1. Définition

* Un **account** représente la **liaison entre un utilisateur et un fournisseur d’authentification** (_provider_).
* Chaque fois qu’un utilisateur s’authentifie via un provider (Email/Password, Google, GitHub, etc.), une **entrée est créée dans la table `account`**.

👉 Cela permet à un même utilisateur (`user`) d’avoir **plusieurs comptes liés** :

* Un via email/password
* Un via Google OAuth
* Un via GitHub OAuth\
  ➡️ Tous reliés au **même `userId`**.

***

### 2. Rôle de la table `account`

La table **`account`** stocke :

* 🔹 **userId** → identifie l’utilisateur dans la table `user`
* 🔹 **providerId** → identifiant du provider (`google`, `github`, `email`, etc.)
* 🔹 **providerAccountId** → ID renvoyé par le provider (ex : `sub` d’un token Google)
* 🔹 **accessToken** (facultatif) → jeton d’accès renvoyé par le provider
* 🔹 **refreshToken** (facultatif) → jeton pour rafraîchir l’accès
* 🔹 **expiresAt** → expiration du jeton
* 🔹 Autres infos provider spécifiques (ex: scopes)

👉 Ainsi, tu peux gérer **plusieurs providers par utilisateur**.

***

### 3. Exemple concret

#### 1 utilisateur → 3 providers

| userId | provider | providerAccountId | accessToken | refreshToken |
| ------ | -------- | ----------------- | ----------- | ------------ |
| u1     | email    | u1@email.com      | null        | null         |
| u1     | google   | 1234567890        | at\_abc     | rt\_def      |
| u1     | github   | 9876543210        | at\_xyz     | rt\_pqr      |

***

### 4. Cas d’usage

* 🔗 **Lien d’un compte OAuth** à un utilisateur existant (via `linkSocial`)
* 🔄 **Récupération/refresh du jeton** d’un provider (`getAccessToken`)
* 🛠️ **Migration** : possibilité d’ajouter plusieurs providers au même utilisateur

***

⚡ Résumé :

* `user` = entité principale (profil de l’utilisateur)
* `account` = liaison avec chaque méthode d’authentification
* Tu peux donc **supporter multi-login** (Email + Google + GitHub) pour le même utilisateur sans dupliquer les données.

## 📑 Lister les comptes liés à un utilisateur

Better Auth permet de récupérer **tous les comptes (providers)** rattachés à un même utilisateur.\
👉 C’est utile pour savoir si un utilisateur a lié son compte Google, GitHub, Email/Password, etc.

***

### ✅ Utilisation côté client

```ts
import { authClient } from "@/lib/auth-client";

const accounts = await authClient.listAccounts();

console.log(accounts);
/*
[
  {
    id: "acc_123",
    providerId: "email",
    providerAccountId: "user@email.com",
    userId: "usr_456",
    createdAt: "2025-09-29T12:34:56.000Z",
    updatedAt: "2025-09-29T12:34:56.000Z",
  },
  {
    id: "acc_124",
    providerId: "google",
    providerAccountId: "1092387456123",
    userId: "usr_456",
    accessToken: "...",
    refreshToken: "...",
    expiresAt: "2025-09-30T12:34:56.000Z"
  }
]
*/
```

***

### 🔎 Détails de la réponse

Chaque objet `account` contient (selon le provider) :

* `id` → identifiant interne du compte
* `providerId` → ex. `"email"`, `"google"`, `"github"`
* `providerAccountId` → identifiant côté provider (email ou `sub` Google, etc.)
* `userId` → ID de l’utilisateur (`user` table)
* `accessToken`, `refreshToken`, `expiresAt` → si c’est un OAuth provider
* `createdAt`, `updatedAt`

***

### ⚡ Cas d’usage

* Afficher à l’utilisateur **quels comptes sont liés** (Email, Google, GitHub, etc.)
* Permettre de **gérer les connexions** (ex: bouton "Délier Google")
* Vérifier si un utilisateur a déjà lié un provider avant d’en demander un nouveau.

## 🔑 Token Encryption dans Better Auth

### ⚠️ Par défaut

* **Better Auth ne chiffre pas les tokens** (`accessToken`, `refreshToken`) stockés en base.
* Raison : cela te laisse le **contrôle total** de l’implémentation (algorithme, clé, méthode).
* Tu peux donc appliquer ton propre système d’**encryption/décryption** via les **databaseHooks**.

***

### ✅ Exemple d’encryption à la création

```ts
import { betterAuth } from "better-auth";
import { encrypt } from "./crypto-utils"; // ta fonction maison

export const auth = betterAuth({
  databaseHooks: {
    account: {
      create: {
        before(account, context) {
          const withEncryptedTokens = { ...account };

          if (account.accessToken) {
            const encryptedAccessToken = encrypt(account.accessToken);
            withEncryptedTokens.accessToken = encryptedAccessToken;
          }

          if (account.refreshToken) {
            const encryptedRefreshToken = encrypt(account.refreshToken);
            withEncryptedTokens.refreshToken = encryptedRefreshToken;
          }

          return {
            data: withEncryptedTokens,
          };
        },
      },
    },
  },
});
```

***

### 🔓 Déchiffrement lors de l’utilisation

Quand tu lis l’`account` depuis la DB, tu dois **décrypter les tokens avant usage** :

```ts
import { decrypt } from "./crypto-utils";

const account = await db.account.findUnique({ where: { id: "acc_123" } });

if (account?.accessToken) {
  account.accessToken = decrypt(account.accessToken);
}

if (account?.refreshToken) {
  account.refreshToken = decrypt(account.refreshToken);
}
```

***

### 🔐 Bonnes pratiques

1. Utiliser un algo standard : **AES-256-GCM** ou **libsodium**.
2. Stocker ta clé d’encryption dans un **secret manager** (AWS KMS, GCP Secret Manager, Vault…).
3. Éviter les algos faibles ou "fait maison".
4. Définir un **schéma clair** :
   * chiffrement en DB (repos)
   * déchiffrement uniquement quand nécessaire

## 🔗 Account Linking dans Better Auth

### ⚙️ Activation de base

Pour autoriser le **multi-provider par utilisateur** :

```ts
export const auth = betterAuth({
  account: {
    accountLinking: {
      enabled: true,
    },
  },
});
```

👉 Sans ça, chaque nouveau provider crée **un nouvel utilisateur** séparé.

***

### 🚀 Forced Linking (trusted providers)

Tu peux désigner certains providers comme "de confiance" :

* Si un utilisateur se connecte avec eux, son compte sera **automatiquement lié**, même si l’email n’est pas marqué comme vérifié par le provider.\
  ⚠️ Attention : cela augmente le risque d’**account takeover** si le provider ne valide pas bien l’email.

```ts
export const auth = betterAuth({
  account: {
    accountLinking: {
      enabled: true,
      trustedProviders: ["google", "github"], // attention à qui tu mets ici
    },
  },
});
```

***

### 🖇️ Lier manuellement des comptes

#### 1. Lier un **compte social**

Depuis le **client**, si l’utilisateur est déjà connecté :

```ts
await authClient.linkSocial({
  provider: "google",
  callbackURL: "/callback", // où rediriger après le lien
});
```

👉 Tu peux demander des **scopes supplémentaires** différents de ceux utilisés au signup initial :

```ts
await authClient.linkSocial({
  provider: "google",
  scopes: ["https://www.googleapis.com/auth/drive.readonly"],
});
```

***

#### 2. Lier via **ID Token**

Tu peux **éviter un nouveau flow OAuth** en passant directement un ID token valide (utile sur mobile avec SDK natif, ou pour des flows customisés) :

```ts
await authClient.linkSocial({
  provider: "google",
  idToken: {
    token: "id_token_from_provider",
    nonce: "nonce_used_for_token", // facultatif
    accessToken: "access_token",   // facultatif (certains providers le demandent)
    refreshToken: "refresh_token"  // facultatif
  },
});
```

⚠️ L’ID token doit être **valide et vérifiable** par le provider.

***

#### 3. Lier des **comptes credentials (email/password)**

Impossible côté client (sécurité).\
Tu dois le faire côté **serveur** via `setPassword` :

```ts
await auth.api.setPassword({
  headers: /* session token de l'utilisateur */,
  password: "nouveauPasswordFort123!",
});
```

***

### ⚙️ Options avancées

* **allowDifferentEmails**\
  Permet de lier un compte social avec une **autre adresse email** que celle du compte principal (par défaut : interdit).

```ts
accountLinking: {
  allowDifferentEmails: true
}
```

* **updateUserInfoOnLink**\
  Permet de mettre à jour le **profil utilisateur** (nom, image, etc.) quand un nouveau provider est lié.

```ts
accountLinking: {
  updateUserInfoOnLink: true
}
```

***

## ✅ Bonnes pratiques

1. **Toujours forcer la vérification email** sauf si tu es certain que ton provider est fiable.
2. Utiliser `updateUserInfoOnLink: true` uniquement si tu veux centraliser les infos utilisateur.
3. Sur mobile, privilégier l’usage d’`idToken` pour éviter des redirects complexes.
4. Garde en tête que **setPassword** doit rester **server-side only** pour ne pas exposer de risques.

## 🔓 Account Unlinking dans Better Auth

### 🛠️ Usage de base

Tu peux **délier un compte social ou provider** lié à un utilisateur :

```ts
// Délier un provider entier
await authClient.unlinkAccount({
  providerId: "google"
});

// Délier un compte précis (utile si plusieurs comptes Google sont liés)
await authClient.unlinkAccount({
  providerId: "google",
  accountId: "123" // l’ID du compte retourné par listAccounts()
});
```

👉 Si le compte n’existe pas, une **erreur est levée**.

***

### ⚠️ Sécurité & Prévention de lockout

Par défaut, Better Auth **empêche de délier le dernier compte lié** pour éviter de bloquer définitivement l’utilisateur (perte d’accès totale).

#### Exemple :

* L’utilisateur n’a **qu’un seul provider (Google)** → unlink interdit.
* L’utilisateur a **Google + GitHub** → unlink possible tant qu’il reste **au moins un provider** actif.

***

### 🔑 Autoriser le délien complet (risqué)

Si tu veux autoriser les utilisateurs à **supprimer tous leurs comptes liés** (au risque de lockout), tu peux le configurer :

```ts
export const auth = betterAuth({
  account: {
    accountLinking: {
      allowUnlinkingAll: true
    }
  }
});
```

👉 À utiliser uniquement si tu prévois une **autre méthode de récupération** (par ex. réinvitation admin, mot de passe temporaire, etc.).

***

### ✅ Bonnes pratiques

1. **Ne pas activer `allowUnlinkingAll`** sauf si tu as un mécanisme de récupération.
2. Proposer une **confirmation forte** avant de permettre l’unlinking (ex: re-saisie du mot de passe ou 2FA).
3. Toujours afficher la **liste des comptes liés** (`listAccounts()`) avant d’autoriser un unlink.
4. Pour l’UX, si c’est le **dernier compte** → redirige plutôt l’utilisateur vers **deleteUser** (suppression complète du compte).
