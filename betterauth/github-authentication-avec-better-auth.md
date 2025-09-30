# 🔑 GitHub Authentication avec Better Auth

### 1️⃣ Récupérer vos credentials GitHub

1. Allez dans le [**GitHub Developer Portal**](https://github.com/settings/developers).
2. Créez une **OAuth App** (ou GitHub App, avec configuration spéciale).
3. Renseignez :
   * **App name** : ex. `MyApp GitHub Login`
   * **Homepage URL** : `http://localhost:3000` (ou domaine en prod)
   *   **Authorization callback URL** :

       ```
       http://localhost:3000/api/auth/callback/github
       ```

       (ou équivalent en prod).

⚠️ **Important** :

* Vous devez inclure la **scope `user:email`** (sinon vous n’aurez pas l’email utilisateur).
* Si vous utilisez un **GitHub App**, allez dans :\
  `Permissions and Events → Account Permissions → Email Addresses → Read-Only`\
  puis sauvegardez.

***

### 2️⃣ Configuration côté serveur (`auth.ts`)

```ts
import { betterAuth } from "better-auth"

export const auth = betterAuth({
  socialProviders: {
    github: {
      clientId: process.env.GITHUB_CLIENT_ID as string,
      clientSecret: process.env.GITHUB_CLIENT_SECRET as string,
    },
  },
});
```

***

### 3️⃣ Utilisation côté client (`auth-client.ts`)

```ts
import { createAuthClient } from "better-auth/client"

const authClient = createAuthClient();

const signIn = async () => {
  const { data, error } = await authClient.signIn.social({
    provider: "github",
  });
  
  if (error) {
    console.error("GitHub sign-in failed:", error);
  } else {
    console.log("Signed in:", data);
  }
};
```

***

### 4️⃣ Particularités GitHub

* ✅ **Pas de refresh token**\
  Contrairement à Google ou Facebook, **GitHub n’émet pas de refresh token**.\
  → Les access tokens restent valides indéfiniment (sauf révocation ou inactivité > 1 an).
* ⚠️ **Erreur `email_not_found`**\
  Cela survient si :
  * Vous utilisez un **GitHub App** sans activer `Email Read-Only`.
  * Vous n’avez pas inclus la scope `user:email`.

***

📌 **En résumé** :

* Utilisez une **OAuth App** si vous voulez une intégration simple.
* Si vous utilisez une **GitHub App**, pensez à activer l’accès aux emails.
* Pas besoin de refresh token → vous pouvez stocker directement l’`access_token`.
