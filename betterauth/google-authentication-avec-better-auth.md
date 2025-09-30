# 🌐 Google Authentication avec Better Auth

### 1️⃣ Obtenir vos credentials Google

1. Allez dans la **Google Cloud Console**.
2. Créez un **nouveau projet** (ou utilisez-en un existant).
3. Activez l’API **Google Identity Services**.
4. Dans **API & Services → Identifiants (Credentials)** :
   * Cliquez sur **Créer des identifiants → ID client OAuth 2.0**.
   * Type d’application : **Application Web**.
   * Ajoutez vos **URIs autorisés** :
     *   Développement :

         ```
         http://localhost:3000
         ```
     *   Production (exemple) :

         ```
         https://example.com
         ```
   * Ajoutez vos **Authorized redirect URIs** :
     *   Développement :

         ```
         http://localhost:3000/api/auth/callback/google
         ```
     *   Production (exemple) :

         ```
         https://example.com/api/auth/callback/google
         ```
5. Récupérez le **Client ID** et le **Client Secret**.

***

### 2️⃣ Configurer le provider (`auth.ts`)

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID as string,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET as string,
    },
  },
});
```

***

### 3️⃣ Utilisation côté client (`auth-client.ts`)

```ts
import { createAuthClient } from "better-auth/client";

const authClient = createAuthClient();

const signInWithGoogle = async () => {
  const { data, error } = await authClient.signIn.social({
    provider: "google",
  });

  if (error) {
    console.error("Google sign-in failed:", error);
  } else {
    console.log("Signed in successfully:", data);
  }
};
```

***

### 4️⃣ Notes importantes

* ✅ **Scopes par défaut** : `email`, `profile`.
* ⚙️ Vous pouvez personnaliser les **scopes** (par ex. accès à Google Drive, Calendar, etc.) avec `scope` dans la config du provider.
* ⚠️ **Redirect URI obligatoire** : doit être **exactement identique** dans Google Cloud Console et Better Auth, sinon → erreur `redirect_uri_mismatch`.

## 🔑 Usage — Sign In with Google

Pour connecter un utilisateur avec **Google**, vous pouvez utiliser la fonction `signIn.social` du client.

#### 📌 Propriété obligatoire

* **provider** : doit être défini sur `"google"`.

***

#### ✅ Exemple côté client (`auth-client.ts`)

```ts
import { createAuthClient } from "better-auth/client";

const authClient = createAuthClient();

const signIn = async () => {
  const { data, error } = await authClient.signIn.social({
    provider: "google",
  });

  if (error) {
    console.error("❌ Échec de connexion avec Google :", error);
  } else {
    console.log("✅ Connexion réussie :", data);
  }
};
```

## 🔑 Sign In with Google With ID Token

Il est possible de se connecter avec **Google** en utilisant directement un **ID Token**.\
Cette méthode est utile lorsque :

* Vous récupérez déjà l’ID Token côté **client** (par ex. via Google One Tap, un SDK mobile, ou une intégration OAuth personnalisée).
* Vous souhaitez éviter toute redirection et signer l’utilisateur directement.

⚡ **Remarque** : Si un `idToken` est fourni, aucune redirection n’aura lieu, l’utilisateur sera connecté immédiatement.

***

#### ✅ Exemple côté client (`auth-client.ts`)

```ts
const { data, error } = await authClient.signIn.social({
  provider: "google",
  idToken: {
    token: "GOOGLE_ID_TOKEN",       // requis : ID Token de Google
    accessToken: "GOOGLE_ACCESS_TOKEN" // optionnel : Access Token
  }
});

if (error) {
  console.error("❌ Erreur lors de la connexion avec Google :", error);
} else {
  console.log("✅ Connexion réussie :", data);
}
```

***

👉 Pour une intégration simplifiée avec **Google One Tap**, BetterAuth propose un **One Tap Plugin** dédié.

## 🟢 Google One Tap Plugin avec Better Auth

Le **Google One Tap** permet aux utilisateurs de se connecter rapidement sans avoir à saisir leur email/mot de passe ni passer par une redirection classique.\
Better Auth fournit un **plugin dédié** qui simplifie cette intégration.

***

### ⚙️ Étape 1 : Activer One Tap dans Google Cloud Console

1. Rendez-vous dans Google Cloud Console.
2. Créez ou sélectionnez un projet.
3. Allez dans **APIs & Services > Credentials**.
4. Créez des **OAuth 2.0 Client IDs** (type : Web).
5. Ajoutez vos **Authorized JavaScript origins** (ex. `http://localhost:3000`, `https://example.com`).
6. Récupérez votre **Client ID** (`GOOGLE_CLIENT_ID`).

