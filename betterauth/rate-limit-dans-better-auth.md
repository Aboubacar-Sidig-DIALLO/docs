# 🚦 Rate Limit dans Better Auth

### 📌 Définition

Le **Rate Limit** (limitation de débit) permet de restreindre le nombre de requêtes qu’un utilisateur/une IP peut envoyer sur une période donnée.\
👉 Objectif : **protéger contre les abus**, comme le spam, le brute force sur les mots de passe, ou encore les attaques DDoS légères.

***

### ⚙️ **Configuration par défaut**

En **production**, Better Auth active un rate limiter global avec les valeurs suivantes :

* **Fenêtre (`window`)** : `60` secondes
* **Max requêtes (`max`)** : `100` par fenêtre

➡️ Un client ne peut donc pas faire plus de 100 requêtes par minute.

⚠️ **Important** :

* Les appels **server-side avec `auth.api`** (depuis ton backend) **ne sont pas affectés** ✅.
* Le rate limit s’applique uniquement aux **requêtes initiées par les clients (navigateur, mobile, etc.)**.

***

### 🛠️ **Personnalisation**

Tu peux modifier les règles globales en configurant `rateLimit` dans ton `auth.ts` :

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  rateLimit: {
    window: 10,   // durée de la fenêtre en secondes
    max: 100,     // nombre max de requêtes dans la fenêtre
  },
});
```

***

### 🧑‍💻 **Activer en développement**

Par défaut, le rate limit est **désactivé en mode développement** pour éviter de bloquer les tests.\
👉 Pour l’activer manuellement :

```ts
export const auth = betterAuth({
  rateLimit: {
    enabled: true,
    window: 20,
    max: 50,
  },
});
```

***

### 🎯 **Règles spécifiques**

Better Auth applique des **restrictions renforcées** sur certains endpoints sensibles, par défaut :

* **`/sign-in/email`** → max **3 requêtes / 10 sec**\
  (évite le brute force sur les mots de passe)
* **`/two-factor/verify`** _(plugin 2FA)_ → max **3 requêtes / 10 sec**\
  (évite les attaques par essai de codes OTP)

⚡ Ces règles s’appliquent **en plus** des règles globales.

***

### 🔐 **Avantages**

* ✅ Protection **anti-brute-force**
* ✅ Prévention **d’abus d’API**
* ✅ Paramétrable **globalement** et **par endpoint**
* ✅ Plugins ajoutent **leurs propres règles de sécurité**

## ⚡ Configurer le Rate Limit dans Better Auth

Le **Rate Limit** repose sur l’**adresse IP du client** pour suivre le nombre de requêtes effectuées dans une fenêtre de temps donnée.\
👉 Tu peux ajuster cette logique avec une grande précision : choix des headers IP, fenêtres personnalisées, règles par endpoint, etc.

***

### 🌍 Détection de l’adresse IP

Par défaut, Better Auth lit l’IP depuis le header :

```http
x-forwarded-for
```

➡️ Standard dans de nombreux reverse proxies (NGINX, Vercel, etc.).

⚠️ Mais si tu utilises un service comme **Cloudflare**, tu devras préciser leur en-tête dédié, par ex. `cf-connecting-ip` :

```ts
export const auth = betterAuth({
  advanced: {
    ipAddress: {
      ipAddressHeaders: ["cf-connecting-ip"], // 🌩️ Cloudflare
    },
  },
  rateLimit: {
    enabled: true,
    window: 60, // durée en secondes
    max: 100,   // max requêtes
  },
});
```

***

### ⏳ Configurer la fenêtre globale

Exemple classique → max 100 requêtes par minute :

```ts
export const auth = betterAuth({
  rateLimit: {
    window: 60, // fenêtre en secondes
    max: 100,   // max requêtes dans la fenêtre
  },
});
```

***

### 🎯 Règles personnalisées par chemin

Tu peux définir des règles **spécifiques à certains endpoints** (ex. limiter fortement `/sign-in/email`).

```ts
export const auth = betterAuth({
  rateLimit: {
    window: 60,
    max: 100,
    customRules: {
      "/sign-in/email": {
        window: 10, // fenêtre de 10 sec
        max: 3,     // max 3 essais de connexion
      },
      "/two-factor/*": async (request) => {
        // règle dynamique (par exemple stricte pour OTP)
        return {
          window: 10,
          max: 3,
        };
      },
    },
  },
});
```

***

### 🚫 Désactiver le rate limit pour un chemin

Tu peux désactiver totalement la limitation sur un endpoint (utile pour des routes peu sensibles comme `/get-session`) :

```ts
export const auth = betterAuth({
  rateLimit: {
    customRules: {
      "/get-session": false, // ❌ pas de rate limit sur cette route
    },
  },
});
```

***

### ✅ Récap des bonnes pratiques

1. **Limiter fortement les endpoints sensibles**
   * `/sign-in/email` → anti brute-force
   * `/two-factor/verify` → OTP sécurisé
2. **Désactiver pour les endpoints safe**
   * `/get-session` ou `/health-check`
3. **Adapter les headers IP selon ton infra**
   * `x-forwarded-for` (NGINX, Vercel, Render…)
   * `cf-connecting-ip` (Cloudflare)
4. **Mixer règles globales + règles spécifiques**\
   → plus de souplesse selon le type de requête

## 🗄️ Stockage du Rate Limit dans Better Auth

Par défaut → les données de rate limit sont gardées **en mémoire locale**.\
👉 Problème : en **serverless** (Vercel, AWS Lambda, Cloudflare Workers…), chaque instance a sa propre mémoire → donc le tracking des requêtes n’est pas partagé → ❌ inefficace contre les abus.

Better Auth permet donc 3 solutions :

***

### 1️⃣ Stockage en **Base de données**

Tu peux stocker les compteurs de rate limit dans ta DB relationnelle.

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  rateLimit: {
    storage: "database",
    modelName: "rateLimit", // (optionnel, par défaut "rateLimit")
  },
});
```

