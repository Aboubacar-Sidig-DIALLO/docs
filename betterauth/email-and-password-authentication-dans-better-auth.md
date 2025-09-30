# 📧🔑 Email & Password Authentication dans Better Auth

Better Auth inclut **nativement un provider email + mot de passe**, sans plugin supplémentaire.\
C’est la méthode la plus classique d’authentification.

***

### ⚙️ Configuration de base

Rien de spécial à activer, c’est disponible par défaut.\
Tu peux directement utiliser :

#### **Client-side (frontend)**

```ts
import { authClient } from "@/lib/auth-client";

// Inscription
await authClient.signUp.email({
  email: "john@example.com",
  password: "mypassword123",
});

// Connexion
await authClient.signIn.email({
  email: "john@example.com",
  password: "mypassword123",
});
```

#### **Server-side (backend)**

```ts
import { auth } from "@/lib/auth";

// Connexion côté serveur
await auth.api.signInEmail({
  body: {
    email: "john@example.com",
    password: "mypassword123",
  },
});

// Inscription côté serveur
await auth.api.signUpEmail({
  body: {
    email: "john@example.com",
    password: "mypassword123",
  },
});
```

***

### 🛠️ Personnalisation

* **Validation mot de passe** : tu peux imposer des règles (longueur, complexité, etc.) via les `hooks.before`.
* **Désactivation** : si tu veux uniquement des providers sociaux (Google, GitHub…), tu peux désactiver l’auth email.
* **Username au lieu d’email** : Better Auth propose un **plugin username** qui étend ce mode d’authentification.

***

### ✅ Bonnes pratiques

1. Toujours activer la **vérification d’email** pour renforcer la sécurité.
2. Penser à configurer un **forgot password** flow (réinitialisation).
3. Éventuellement ajouter une **2FA** pour plus de robustesse.
4. Stockage sécurisé : Better Auth gère le hashage des mots de passe (pas besoin d’implémenter toi-même).

#### 🔑 Activer l’authentification Email & Password

Par défaut, **Better Auth désactive le provider email + mot de passe**.\
Il faut donc l’activer explicitement dans la configuration de ton instance :

```ts
// auth.ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  // ... autres options
  emailAndPassword: { 
    enabled: true, 
  }, 
});
```

***

✅ Une fois activé :

*   Tu peux utiliser les méthodes côté client :

    ```ts
    await authClient.signUp.email({ email, password });
    await authClient.signIn.email({ email, password });
    ```
* Tu pourras ensuite enrichir la config avec :
  * **passwordPolicy** → définir les règles de complexité du mot de passe
  * **sendResetPassword** → permettre la réinitialisation par email
  * **sendVerificationEmail** → exiger une vérification avant connexion

#### ✍️ Sign Up avec Email & Mot de Passe

Une fois **emailAndPassword activé**, tu peux inscrire un utilisateur avec la méthode `signUp.email`.

***

**🚀 Côté Client**

```ts
import { authClient } from "@/lib/auth-client";

const { data, error } = await authClient.signUp.email({
  name: "John Doe",                      // requis
  email: "john.doe@example.com",         // requis
  password: "password1234",              // requis (8–128 caractères par défaut)
  image: "https://example.com/image.png", // optionnel
  callbackURL: "https://example.com/callback", // optionnel
});
```

***

**⚙️ Côté Serveur**

Tu peux aussi appeler directement l’endpoint :

```ts
await auth.api.signUpEmail({
  body: {
    name: "John Doe",
    email: "john.doe@example.com",
    password: "password1234",
    image: "https://example.com/image.png",
    callbackURL: "https://example.com/callback",
  },
});
```

***

**📑 Paramètres disponibles**

| Prop          | Description                                                         | Type   |
| ------------- | ------------------------------------------------------------------- | ------ |
| `name`        | Nom complet de l’utilisateur (obligatoire)                          | string |
| `email`       | Adresse email (obligatoire)                                         | string |
| `password`    | Mot de passe (min 8, max 128 caractères par défaut)                 | string |
| `image`       | (Optionnel) URL d’une image de profil                               | string |
| `callbackURL` | (Optionnel) URL de redirection après inscription (ex: `/dashboard`) | string |

#### 🔐 Sign In avec Email & Mot de Passe

Une fois un utilisateur inscrit, tu peux l’authentifier avec la méthode `signIn.email`.

***

**🚀 Côté Client**