***

### ⚙️ Étape 2 : Installer le plugin One Tap

```bash
pnpm add @better-auth/plugin-google-one-tap
```

ou

```bash
npm install @better-auth/plugin-google-one-tap
```

***

### ⚙️ Étape 3 : Configurer Better Auth avec One Tap

Dans ton fichier `auth.ts` :

```ts
import { betterAuth } from "better-auth";
import { googleOneTap } from "@better-auth/plugin-google-one-tap";

export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID as string,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET as string,
    },
  },
  plugins: [
    googleOneTap({
      clientId: process.env.GOOGLE_CLIENT_ID as string,
      autoSelect: true, // Permet de réutiliser automatiquement le compte
      cancelOnTapOutside: false, // Pour éviter une fermeture accidentelle
    }),
  ],
});
```

***

### ⚙️ Étape 4 : Utiliser One Tap côté client

Dans ton `auth-client.ts` :

```ts
import { createAuthClient } from "better-auth/client";

const authClient = createAuthClient();

export const initGoogleOneTap = async () => {
  await authClient.signIn.googleOneTap({
    callbackURL: "/dashboard", // Redirige après succès
  });
};
```

Puis, dans ton `index.tsx` ou `App.tsx` :

```ts
import { useEffect } from "react";
import { initGoogleOneTap } from "@/lib/auth-client";

export default function Home() {
  useEffect(() => {
    initGoogleOneTap();
  }, []);

  return <div>Bienvenue 👋</div>;
}
```

***

### ✨ Résultat

* L’utilisateur verra le **prompt One Tap** Google directement.
* S’il est déjà connecté à Google → connexion immédiate.
* Sinon, un sélecteur de compte Google apparaîtra.

## 🔹 Toujours demander la sélection d’un compte Google

Par défaut, Google peut choisir automatiquement un compte déjà connecté.\
Si tu veux forcer l’utilisateur à **choisir un compte à chaque connexion**, tu dois configurer `prompt: "select_account"` dans ton provider Google.

***

### ⚙️ Exemple de configuration

```ts
// auth.ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID as string,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET as string,
      prompt: "select_account", // ✅ oblige Google à afficher le sélecteur de compte
    },
  },
});
```

***

### 🔹 Autres valeurs possibles pour `prompt`

* `"select_account"` → toujours forcer le choix d’un compte.
* `"consent"` → demander à l’utilisateur de **réautoriser** les permissions.
* `"login"` → forcer la saisie de l’identifiant/mot de passe (même si déjà connecté).
* `"none"` → aucune interaction utilisateur (connexion silencieuse, échoue si pas déjà connecté).
* Tu peux même les **combiner** → `"select_account consent"`.

## 🔹 Demander des **scopes Google supplémentaires** après inscription

👉 Cas typique :

* L’utilisateur s’inscrit avec Google (scope minimal : `email`, `profile`).
* Plus tard, ton app a besoin d’un accès à **Google Drive** ou **Gmail**.
* Plutôt que de redemander toute l’authentification, tu relances un **OAuth flow ciblé** pour ajouter ces permissions.

***

### ⚙️ Exemple pratique avec Better Auth

```ts
// auth-client.ts
const requestGoogleDriveAccess = async () => {
  await authClient.linkSocial({
    provider: "google",
    scopes: ["https://www.googleapis.com/auth/drive.file"], // ✅ Nouveau scope demandé
  });
};
```

#### 🔹 Dans ton UI React

```tsx
<button onClick={requestGoogleDriveAccess}>
  Autoriser l'accès à Google Drive
</button>
```

***

### ✅ Ce qu’il se passe en coulisse

1. Better Auth relance le flow OAuth **uniquement pour le provider Google**.
2. Google affiche un écran demandant la nouvelle autorisation (ex. accès à Drive).
3. Si accepté :
   * Le **compte reste lié** (pas de doublon).
   * La **table Account** en DB est mise à jour avec le nouveau scope.
   * Les futurs **accessToken** incluront les nouveaux scopes.

