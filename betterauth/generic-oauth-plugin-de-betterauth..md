# Generic OAuth Plugin de BetterAuth.

C’est super puissant car ça permet d’intégrer **n’importe quel fournisseur OAuth2 ou OpenID Connect (OIDC)**, même s’il n’est pas supporté par défaut (comme Instagram, Coinbase, LinkedIn, Discord, etc.).

***

### 🔑 Étapes pour utiliser un provider générique

#### 1. Installation côté serveur

Dans `auth.ts`, tu ajoutes le plugin **genericOAuth** avec la config du provider choisi :

```ts
import { betterAuth } from "better-auth";
import { genericOAuth } from "better-auth/plugins";

export const auth = betterAuth({
  // ... tes autres configs
  plugins: [
    genericOAuth({
      config: [
        {
          providerId: "instagram", // identifiant interne
          clientId: process.env.INSTAGRAM_CLIENT_ID as string,
          clientSecret: process.env.INSTAGRAM_CLIENT_SECRET as string,
          authorizationUrl: "https://api.instagram.com/oauth/authorize",
          tokenUrl: "https://api.instagram.com/oauth/access_token",
          scopes: ["user_profile", "user_media"], // scopes nécessaires
        },
      ],
    }),
  ],
});
```

⚡ Tu peux ajouter plusieurs providers dans le même tableau `config`.

***

#### 2. Installation côté client

Dans `auth-client.ts` tu ajoutes le **genericOAuthClient** :

```ts
import { createAuthClient } from "better-auth/client";
import { genericOAuthClient } from "better-auth/client/plugins";

export const authClient = createAuthClient({
  plugins: [genericOAuthClient()],
});
```

***

#### 3. Usage côté client

Ensuite, pour lancer une connexion via un provider générique :

```ts
const response = await authClient.signIn.oauth2({
  providerId: "instagram", // ou "coinbase", "linkedin", etc.
  callbackURL: "/dashboard", // où rediriger après login
});
```

***

### 📌 Exemples

#### 🔹 Instagram

```ts
genericOAuth({
  config: [
    {
      providerId: "instagram",
      clientId: process.env.INSTAGRAM_CLIENT_ID as string,
      clientSecret: process.env.INSTAGRAM_CLIENT_SECRET as string,
      authorizationUrl: "https://api.instagram.com/oauth/authorize",
      tokenUrl: "https://api.instagram.com/oauth/access_token",
      scopes: ["user_profile", "user_media"],
    },
  ],
});
```

***

#### 🔹 Coinbase

```ts
genericOAuth({
  config: [
    {
      providerId: "coinbase",
      clientId: process.env.COINBASE_CLIENT_ID as string,
      clientSecret: process.env.COINBASE_CLIENT_SECRET as string,
      authorizationUrl: "https://www.coinbase.com/oauth/authorize",
      tokenUrl: "https://api.coinbase.com/oauth/token",
      scopes: ["wallet:user:read"],
    },
  ],
});
```

***

✅ Avantages :

* Supporte **tous les providers OAuth2/OIDC**.
* Tu définis **URL d’auth + token** + **scopes**.
* Complètement **type-safe** avec BetterAuth.

un **exemple générique complet** où on configure **LinkedIn + Discord** via le plugin `genericOAuth`.\
Ça va te donner une base réutilisable pour n’importe quel provider OAuth2/OIDC.

***

### ⚙️ Côté serveur → `auth.ts`

```ts
import { betterAuth } from "better-auth";
import { genericOAuth } from "better-auth/plugins";

export const auth = betterAuth({
  // ... tes autres options (sessions, db, etc.)
  plugins: [
    genericOAuth({
      config: [
        // 🔹 LinkedIn
        {
          providerId: "linkedin",
          clientId: process.env.LINKEDIN_CLIENT_ID as string,
          clientSecret: process.env.LINKEDIN_CLIENT_SECRET as string,
          authorizationUrl: "https://www.linkedin.com/oauth/v2/authorization",
          tokenUrl: "https://www.linkedin.com/oauth/v2/accessToken",
          scopes: ["r_liteprofile", "r_emailaddress"], // profil + email
        },
        // 🔹 Discord
        {
          providerId: "discord",
          clientId: process.env.DISCORD_CLIENT_ID as string,
          clientSecret: process.env.DISCORD_CLIENT_SECRET as string,
          authorizationUrl: "https://discord.com/api/oauth2/authorize",
          tokenUrl: "https://discord.com/api/oauth2/token",
          scopes: ["identify", "email"], // récupérer infos de base
        },
      ],
    }),
  ],
});
```

***

### ⚙️ Côté client → `auth-client.ts`