```ts
import { authClient } from "@/lib/auth-client";

const { data, error } = await authClient.signIn.email({
  email: "john.doe@example.com",           // requis
  password: "password1234",                // requis (8–128 caractères par défaut)
  rememberMe: true,                        // optionnel (true par défaut)
  callbackURL: "https://example.com/callback", // optionnel
});
```

***

**⚙️ Côté Serveur**

```ts
await auth.api.signInEmail({
  body: {
    email: "john.doe@example.com",
    password: "password1234",
    rememberMe: true,
    callbackURL: "https://example.com/callback",
  },
});
```

***

**📑 Paramètres disponibles**

| Prop          | Description                                                                 | Type    |
| ------------- | --------------------------------------------------------------------------- | ------- |
| `email`       | Adresse email de l’utilisateur (obligatoire)                                | string  |
| `password`    | Mot de passe (8–128 caractères par défaut)                                  | string  |
| `rememberMe`  | Si `false`, la session expire à la fermeture du navigateur (défaut: `true`) | boolean |
| `callbackURL` | URL de redirection après connexion (optionnel, ex: `/dashboard`)            | string  |

#### 🚪 **Sign Out (Déconnexion)**

Better Auth fournit une méthode simple pour gérer la déconnexion utilisateur via la fonction `signOut`.

***

**⚛️ Côté Client**

```ts
import { authClient } from "@/lib/auth-client";

await authClient.signOut();
```

👉 Cela met fin à la session active (cookie supprimé côté serveur et côté client).

***

**⚙️ Côté Serveur**

```ts
await auth.api.signOut({
  headers: req.headers, // transmettre les cookies de session
});
```

***

**🔄 Redirection après déconnexion**

Tu peux utiliser `fetchOptions.onSuccess` pour rediriger l’utilisateur après une déconnexion réussie.

```ts
import { useRouter } from "next/navigation";

await authClient.signOut({
  fetchOptions: {
    onSuccess: () => {
      router.push("/login"); // redirige vers la page de connexion
    },
  },
});
```

#### 📧 **Email Verification (Vérification d’email)**

La **vérification d’email** dans **Better Auth** permet de confirmer qu’un utilisateur possède bien l’adresse email qu’il a fournie. Cela renforce la sécurité et réduit les risques de faux comptes.

***

#### ⚙️ **Configuration côté serveur (`auth.ts`)**

Tu dois définir la fonction `sendVerificationEmail`, qui envoie un email avec un lien de vérification.

```ts
import { betterAuth } from "better-auth";
import { sendEmail } from "./email"; // ta fonction personnalisée pour envoyer des mails

export const auth = betterAuth({
  emailVerification: {
    sendVerificationEmail: async ({ user, url, token }, request) => {
      await sendEmail({
        to: user.email,
        subject: "Verify your email address",
        text: `Click the link to verify your email: ${url}`,
      });
    },
  },
});
```

* **`user`** → L’objet utilisateur concerné.
* **`url`** → L’URL de vérification à envoyer à l’utilisateur.
* **`token`** → Jeton de vérification associé à l’utilisateur.
* **`request`** → Objet `Request` pour récupérer des infos additionnelles si nécessaire.

***

#### 🖥️ **Côté Client**

Tu peux déclencher l’envoi d’un email de vérification via la fonction `sendVerificationEmail`.

```ts
await authClient.sendVerificationEmail({
  email: "user@example.com",
  callbackURL: "/dashboard" // URL de redirection après la vérification
});
```

***

#### 🔄 **Redirection après clic sur le lien**

* ✅ **Si le token est valide** → l’email est marqué comme vérifié et l’utilisateur est redirigé vers `callbackURL`.
* ❌ **Si le token est invalide** → redirection vers `callbackURL?error=invalid_token`.

#### 📌 **Require Email Verification (Obliger la vérification d’email avant connexion)**

Avec **Better Auth**, tu peux forcer la vérification d’email avant qu’un utilisateur puisse se connecter.

***

#### ⚙️ **Configuration côté serveur (`auth.ts`)**