***

### ⚠️ Points importants

* Nécessite **Better Auth ≥ 1.2.7** (sinon tu risques l’erreur `Social account already linked`).
* Les scopes existants sont **conservés** et les nouveaux sont ajoutés.
* Si tu veux redemander plusieurs fois, fais attention à la **gestion du refresh token** (souvent Google n’en renvoie qu’un).
*   Tu peux chaîner plusieurs scopes :

    ```ts
    scopes: [
      "https://www.googleapis.com/auth/drive.file",
      "https://www.googleapis.com/auth/gmail.readonly",
    ]
    ```

## 🔹 Toujours obtenir un **Refresh Token** avec Google

👉 Problème :

* Google ne donne un **refresh token** qu’au **premier consentement** de l’utilisateur.
* Si l’utilisateur se reconnecte sans révoquer l’accès, Google renvoie **seulement un access token** (valide mais temporaire).
* Résultat : ton app ne peut pas rafraîchir automatiquement le token pour garder un accès long terme.

***

### ⚙️ Solution : forcer Google à renvoyer un refresh token

#### Config côté `auth.ts`

```ts
socialProviders: {
  google: {
    clientId: process.env.GOOGLE_CLIENT_ID as string,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET as string,
    accessType: "offline",  // ✅ Demande un refresh token
    prompt: "select_account consent", // ✅ Forcer re-consent et sélection de compte
  },
}
```

* `accessType: "offline"` → Google donne un **refresh token** utilisable même quand l’utilisateur est hors ligne.
* `prompt: "select_account consent"` → oblige Google à afficher la fenêtre de consentement et à redemander les scopes.

***

### 🔄 Cas où ça ne marche pas encore

Même avec cette config :

* Si l’utilisateur a déjà autorisé ton app, **Google ne redonne pas de refresh token** (sauf si consent forcé).
* Pour vraiment obtenir un nouveau refresh token, il doit **révoquer l’accès** depuis son compte Google :
  * Mon compte Google → Sécurité → Applications tierces avec accès au compte
  * Supprimer ton app
  * Puis se reconnecter → nouveau refresh token généré ✅

***

### 📌 Bonnes pratiques

1. **Stocker le refresh token** en base de données dès la première connexion.
2. **Ne jamais le perdre** → c’est la clé qui te permet de régénérer des access tokens.
3. Mettre en place un système de **refresh automatique** via l’API de Google (si l’access token expire).
4. Si l’utilisateur supprime/revoque l’accès, prévoir un flow **linkSocial** pour redemander les scopes et rafraîchir le refresh token.

## ⚙️ 1. Configuration du provider (avec refresh token garanti)

`auth.ts`

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID as string,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET as string,

      // 🔑 Forcer Google à donner un refresh token
      accessType: "offline",
      prompt: "select_account consent",

      // 🔄 Fonction personnalisée de refresh
      refreshAccessToken: async (token) => {
        // Google endpoint pour régénérer un access token
        const response = await fetch("https://oauth2.googleapis.com/token", {
          method: "POST",
          headers: { "Content-Type": "application/x-www-form-urlencoded" },
          body: new URLSearchParams({
            client_id: process.env.GOOGLE_CLIENT_ID!,
            client_secret: process.env.GOOGLE_CLIENT_SECRET!,
            grant_type: "refresh_token",
            refresh_token: token.refreshToken!,
          }),
        });

        const newTokens = await response.json();

        if (!response.ok) {
          throw new Error(`Failed to refresh Google token: ${newTokens.error}`);
        }

        return {
          accessToken: newTokens.access_token,
          refreshToken: newTokens.refresh_token ?? token.refreshToken, // garder l’ancien si Google n’en renvoie pas
        };
      },
    },
  },
});
```

🔎 Explication :

* `refreshAccessToken` est déclenché automatiquement par Better Auth si l’`accessToken` est expiré.
* Tu appelles l’API officielle Google pour échanger le `refreshToken` → nouveau `accessToken`.
* Si Google ne renvoie pas de nouveau refresh token → on garde l’ancien (`token.refreshToken`).

***

## ⚙️ 2. Usage côté client : obtenir un access token valide

`auth-client.ts`

```ts
import { createAuthClient } from "better-auth/client";