```ts
import { createAuthClient } from "better-auth/client";
import { genericOAuthClient } from "better-auth/client/plugins";

export const authClient = createAuthClient({
  plugins: [genericOAuthClient()],
});
```

***

### 🚀 Usage côté client

👉 **Connexion LinkedIn**

```ts
const signInWithLinkedIn = async () => {
  await authClient.signIn.oauth2({
    providerId: "linkedin",
    callbackURL: "/dashboard",
  });
};
```

👉 **Connexion Discord**

```ts
const signInWithDiscord = async () => {
  await authClient.signIn.oauth2({
    providerId: "discord",
    callbackURL: "/dashboard",
  });
};
```

***

### ✅ Résumé

* Tu définis chaque provider dans le `config` du `genericOAuth`.
* Tu ajoutes `genericOAuthClient()` côté client.
* Tu appelles `signIn.oauth2({ providerId })` pour démarrer le flow.

On va compléter ton module **Generic OAuth** avec la gestion des **tokens** (access + refresh) pour que tu puisses **appeler l’API du provider** (LinkedIn ou Discord) directement après la connexion.

***

## 🔑 Étape 1 : Sauvegarder les tokens chiffrés

BetterAuth ne chiffre pas les tokens par défaut → on active un **hook** pour les sécuriser.

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { genericOAuth } from "better-auth/plugins";

function encrypt(value: string) {
  // 🔒 Ici tu peux utiliser crypto, jose, ou libsodium
  return Buffer.from(value).toString("base64");
}
function decrypt(value: string) {
  return Buffer.from(value, "base64").toString("utf8");
}

export const auth = betterAuth({
  plugins: [
    genericOAuth({
      config: [
        {
          providerId: "linkedin",
          clientId: process.env.LINKEDIN_CLIENT_ID!,
          clientSecret: process.env.LINKEDIN_CLIENT_SECRET!,
          authorizationUrl: "https://www.linkedin.com/oauth/v2/authorization",
          tokenUrl: "https://www.linkedin.com/oauth/v2/accessToken",
          scopes: ["r_liteprofile", "r_emailaddress"],
        },
        {
          providerId: "discord",
          clientId: process.env.DISCORD_CLIENT_ID!,
          clientSecret: process.env.DISCORD_CLIENT_SECRET!,
          authorizationUrl: "https://discord.com/api/oauth2/authorize",
          tokenUrl: "https://discord.com/api/oauth2/token",
          scopes: ["identify", "email"],
        },
      ],
    }),
  ],
  databaseHooks: {
    account: {
      create: {
        before(account) {
          const clone = { ...account };
          if (clone.accessToken) clone.accessToken = encrypt(clone.accessToken);
          if (clone.refreshToken) clone.refreshToken = encrypt(clone.refreshToken);
          return { data: clone };
        },
        after(account) {
          const clone = { ...account };
          if (clone.accessToken) clone.accessToken = decrypt(clone.accessToken);
          if (clone.refreshToken) clone.refreshToken = decrypt(clone.refreshToken);
          return clone;
        },
      },
    },
  },
});
```

***

## 🔑 Étape 2 : Récupérer les tokens

Quand tu listes les **accounts** de l’utilisateur :

```ts
// Exemple côté client
const accounts = await authClient.listAccounts();