Ajoute l’option `requireEmailVerification: true` dans le bloc `emailAndPassword`.

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  emailAndPassword: {
    requireEmailVerification: true, // 🚨 Oblige la vérification avant connexion
  },
});
```

⚠️ Cela fonctionne uniquement si tu as déjà configuré `sendVerificationEmail` (la fonction qui envoie l’email avec lien/token).

***

#### 🖥️ **Gestion côté client (`auth-client.ts`)**

Si un utilisateur essaie de se connecter sans avoir validé son email, Better Auth renverra une **erreur 403**.\
Tu peux intercepter cette erreur avec `onError`.

```ts
await authClient.signIn.email(
  {
    email: "email@example.com",
    password: "password",
  },
  {
    onError: (ctx) => {
      if (ctx.error.status === 403) {
        alert("⚠️ Please verify your email address before logging in.");
      } else {
        alert(ctx.error.message);
      }
    },
  }
);
```

***

✅ **Résumé du comportement :**

* Lors de la première connexion → `sendVerificationEmail` est appelé.
* Si l’utilisateur clique le lien de vérification → email validé ✅.
* Tant que l’utilisateur n’a pas validé son email → connexion refusée avec **403 Forbidden**.

#### 📌 **Déclencher manuellement la vérification d’email**

En plus de l’envoi automatique à l’inscription ou à la connexion, tu peux **forcer l’envoi d’un email de vérification** à tout moment depuis ton client.

***

#### 🖥️ **Exemple côté client (`auth-client.ts`)**

```ts
await authClient.sendVerificationEmail({
  email: "user@email.com",
  callbackURL: "/", // 🚀 Redirigé ici après validation
});
```

***

#### ⚙️ **Comment ça marche ?**

* `email`: l’adresse de l’utilisateur à vérifier.
* `callbackURL`: URL de redirection après clic sur le lien.
  * Si le token est **valide** → redirection vers `/` (ou ton URL choisie).
  * Si le token est **invalide/expiré** → redirection vers `/` avec `?error=invalid_token`.

***

✅ **Cas d’usage typiques** :

* Bouton "Renvoyer l’email de vérification" dans ton dashboard.
* Support qui déclenche manuellement une nouvelle vérification.
* Rappel automatique après X minutes si l’utilisateur n’a pas encore validé.

#### 🔐 **Réinitialisation de mot de passe avec Better Auth**

Better Auth fournit un **flux complet de réinitialisation de mot de passe** basé sur un email contenant un lien sécurisé avec **token**.

***

### ⚙️ **Configuration côté serveur (`auth.ts`)**

Tu dois d’abord fournir une fonction `sendResetPassword` qui envoie le mail au user :

```ts
import { betterAuth } from "better-auth";
import { sendEmail } from "./email"; // ta fonction d'envoi d'email

export const auth = betterAuth({
  emailAndPassword: {
    enabled: true,
    sendResetPassword: async ({ user, url, token }, request) => {
      await sendEmail({
        to: user.email,
        subject: "Reset your password",
        text: `Click the link to reset your password: ${url}`,
      });
    },
    onPasswordReset: async ({ user }, request) => {
      // Action après succès (log, analytics, etc.)
      console.log(`Password for user ${user.email} has been reset.`);
    },
  },
});
```

***

### 🖥️ **Étape 1 : Demander la réinitialisation (client)**

L’utilisateur saisit son email → on envoie un lien avec token :

```ts
const { data, error } = await authClient.requestPasswordReset({
  email: "john.doe@example.com", // requis
  redirectTo: "https://example.com/reset-password", // page frontend de reset
});
```

🔎 **Params :**

* `email`: email du user.
* `redirectTo`: URL frontend où l’utilisateur sera redirigé avec `?token=...`.

***

### 🖥️ **Étape 2 : Page Reset Password (frontend)**

Sur ta page `/reset-password`, tu récupères le token dans l’URL et tu appelles `resetPassword`.

```ts
const token = new URLSearchParams(window.location.search).get("token");
if (!token) {
  throw new Error("Invalid or missing token");
}

const { data, error } = await authClient.resetPassword({
  newPassword: "password1234", // requis
  token, // requis
});
```

***

### ✅ **Résumé du flow**

1. User clique sur **"Mot de passe oublié"**.
2. Appel `authClient.requestPasswordReset()` → envoi d’un email avec `?token=xxx`.
3. User clique → redirection vers `/reset-password?token=xxx`.
4. Appel `authClient.resetPassword({ newPassword, token })`.
5. Mot de passe changé → `onPasswordReset` exécuté côté serveur.

#### 🔑 **Mise à jour du mot de passe avec Better Auth**

Contrairement à d’autres systèmes où le mot de passe est directement dans la table `user`, **Better Auth stocke les credentials dans la table `account`**.\
Pour modifier le mot de passe d’un utilisateur connecté, tu peux utiliser l’action `changePassword`.

***

### 🖥️ **Côté Client**

Un utilisateur authentifié peut mettre à jour son mot de passe en appelant `authClient.changePassword` :

```ts
const { data, error } = await authClient.changePassword({
  currentPassword: "oldpassword1234",   // 🔐 mot de passe actuel (obligatoire)
  newPassword: "newpassword1234",       // 🔑 nouveau mot de passe (obligatoire)
  revokeOtherSessions: true,            // optionnel : déconnecte toutes les autres sessions
});