export const authClient = createAuthClient();
```

Exemple :

```ts
// Toujours obtenir un accessToken valide (auto-refresh si expiré)
const { accessToken } = await authClient.getAccessToken({
  providerId: "google",
});

// Tu peux maintenant l’utiliser pour appeler l’API Google
const res = await fetch("https://www.googleapis.com/drive/v3/files", {
  headers: { Authorization: `Bearer ${accessToken}` },
});
const files = await res.json();
console.log(files);
```

***

## ⚙️ 3. Bonnes pratiques

✅ Stocker `refreshToken` en base (dans la table `account` → Better Auth le gère déjà).\
✅ Utiliser **toujours** `authClient.getAccessToken()` pour garantir un token valide.\
✅ Ne pas exposer `refreshToken` côté client → toujours gérer le refresh côté serveur.

## 🚀 Flow complet Google Drive avec Better Auth

### 1️⃣ Configuration côté serveur (`auth.ts`)

On ajoute **Google provider** avec `offline access` et refresh auto des tokens :

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID as string,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET as string,
      accessType: "offline", // 🔑 toujours récupérer un refreshToken
      prompt: "select_account consent",
      scope: [
        "openid",
        "email",
        "profile",
        "https://www.googleapis.com/auth/drive.file", // accès Google Drive limité aux fichiers créés par l’app
      ],
      refreshAccessToken: async (token) => {
        const response = await fetch("https://oauth2.googleapis.com/token", {
          method: "POST",
          headers: { "Content-Type": "application/x-www-form-urlencoded" },
          body: new URLSearchParams({
            client_id: process.env.GOOGLE_CLIENT_ID!,
            client_secret: process.env.GOOGLE_CLIENT_SECRET!,
            grant_type: "refresh_token",
            refresh_token: token.refreshToken!,
          }),
        });

        const newTokens = await response.json();

        if (!response.ok) {
          throw new Error(
            `Google token refresh failed: ${newTokens.error_description}`
          );
        }

        return {
          accessToken: newTokens.access_token,
          refreshToken: newTokens.refresh_token ?? token.refreshToken,
        };
      },
    },
  },
});
```

***

### 2️⃣ Client Auth (`auth-client.ts`)

```ts
import { createAuthClient } from "better-auth/client";

export const authClient = createAuthClient();
```

***

### 3️⃣ Utiliser Google Drive avec accessToken

👉 Exemple React : **Lister les fichiers d’un utilisateur**

```tsx
"use client";

import { authClient } from "@/lib/auth-client";
import { useState } from "react";

export default function GoogleDriveFiles() {
  const [files, setFiles] = useState<any[]>([]);

  const listFiles = async () => {
    const { accessToken } = await authClient.getAccessToken({
      providerId: "google",
    });

    const res = await fetch(
      "https://www.googleapis.com/drive/v3/files?pageSize=10&fields=files(id,name,mimeType,webViewLink)",
      {
        headers: {
          Authorization: `Bearer ${accessToken}`,
        },
      }
    );

    const data = await res.json();
    setFiles(data.files || []);
  };

  return (
    <div>
      <button
        onClick={listFiles}
        className="px-4 py-2 bg-blue-500 text-white rounded"
      >
        Lister mes fichiers Google Drive
      </button>

      <ul className="mt-4">
        {files.map((file) => (
          <li key={file.id}>
            <a
              href={file.webViewLink}
              target="_blank"
              rel="noreferrer"
              className="text-blue-600 underline"
            >
              {file.name} ({file.mimeType})
            </a>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

***

### 4️⃣ Upload d’un fichier vers Google Drive

```ts
const uploadFile = async () => {
  const { accessToken } = await authClient.getAccessToken({
    providerId: "google",
  });

  const metadata = {
    name: "example.txt",
    mimeType: "text/plain",
  };

  const fileContent = new Blob(["Hello Google Drive!"], {
    type: "text/plain",
  });

  const form = new FormData();
  form.append(
    "metadata",
    new Blob([JSON.stringify(metadata)], { type: "application/json" })
  );
  form.append("file", fileContent);

  const res = await fetch(
    "https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart",
    {
      method: "POST",
      headers: { Authorization: `Bearer ${accessToken}` },
      body: form,
    }
  );

  const data = await res.json();
  console.log("File uploaded:", data);
};
```

***

### 5️⃣ Télécharger un fichier depuis Google Drive

```ts
const downloadFile = async (fileId: string) => {
  const { accessToken } = await authClient.getAccessToken({
    providerId: "google",
  });

  const res = await fetch(
    `https://www.googleapis.com/drive/v3/files/${fileId}?alt=media`,
    {
      headers: { Authorization: `Bearer ${accessToken}` },
    }
  );

  const blob = await res.blob();
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = "downloaded-file.txt";
  a.click();
};
```

***

## ✅ Résumé

* `auth.ts` → config Google avec refresh automatique.
* `authClient.getAccessToken` → garantit toujours un token valide.
* `fetch` → appels directs aux API Google Drive.
* Fonctions prêtes : **lister, uploader, télécharger** fichiers.

## Mini module

## 📂 `lib/googleDrive.ts`

```ts
import { authClient } from "@/lib/auth-client";