➡️ Exécute une migration pour créer la table rateLimit :

```bash
npx @better-auth/cli migrate
```

👉 Avantage : persistance et partage entre instances.\
👉 Inconvénient : peut être plus lent (selon ta DB et ton trafic).

***

### 2️⃣ Stockage en **Secondary Storage** (ex. Redis)

Si tu as configuré un `secondaryStorage` (par ex. Redis, Memcached, KV Store), tu peux demander à Better Auth de l’utiliser directement pour le rate limiting :

```ts
export const auth = betterAuth({
  rateLimit: {
    storage: "secondary-storage",
  },
});
```

✅ Recommandé en prod → Redis = rapide, distribué, parfait pour du rate limiting en cluster/serverless.

***

### 3️⃣ Stockage **Custom** (implémentation perso)

Si tu veux stocker les compteurs ailleurs (ex. DynamoDB, MongoDB, API externe), tu peux définir un `customStorage` :

```ts
export const auth = betterAuth({
  rateLimit: {
    customStorage: {
      get: async (key) => {
        // 🔎 Récupère les données de rate limit (par ex. depuis DynamoDB/Redis)
      },
      set: async (key, value) => {
        // 💾 Enregistre les données (compteur + TTL)
      },
    },
  },
});
```

👉 Avantage : totale flexibilité.\
👉 Exemple : dans une infra multi-régions, tu pourrais stocker ça dans **Cloudflare KV** ou **AWS DynamoDB Global Tables**.

***

## 🚀 Bonnes pratiques de stockage Rate Limit

* 🔐 **Toujours persister** (DB, Redis, custom) en production serverless.
* ⚡ Utiliser Redis (ou KV store rapide) pour les environnements à **fort trafic**.
* 🛡️ Mettre des règles strictes pour les endpoints sensibles (`/sign-in`, `/two-factor/verify`).
* 📊 Monitorer → tu peux loguer les hits/bloqués pour détecter les attaques.

## 🚦 Gestion des erreurs Rate Limit dans Better Auth

Quand le client dépasse la limite → **Better Auth renvoie une erreur HTTP 429** + un header :

```
X-Retry-After: nombre de secondes à attendre
```

***

### 1️⃣ Gestion **globale** (pour tout le client)

Tu peux intercepter toutes les erreurs via `fetchOptions.onError` :

```ts
// auth-client.ts
import { createAuthClient } from "better-auth/client";

export const authClient = createAuthClient({
  fetchOptions: {
    onError: async (context) => {
      const { response } = context;
      if (response.status === 429) {
        const retryAfter = response.headers.get("X-Retry-After");
        console.warn(`⏳ Trop de requêtes. Réessaye dans ${retryAfter} secondes.`);
        
        // 👉 Exemple UX : afficher un toast
        alert(`Vous avez dépassé la limite. Réessayez dans ${retryAfter}s.`);
      }
    },
  },
});
```

✅ Utile pour **uniformiser les erreurs de rate limit** dans toute ton app (sign-in, reset password, etc.).

***

### 2️⃣ Gestion **au cas par cas** (par requête)

Si tu veux gérer **seulement certains appels sensibles** (ex : `/sign-in/email`) :

```ts
import { authClient } from "./auth-client";

await authClient.signIn.email({
  email: "test@example.com",
  password: "password123",
}, {
  fetchOptions: {
    onError: async (context) => {
      const { response } = context;
      if (response.status === 429) {
        const retryAfter = response.headers.get("X-Retry-After");
        console.warn(`⏳ Attendez ${retryAfter} secondes avant de réessayer.`);
      }
    },
  },
});
```

***

### 3️⃣ UX / Bonnes pratiques

* 🔄 **Désactiver les boutons** concernés pendant `retryAfter` secondes (empêche spam).
* ⏳ Afficher un **compte à rebours** ("Réessayez dans 9s…").
* 🛡️ Pour les endpoints sensibles (`/sign-in`, `/two-factor/verify`) → limites très basses (`3 req / 10s`).

***

### 4️⃣ 📦 Schema DB (si tu stockes en Database)

Si tu utilises une **base relationnelle** pour stocker les limites, la table `rateLimit` doit ressembler à ça :

| Field       | Type    | Key | Description                                    |
| ----------- | ------- | --- | ---------------------------------------------- |
| id          | string  | PK  | Identifiant unique (UUID, Snowflake, etc.)     |
| key         | string  | -   | Identifiant unique du rate limit (IP, userId…) |
| count       | integer | -   | Nombre de requêtes effectuées                  |
| lastRequest | bigint  | -   | Timestamp du dernier appel (ms ou s)           |

👉 Better Auth gère ça automatiquement si tu choisis `storage: "database"`.