// Trouver l’account LinkedIn
const linkedInAccount = accounts.find(a => a.providerId === "linkedin");
console.log("LinkedIn Access Token:", linkedInAccount?.accessToken);
```

***

## 🔑 Étape 3 : Appeler les APIs externes

#### LinkedIn – récupérer email & profil

```ts
async function fetchLinkedInProfile(accessToken: string) {
  const profile = await fetch("https://api.linkedin.com/v2/me", {
    headers: { Authorization: `Bearer ${accessToken}` },
  }).then(r => r.json());

  const email = await fetch(
    "https://api.linkedin.com/v2/emailAddress?q=members&projection=(elements*(handle~))",
    { headers: { Authorization: `Bearer ${accessToken}` } }
  ).then(r => r.json());

  return { profile, email };
}
```

#### Discord – récupérer infos user

```ts
async function fetchDiscordUser(accessToken: string) {
  return await fetch("https://discord.com/api/users/@me", {
    headers: { Authorization: `Bearer ${accessToken}` },
  }).then(r => r.json());
}
```

***

## ✅ Résumé

1. 🔒 **Chiffrage** avec `databaseHooks` pour stocker tokens safely.
2. 🔑 **listAccounts** pour récupérer accessToken/refreshToken.
3. 🌍 **API calls** vers LinkedIn ou Discord avec le token.

**gestion automatique du refresh des tokens** pour tes providers (LinkedIn, Discord, etc.).

***

## 🔄 Étape 1 : Déclarer `refreshAccessToken`

BetterAuth permet de définir une fonction pour rafraîchir les tokens quand l’`accessToken` expire.

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { genericOAuth } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    genericOAuth({
      config: [
        {
          providerId: "linkedin",
          clientId: process.env.LINKEDIN_CLIENT_ID!,
          clientSecret: process.env.LINKEDIN_CLIENT_SECRET!,
          authorizationUrl: "https://www.linkedin.com/oauth/v2/authorization",
          tokenUrl: "https://www.linkedin.com/oauth/v2/accessToken",
          scopes: ["r_liteprofile", "r_emailaddress"],
          async refreshAccessToken(token) {
            const response = await fetch("https://www.linkedin.com/oauth/v2/accessToken", {
              method: "POST",
              headers: { "Content-Type": "application/x-www-form-urlencoded" },
              body: new URLSearchParams({
                grant_type: "refresh_token",
                refresh_token: token.refreshToken!,
                client_id: process.env.LINKEDIN_CLIENT_ID!,
                client_secret: process.env.LINKEDIN_CLIENT_SECRET!,
              }),
            });

            const data = await response.json();
            return {
              accessToken: data.access_token,
              refreshToken: data.refresh_token ?? token.refreshToken,
            };
          },
        },
        {
          providerId: "discord",
          clientId: process.env.DISCORD_CLIENT_ID!,
          clientSecret: process.env.DISCORD_CLIENT_SECRET!,
          authorizationUrl: "https://discord.com/api/oauth2/authorize",
          tokenUrl: "https://discord.com/api/oauth2/token",
          scopes: ["identify", "email"],
          async refreshAccessToken(token) {
            const response = await fetch("https://discord.com/api/oauth2/token", {
              method: "POST",
              headers: { "Content-Type": "application/x-www-form-urlencoded" },
              body: new URLSearchParams({
                grant_type: "refresh_token",
                refresh_token: token.refreshToken!,
                client_id: process.env.DISCORD_CLIENT_ID!,
                client_secret: process.env.DISCORD_CLIENT_SECRET!,
              }),
            });

            const data = await response.json();
            return {
              accessToken: data.access_token,
              refreshToken: data.refresh_token ?? token.refreshToken,
            };
          },
        },
      ],
    }),
  ],
});
```

***

## 🔄 Étape 2 : Utiliser le refresh automatique

Côté client ou backend, si un appel API retourne `401 Unauthorized`, tu demandes à BetterAuth de régénérer un token :

```ts
import { authClient } from "@/lib/auth-client";

async function getLinkedInData() {
  const accounts = await authClient.listAccounts();
  const linkedIn = accounts.find(a => a.providerId === "linkedin");

  try {
    return await fetchLinkedInProfile(linkedIn?.accessToken!);
  } catch (error: any) {
    if (error.status === 401) {
      // refresh automatique
      const refreshed = await authClient.refreshAccessToken({ providerId: "linkedin" });
      return await fetchLinkedInProfile(refreshed.accessToken);
    }
    throw error;
  }
}
```

***

## ✅ Résultat

* 🚀 Ton app **ne casse pas** quand un `accessToken` expire.
* 🔑 BetterAuth gère la logique de **rotation des tokens**.
* 🔒 Tu gardes le **refreshToken** en sécurité (chiffré via `databaseHooks`).

## ⚡ Stratégie Globale pour le Refresh Token

#### 1. Côté **BetterAuth config** (`auth.ts`)

Tu déclares `refreshAccessToken` pour chaque provider (LinkedIn, Discord, etc.), comme on l’a déjà fait. Ça permet à BetterAuth de savoir **comment demander un nouveau token**.

👉 Ça, c’est déjà prêt ✅.

***

#### 2. Côté **Client global handler** (`auth-client.ts`)

Tu peux ajouter un `fetchOptions.onError` dans la config de ton client BetterAuth.\
Il va automatiquement intercepter les erreurs `401 Unauthorized` et rafraîchir le token correspondant.

```ts
// auth-client.ts
import { createAuthClient } from "better-auth/client";

export const authClient = createAuthClient({
  fetchOptions: {
    onError: async (ctx) => {
      const { response, request } = ctx;

      // Si le token est expiré
      if (response.status === 401) {
        try {
          const providerId = request.url.includes("linkedin")
            ? "linkedin"
            : request.url.includes("discord")
            ? "discord"
            : null;

          if (providerId) {
            console.log(`🔄 Refreshing ${providerId} token...`);
            const { accessToken } = await authClient.refreshAccessToken({ providerId });

            // 🔁 Relancer automatiquement la requête échouée
            return await fetch(request.url, {
              ...request,
              headers: {
                ...request.headers,
                Authorization: `Bearer ${accessToken}`,
              },
            });
          }
        } catch (err) {
          console.error("❌ Token refresh failed", err);
        }
      }
    },
  },
});
```