/**
 * Récupère un access token Google valide (rafraîchi si besoin)
 */
async function getGoogleAccessToken() {
  const { accessToken } = await authClient.getAccessToken({
    providerId: "google",
  });

  if (!accessToken) {
    throw new Error("Impossible de récupérer un token Google valide.");
  }

  return accessToken;
}

/**
 * Liste les fichiers Google Drive (10 premiers par défaut)
 */
export async function listFiles() {
  const accessToken = await getGoogleAccessToken();

  const res = await fetch(
    "https://www.googleapis.com/drive/v3/files?pageSize=10&fields=files(id,name,mimeType,webViewLink)",
    {
      headers: { Authorization: `Bearer ${accessToken}` },
    }
  );

  if (!res.ok) {
    throw new Error(`Erreur Google Drive: ${res.statusText}`);
  }

  const data = await res.json();
  return data.files || [];
}

/**
 * Upload un fichier texte sur Google Drive
 */
export async function uploadFile(
  filename: string,
  content: string,
  mimeType: string = "text/plain"
) {
  const accessToken = await getGoogleAccessToken();

  const metadata = {
    name: filename,
    mimeType,
  };

  const fileContent = new Blob([content], { type: mimeType });

  const form = new FormData();
  form.append(
    "metadata",
    new Blob([JSON.stringify(metadata)], { type: "application/json" })
  );
  form.append("file", fileContent);

  const res = await fetch(
    "https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart",
    {
      method: "POST",
      headers: { Authorization: `Bearer ${accessToken}` },
      body: form,
    }
  );

  if (!res.ok) {
    throw new Error(`Erreur Upload: ${res.statusText}`);
  }

  return res.json();
}

/**
 * Télécharge un fichier depuis Google Drive
 */
export async function downloadFile(fileId: string): Promise<Blob> {
  const accessToken = await getGoogleAccessToken();

  const res = await fetch(
    `https://www.googleapis.com/drive/v3/files/${fileId}?alt=media`,
    {
      headers: { Authorization: `Bearer ${accessToken}` },
    }
  );

  if (!res.ok) {
    throw new Error(`Erreur Download: ${res.statusText}`);
  }

  return res.blob();
}
```

***

## 📌 Exemple d’utilisation dans un composant React

```tsx
"use client";

import { useState } from "react";
import { listFiles, uploadFile, downloadFile } from "@/lib/googleDrive";