if (error) {
  console.error("Erreur lors du changement de mot de passe :", error.message);
} else {
  console.log("Mot de passe mis à jour avec succès !");
}
```

***

### ⚙️ **Côté Serveur**

Tu peux aussi appeler l’endpoint `/change-password` directement (utile si tu gères une API REST ou GraphQL custom par-dessus).

```ts
await auth.api.changePassword({
  body: {
    currentPassword: "oldpassword1234",
    newPassword: "newpassword1234",
    revokeOtherSessions: true,
  },
  headers: req.headers, // important pour inclure le token de session
});
```

***

### 📌 **Props disponibles**

| Prop                  | Description                                                             | Type      | Obligatoire            |
| --------------------- | ----------------------------------------------------------------------- | --------- | ---------------------- |
| `newPassword`         | Le nouveau mot de passe à définir                                       | `string`  | ✅                      |
| `currentPassword`     | Le mot de passe actuel de l’utilisateur                                 | `string`  | ✅                      |
| `revokeOtherSessions` | Si `true`, invalide toutes les autres sessions actives de l’utilisateur | `boolean` | ❌ (par défaut `false`) |

#### ⚙️ **Configuration des mots de passe avec Better Auth**

Better Auth stocke les mots de passe dans la table **`account`** avec `providerId = "credential"`.\
Par défaut, il utilise **scrypt** comme algorithme de hachage, car :

* ✅ natif à Node.js
* ✅ recommandé par l’OWASP si `argon2id` n’est pas disponible
* ✅ conçu pour être lent et résistant aux attaques par force brute et GPU

***

### 🔒 **Options disponibles pour la configuration des mots de passe**

Tu peux personnaliser les paramètres liés aux mots de passe dans ton fichier **`auth.ts`**.

```ts
import { betterAuth } from "better-auth"

export const auth = betterAuth({
  emailAndPassword: {
    enabled: true, // Active l’authentification par email + mot de passe
    
    password: {
      hash: async (password: string) => {
        // ta fonction de hachage custom
        return "hashedPassword"
      },
      verify: async (password: string, hash: string) => {
        // ta fonction de vérification custom
        return password === hash
      },
    },
    
    minPasswordLength: 8,    // Longueur minimale (par défaut : 8)
    maxPasswordLength: 128,  // Longueur maximale (par défaut : 128)

    sendResetPassword: async ({ user, url, token }) => {
      // logique d’envoi d’email de reset
      console.log(`Envoyer reset à ${user.email} avec ${url}`)
    },

    onPasswordReset: async ({ user }) => {
      // callback exécuté après reset réussi
      console.log(`Password reset pour ${user.email}`)
    },

    resetPasswordTokenExpiresIn: 3600, // durée de validité du token (par défaut 1h)
    
    disableSignUp: false, // si true, empêche les nouveaux comptes avec email+password
  },
})
```

***

### 📌 **Résumé des propriétés**

| Option                        | Type       | Défaut   | Description                                     |
| ----------------------------- | ---------- | -------- | ----------------------------------------------- |
| `enabled`                     | `boolean`  | `false`  | Active l’authentification email + mot de passe  |
| `disableSignUp`               | `boolean`  | `false`  | Bloque les inscriptions par mot de passe        |
| `minPasswordLength`           | `number`   | `8`      | Taille minimale du mot de passe                 |
| `maxPasswordLength`           | `number`   | `128`    | Taille maximale du mot de passe                 |
| `sendResetPassword`           | `function` | `-`      | Fonction qui envoie l’email de réinitialisation |
| `onPasswordReset`             | `function` | `-`      | Callback après un reset réussi                  |
| `resetPasswordTokenExpiresIn` | `number`   | `3600`   | Durée de validité du token en secondes          |
| `password.hash`               | `function` | `scrypt` | Fonction custom de hachage                      |
| `password.verify`             | `function` | `scrypt` | Fonction custom de vérification                 |