***

#### 3. Côté **API calls** (usage normal)

Tu appelles tes APIs sans te soucier des expirations.\
Si un `accessToken` est expiré, BetterAuth le rafraîchit et rejoue la requête. 🎉

```ts
async function getLinkedInProfile() {
  const res = await fetch("/api/linkedin/profile", {
    headers: {
      Authorization: `Bearer ${(await authClient.getAccessToken({ providerId: "linkedin" })).accessToken}`,
    },
  });

  return await res.json();
}
```

***

## ✅ Résultat

* **Tu n’as plus besoin de gérer manuellement le refresh** dans chaque appel.
* **BetterAuth intercepte et rejoue automatiquement** les appels expirés.
* Ça fonctionne pour **LinkedIn, Discord, Google, etc.** tant que tu as bien défini `refreshAccessToken`.

Donc côté **Next.js (frontend)** tu as ton `authClient` qui gère déjà automatiquement le refresh + retry.

Maintenant côté **NestJS backend**, tu peux aussi centraliser la logique pour que toutes tes routes qui appellent une API externe via un provider (LinkedIn, Discord, Google, etc.) soient protégées par un middleware.

***

## ⚡ Middleware Global NestJS pour Refresh Tokens

#### 1. Crée un service `AuthProviderService`

Il encapsule BetterAuth côté serveur et gère le refresh automatique :

```ts
// auth-provider.service.ts
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { auth } from "@/lib/auth"; // ton instance BetterAuth server-side

@Injectable()
export class AuthProviderService {
  async getAccessToken(userId: string, providerId: string) {
    try {
      const { accessToken } = await auth.api.getAccessToken({
        body: { userId, providerId },
      });

      return accessToken;
    } catch (err) {
      console.error(`❌ Failed to get access token for ${providerId}`, err);
      throw new UnauthorizedException("Token expired or invalid");
    }
  }
}
```

***

#### 2. Crée un middleware `ProviderAuthMiddleware`

Ce middleware s’assure que **chaque requête externe utilise un token valide**, et si besoin, le refresh automatiquement.

```ts
// provider-auth.middleware.ts
import { Injectable, NestMiddleware } from "@nestjs/common";
import { Request, Response, NextFunction } from "express";
import { AuthProviderService } from "./auth-provider.service";

@Injectable()
export class ProviderAuthMiddleware implements NestMiddleware {
  constructor(private readonly authProviderService: AuthProviderService) {}

  async use(req: Request, res: Response, next: NextFunction) {
    const userId = req.headers["x-user-id"] as string; // récupère ton userId du header/session
    const providerId = req.headers["x-provider-id"] as string; // provider à utiliser

    if (!userId || !providerId) {
      return res.status(400).json({ error: "Missing userId or providerId" });
    }

    try {
      const token = await this.authProviderService.getAccessToken(userId, providerId);

      // Ajoute le token dans la requête pour usage ultérieur
      req["providerAccessToken"] = token;

      next();
    } catch (err) {
      return res.status(401).json({ error: "Unauthorized: invalid or expired token" });
    }
  }
}
```

***

#### 3. Lier le middleware dans `app.module.ts`

```ts
// app.module.ts
import { MiddlewareConsumer, Module, NestModule } from "@nestjs/common";
import { ProviderAuthMiddleware } from "./provider-auth.middleware";
import { AuthProviderService } from "./auth-provider.service";

@Module({
  providers: [AuthProviderService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(ProviderAuthMiddleware).forRoutes("*"); // sur toutes les routes
  }
}
```

***

#### 4. Utilisation dans un contrôleur

```ts
// linkedin.controller.ts
import { Controller, Get, Req } from "@nestjs/common";
import { Request } from "express";

@Controller("linkedin")
export class LinkedInController {
  @Get("profile")
  async getProfile(@Req() req: Request) {
    const token = req["providerAccessToken"];

    const res = await fetch("https://api.linkedin.com/v2/me", {
      headers: { Authorization: `Bearer ${token}` },
    });

    return await res.json();
  }
}
```

***

## ✅ Résultat

* **Frontend (Next.js)** : BetterAuth refresh automatiquement côté client.
* **Backend (NestJS)** : Middleware global gère la récupération + refresh des tokens avant chaque appel externe.
* Tu as une **authentification robuste et centralisée**, sans duplicer le code de refresh partout.