export default function GoogleDriveDemo() {
  const [files, setFiles] = useState<any[]>([]);

  const handleList = async () => {
    const f = await listFiles();
    setFiles(f);
  };

  const handleUpload = async () => {
    await uploadFile("example.txt", "Hello depuis Better Auth 🚀");
    alert("Fichier uploadé !");
  };

  const handleDownload = async (fileId: string) => {
    const blob = await downloadFile(fileId);
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "downloaded.txt";
    a.click();
  };

  return (
    <div className="space-y-4">
      <button onClick={handleList} className="px-4 py-2 bg-blue-500 text-white rounded">
        📂 Lister mes fichiers
      </button>

      <button onClick={handleUpload} className="px-4 py-2 bg-green-500 text-white rounded">
        ⬆️ Uploader un fichier
      </button>

      <ul>
        {files.map((file) => (
          <li key={file.id} className="flex items-center space-x-2">
            <a
              href={file.webViewLink}
              target="_blank"
              rel="noreferrer"
              className="text-blue-600 underline"
            >
              {file.name}
            </a>
            <button
              onClick={() => handleDownload(file.id)}
              className="px-2 py-1 bg-gray-200 rounded"
            >
              ⬇️ Télécharger
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

***

👉 Maintenant tu as un **module centralisé `googleDrive.ts`** qui gère tout, et un **exemple React prêt à tester**.

un **mini module `auth.ts`** basé sur **BetterAuth**, qui gère **authentification** (login / sign up / sessions sécurisées) et **autorisation** (rôles & accès).

L’idée :

* Authentification robuste : email + password, Google OAuth.
* Sessions sécurisées avec cookies, refresh & revoke.
* Autorisation via champ `role` (user / admin) avec validation côté serveur.

***

## 📂 `lib/auth.ts`

```ts
import { betterAuth } from "better-auth";
import Database from "better-sqlite3"; // Exemple (peut être remplacé par PostgreSQL/MySQL)
import { customSession } from "better-auth/plugins";

// DB simple (adapter selon ton projet : Prisma, Kysely, etc.)
const db = new Database("db.sqlite");

export const auth = betterAuth({
  database: db,

  // ✅ Gestion Email + Password
  emailAndPassword: {
    enabled: true,
    minPasswordLength: 8,
    requireEmailVerification: true,
    sendResetPassword: async ({ user, url }) => {
      console.log(`📧 Password reset for ${user.email} → ${url}`);
    },
    onPasswordReset: async ({ user }) => {
      console.log(`✅ Password reset for ${user.email}`);
    },
  },

  // ✅ Social login (Google)
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      scope: ["email", "profile"],
      prompt: "select_account",
      accessType: "offline",
    },
  },

  // ✅ Sessions sécurisées
  session: {
    expiresIn: 60 * 60 * 24 * 7, // 7 jours
    updateAge: 60 * 60 * 24, // refresh chaque jour
    cookieCache: {
      enabled: true,
      maxAge: 300, // 5 min
    },
  },

  // ✅ Autorisation basique (champ role)
  user: {
    additionalFields: {
      role: { type: "string", input: false }, // user, admin, etc.
    },
  },

  // ✅ Ajout des rôles dans la session
  plugins: [
    customSession(async ({ user, session }) => {
      return {
        user: {
          ...user,
          role: user.role || "user",
        },
        session,
      };
    }),
  ],

  // ✅ Rate limiting
  rateLimit: {
    enabled: true,
    window: 60,
    max: 100,
    customRules: {
      "/sign-in/email": { window: 10, max: 3 }, // bruteforce protection
    },
  },
});
```

***

## 📂 `lib/auth-client.ts`

```ts
import { createAuthClient } from "better-auth/client";
import { inferAdditionalFields } from "better-auth/client/plugins";
import type { auth } from "./auth";

export const authClient = createAuthClient({
  plugins: [inferAdditionalFields<typeof auth>()],
});
```

***

## 📌 Middleware d’autorisation (`middleware.ts`)

```ts
import { NextRequest, NextResponse } from "next/server";
import { auth } from "@/lib/auth";

export async function middleware(req: NextRequest) {
  const session = await auth.api.getSession({ headers: req.headers });

  // 🔒 Bloquer si non connecté
  if (!session?.user) {
    return NextResponse.redirect(new URL("/login", req.url));
  }

  // 🔒 Exemple : bloquer si non-admin
  if (req.nextUrl.pathname.startsWith("/admin")) {
    if (session.user.role !== "admin") {
      return NextResponse.redirect(new URL("/403", req.url));
    }
  }

  return NextResponse.next();
}
```

***

## 📌 Exemple d’utilisation (React)

```tsx
"use client";

import { authClient } from "@/lib/auth-client";

export default function Dashboard() {
  const { data: session } = authClient.useSession();

  if (!session) return <p>⏳ Loading...</p>;

  return (
    <div>
      <h1>Bienvenue {session.user.name}</h1>
      <p>Rôle : {session.user.role}</p>
      {session.user.role === "admin" && (
        <button className="bg-red-500 text-white px-3 py-1 rounded">
          🔒 Action réservée aux admins
        </button>
      )}
    </div>
  );
}
```

***

✅ Résultat :

* Auth robuste (sessions sécurisées, reset password, login social, rate limit).
* Autorisation basique par rôle (`user` / `admin`).
* Middleware Next.js protège les routes sensibles.

**RBAC (Role-Based Access Control)** dans ton module BetterAuth.\
On va gérer **plusieurs rôles** (`user`, `manager`, `admin`) et des **permissions fines** (CRUD sur des ressources).

***

## 📂 `lib/auth.ts` (avec RBAC)

```ts
import { betterAuth } from "better-auth";
import Database from "better-sqlite3";
import { customSession } from "better-auth/plugins";

const db = new Database("db.sqlite");

// 🔑 Définition des rôles et permissions
const rolePermissions: Record<string, string[]> = {
  user: ["read:profile"],
  manager: ["read:profile", "edit:profile", "manage:projects"],
  admin: ["*"], // accès total
};

export const auth = betterAuth({
  database: db,

  // ✅ Email + Password
  emailAndPassword: {
    enabled: true,
    requireEmailVerification: true,
    sendResetPassword: async ({ user, url }) => {
      console.log(`📧 Password reset for ${user.email} → ${url}`);
    },
  },

  // ✅ Social Login (Google)
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      prompt: "select_account",
    },
  },

  // ✅ Session
  session: {
    expiresIn: 60 * 60 * 24 * 7, // 7 jours
    cookieCache: { enabled: true, maxAge: 300 },
  },

  // ✅ Champ rôle
  user: {
    additionalFields: {
      role: { type: "string", input: false }, // user, manager, admin
    },
  },

  // ✅ Ajout permissions dans la session
  plugins: [
    customSession(async ({ user, session }) => {
      const role = user.role || "user";
      const permissions = rolePermissions[role] || [];

      return {
        user: {
          ...user,
          role,
          permissions,
        },
        session,
      };
    }),
  ],
});
```

***

## 📂 `lib/auth-client.ts`

```ts
import { createAuthClient } from "better-auth/client";
import { inferAdditionalFields } from "better-auth/client/plugins";
import type { auth } from "./auth";

export const authClient = createAuthClient({
  plugins: [inferAdditionalFields<typeof auth>()],
});
```

***

## 📌 Middleware d’autorisation (`middleware.ts`)

```ts
import { NextRequest, NextResponse } from "next/server";
import { auth } from "@/lib/auth";

export async function middleware(req: NextRequest) {
  const session = await auth.api.getSession({ headers: req.headers });

  if (!session?.user) {
    return NextResponse.redirect(new URL("/login", req.url));
  }

  // Exemple : restreindre /admin aux admins uniquement
  if (req.nextUrl.pathname.startsWith("/admin")) {
    if (session.user.role !== "admin") {
      return NextResponse.redirect(new URL("/403", req.url));
    }
  }

  return NextResponse.next();
}
```

***

## 📌 Helper d’autorisation (`lib/permissions.ts`)

```ts
import type { Session } from "@/lib/auth";

export function hasPermission(session: Session | null, permission: string) {
  if (!session?.user) return false;

  const { permissions } = session.user;
  return permissions.includes("*") || permissions.includes(permission);
}
```

***

## 📌 Exemple d’utilisation (React)

```tsx
"use client";

import { authClient } from "@/lib/auth-client";
import { hasPermission } from "@/lib/permissions";

export default function Dashboard() {
  const { data: session } = authClient.useSession();

  if (!session) return <p>⏳ Chargement...</p>;

  return (
    <div>
      <h1>Bienvenue {session.user.name}</h1>
      <p>Rôle : {session.user.role}</p>

      {hasPermission(session, "manage:projects") && (
        <button className="bg-blue-500 text-white px-3 py-1 rounded">
          📂 Gérer les projets
        </button>
      )}

      {hasPermission(session, "edit:profile") ? (
        <button className="bg-green-500 text-white px-3 py-1 rounded">
          ✏️ Modifier mon profil
        </button>
      ) : (
        <p className="text-gray-500">Tu n’as pas la permission d’éditer ton profil</p>
      )}
    </div>
  );
}
```

***

✅ Résultat :

* **Rôles gérés dynamiquement** (`user`, `manager`, `admin`).
* **Permissions injectées dans la session** (`read:profile`, `manage:projects`, etc.).
* Middleware + helper `hasPermission` pour contrôler l’accès aux pages et actions.
* Système extensible : tu peux facilement ajouter d’autres permissions.
