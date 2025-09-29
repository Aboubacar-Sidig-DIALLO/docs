# 🛠️ Session Management avec Better Auth

Better Auth utilise une **gestion de session basée sur des cookies** (approche classique et sécurisée) :

* Lorsqu’un utilisateur **se connecte**, Better Auth crée une **session en DB** (ou storage choisi).
* Le serveur **génère un cookie signé** qui contient l’ID ou le token de session.
* À chaque requête, le cookie est automatiquement renvoyé par le navigateur → le serveur le **valide** et récupère l’utilisateur associé.
* Si la session est valide → l’utilisateur reste connecté. Sinon → session invalide ou expirée.

***

### ⚙️ Fonctionnement

1. **Création de session**\
   → Se produit après un login (`signIn`).\
   → Stocke les infos principales :
   * `id` (unique)
   * `userId` (lié à l’utilisateur)
   * `expiresAt` (expiration)
   * `createdAt`
2. **Stockage dans un cookie**
   * Le cookie est signé avec ton `secret` Better Auth.
   * Nom du cookie → dépend du `cookiePrefix` configuré.
   * Peut être configuré en **Secure**, **HttpOnly**, **SameSite**, etc.
3. **Validation à chaque requête**
   * Le serveur lit le cookie.
   * Vérifie la session en DB (ou Redis si secondary storage).
   * Si valide → l’utilisateur est authentifié.
   * Si expirée → rejet avec erreur 401.

***

### ✅ Avantages de cette approche

* 🔒 **Sécurité** : cookies signés, HttpOnly, support `Secure`.
* 🌍 **Compatibilité universelle** : fonctionne sur SSR, API routes, SPAs, etc.
* 🕒 **Expiration configurable** : tu peux définir la durée de vie des sessions.
* ⚡ **Support multi-domaines** (via `crossSubDomainCookies`).

***

### Exemple de récupération de session

Côté **client** :

```ts
import { authClient } from "./auth-client";

const session = await authClient.getSession();
console.log(session?.user); // Données utilisateur si connecté
```

Côté **serveur** (API route, middleware Next.js/Nest.js) :

```ts
import { auth } from "./auth";

export default async function handler(req, res) {
  const session = await auth.api.getSession({ headers: req.headers });
  if (!session) {
    res.status(401).json({ error: "Not authenticated" });
    return;
  }
  res.json({ user: session.user });
}
```

## 🗂️ Table `session` (Better Auth)

La table **`session`** est indispensable pour la gestion des connexions utilisateur.\
Chaque enregistrement correspond à **une session active** (un "login").

#### 📌 Champs

| Champ       | Type          | Description                                                                                                               |
| ----------- | ------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `id`        | string (PK)   | **Le token de session**. C’est lui qui est stocké dans le cookie (`session_token`) et utilisé pour identifier la session. |
| `userId`    | string (FK)   | **Lien vers l’utilisateur** associé.                                                                                      |
| `expiresAt` | date/datetime | Date d’expiration de la session. Après cette date, la session est invalide.                                               |
| `ipAddress` | string        | L’**adresse IP** utilisée lors de la création de la session.                                                              |
| `userAgent` | string        | Le **User-Agent HTTP** (navigateur, OS, device) au moment de la connexion.                                                |

***

#### ⚙️ Exemple SQL (PostgreSQL)

```sql
CREATE TABLE session (
  id TEXT PRIMARY KEY,
  userId TEXT NOT NULL REFERENCES "user"(id) ON DELETE CASCADE,
  expiresAt TIMESTAMPTZ NOT NULL,
  ipAddress TEXT,
  userAgent TEXT
);
```

***

#### 🔐 Utilisation

* **id** → stocké dans un cookie signé (`session_token`).
* **userId** → permet de retrouver l’utilisateur associé.
* **expiresAt** → gère la durée de vie de la session (`expiresIn` dans la config Better Auth).
* **ipAddress** & **userAgent** → utiles pour la **sécurité** (détection de sessions suspectes, multi-device, logs).

***

👉 Dans Better Auth, tu peux aussi configurer :

* ⏳ `expiresIn` → combien de temps dure une session.
* 🔄 `updateAge` → délai avant mise à jour automatique de `expiresAt`.
* ❌ invalidation → déconnexion manuelle (logout).
* 📱 multi-sessions → support de plusieurs connexions simultanées par utilisateur.

### 🔐 Session Expiration & Refresh (Better Auth)

#### ⚙️ Valeurs par défaut

* `expiresIn` → **7 jours** (en secondes)
* `updateAge` → **1 jour** (en secondes)

Cela veut dire que :

1. Une session dure **7 jours max**.
2. Chaque fois qu’un utilisateur fait une requête **et** que la dernière mise à jour de la session remonte à plus de 24h, Better Auth **rafraîchit `expiresAt`** → il repart à `now + expiresIn`.

***

#### 📌 Exemple de config personnalisée

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  session: {
    expiresIn: 60 * 60 * 24 * 14, // 14 jours
    updateAge: 60 * 60 * 6,       // 6 heures
  }
});
```

👉 Ici :

* La session est valide **14 jours**.
* Mais elle se "renouvelle" (rolling session) **toutes les 6 heures d’activité**.
* Résultat : tant que l’utilisateur est actif, il peut rester connecté indéfiniment.
* S’il est inactif pendant plus de 14 jours → il doit se reconnecter.

***

#### 🔎 Bonnes pratiques

* **App sécurisée grand public** : `expiresIn = 7 jours`, `updateAge = 1 jour`.
* **Application critique (admin, bancaire, santé)** : réduire (`expiresIn = 1 jour`, `updateAge = 1h`).
* **App "always logged in" type SaaS** : `expiresIn = 30 jours`, `updateAge = 12h`.

### 🔒 `disableSessionRefresh`

Par défaut, **Better Auth rafraîchit la session** (rolling session) :

* Si la session est utilisée **après `updateAge`** (ex. 24h).
* Alors → `expiresAt` est recalculé (`now + expiresIn`).
* Résultat : tant que l’utilisateur est actif, il ne se déconnecte jamais, sauf s’il reste inactif plus longtemps que `expiresIn`.

***

#### ⚡ Si `disableSessionRefresh = true`

* La session **expire strictement** après `expiresIn`.
* Même si l’utilisateur est actif → l’`expiresAt` ne bouge pas.
* Exemple : `expiresIn = 7 jours` → l’utilisateur sera déconnecté **après 7 jours exactement**, même s’il a utilisé l’app tous les jours.

***

#### 📌 Exemple concret

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  session: {
    expiresIn: 60 * 60 * 24 * 7,  // 7 jours
    updateAge: 60 * 60 * 24,      // 1 jour
    disableSessionRefresh: true   // désactive le rolling session
  }
});
```

👉 Ici :

* La session **expire en 7 jours pile**.
* `updateAge` est ignoré.

***

#### 🔎 Quand l’utiliser ?

* **Applications critiques** (banque, santé, back-office sensible).
* Cas où tu veux forcer une **reconnexion périodique stricte** pour des raisons de conformité (RGPD, sécurité interne).
* Situations où tu veux éviter qu’une session reste valide "indéfiniment" si l’utilisateur reste actif.

### 🔑 **Session Freshness**

#### 🟢 Définition

* Une session est **fraîche** si sa date de création (`createdAt`) est **inférieure à `freshAge`**.
* Par défaut : `freshAge = 1 jour (60 * 60 * 24)`.

👉 Cela signifie que certaines actions sensibles exigent que l’utilisateur ait créé/rénové sa session récemment.

***

#### ⚡ Exemple de configuration

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  session: {
    freshAge: 60 * 5 // 5 minutes
  }
});
```

➡️ Ici, une session est considérée **fraîche uniquement pendant 5 minutes après sa création**.\
Passé ce délai, l’utilisateur devra **se ré-authentifier** pour accéder à certaines actions sensibles (ex : changement d’email, mise à jour de mot de passe, suppression de compte…).

***

#### 🚫 Désactiver la vérification

Si tu veux désactiver complètement la notion de fraîcheur :

```ts
export const auth = betterAuth({
  session: {
    freshAge: 0 // pas de check de fraîcheur
  }
});
```

***

#### 🔎 Quand utiliser la fraîcheur ?

* **Opérations critiques** :
  * Modifier email ou mot de passe.
  * Suppression du compte.
  * Changement de 2FA / clés de sécurité.
* Pour **forcer une re-validation** de l’identité de l’utilisateur (même s’il a une session valide depuis longtemps).

***

👉 En résumé :

* **`expiresIn`** → durée totale de vie de la session.
* **`updateAge`** → rolling refresh (si activé).
* **`freshAge`** → "période de fraîcheur" exigée pour actions sensibles.

### ⚡ **Fonctions principales de gestion de session**

#### 1. **Récupérer la session courante**

```ts
import { authClient } from "@/lib/client";

const { data: session } = await authClient.getSession();
```

➡️ Retourne la session **active** du client courant.\
Pratique pour vérifier si l’utilisateur est connecté et récupérer son profil.

***

#### 2. **Session réactive (hook)**

```ts
import { authClient } from "@/lib/client";

const { data: session } = authClient.useSession();
```

➡️ Version **réactive** : met automatiquement à jour `session` si elle change (login/logout).\
Idéal pour afficher l’état de connexion dans le frontend.

***

#### 3. **Lister toutes les sessions actives**

```ts
import { authClient } from "@/lib/client";

const sessions = await authClient.listSessions();
```

➡️ Renvoie toutes les sessions actives de l’utilisateur (ordinateur, mobile, etc.).\
Pratique pour afficher une liste "Appareils connectés".

***

#### 4. **Révoquer une session spécifique**

```ts
import { authClient } from "@/lib/client";

await authClient.revokeSession({
  token: "session-token"
});
```

➡️ Permet à l’utilisateur de **déconnecter un appareil particulier** (utile en cas de vol ou de perte de téléphone).

***

#### 5. **Révoquer toutes les autres sessions**

```ts
import { authClient } from "@/lib/client";

await authClient.revokeOtherSessions();
```

➡️ Déconnecte l’utilisateur de tous les appareils sauf celui qu’il utilise actuellement.\
Parfait pour un bouton "Déconnecter partout".

***

### 🚀 Exemple d’intégration dans un tableau de bord

```tsx
import { authClient } from "@/lib/client";

export function SessionsManager() {
  const [sessions, setSessions] = React.useState<any[]>([]);

  React.useEffect(() => {
    authClient.listSessions().then(setSessions);
  }, []);

  const revoke = async (token: string) => {
    await authClient.revokeSession({ token });
    setSessions(sessions.filter(s => s.id !== token));
  };

  const revokeOthers = async () => {
    await authClient.revokeOtherSessions();
    setSessions(await authClient.listSessions());
  };

  return (
    <div>
      <h2>Appareils connectés</h2>
      <ul>
        {sessions.map((s) => (
          <li key={s.id}>
            {s.userAgent} - {s.ipAddress}
            <button onClick={() => revoke(s.id)}>Déconnecter</button>
          </li>
        ))}
      </ul>
      <button onClick={revokeOthers}>Déconnecter tous les autres</button>
    </div>
  );
}
```

### 🔐 **1. Révoquer toutes les sessions**

Si tu veux déconnecter l’utilisateur **de tous ses appareils en même temps** :

```ts
import { authClient } from "@/lib/client";

await authClient.revokeSessions();
```

👉 Cela supprime **toutes les sessions actives**, y compris celle en cours.\
(Utile pour un bouton "Se déconnecter partout".)

***

### 🔑 **2. Révoquer toutes les autres sessions lors d’un changement de mot de passe**

Quand un utilisateur change son mot de passe, il est souvent recommandé de **révoquer toutes les anciennes sessions** (sécurité en cas de compromission).

```ts
import { authClient } from "@/lib/client";

await authClient.changePassword({
  newPassword: "mon-nouveau-mot-de-passe",
  currentPassword: "mon-ancien-mot-de-passe",
  revokeOtherSessions: true,
});
```

➡️ Ici, seule la session **actuelle** reste valide, toutes les autres sont déconnectées.

***

### ⚡ Bonnes pratiques

* 🔒 Toujours révoquer toutes les sessions lors d’un **reset de mot de passe** (via email).
* 📱 Fournir un bouton "Se déconnecter partout" dans l’interface utilisateur (ex. "Sécurité du compte").
* 🕵️‍♂️ Coupler ça avec un **historique des connexions/sessions** (`listSessions`) pour donner de la transparence à l’utilisateur.

### 🚀 **Cookie Cache – Principe**

* Par défaut, `useSession` ou `getSession` → vont chercher la session en DB.
* Avec le **cookie cache activé**, Better Auth stocke temporairement les infos de session dans un **cookie signé** :
  * 🔑 Protégé contre la falsification.
  * ⏳ Expire rapidement (`maxAge`).
  * 🔄 Se rafraîchit régulièrement.

👉 Ça fonctionne comme un **JWT d’accès court-terme**, mais reste basé sur cookies (pas de stockage côté client).

***

### ⚙️ **Activer le Cookie Cache**

Dans ta config :

```ts
// auth.ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  session: {
    cookieCache: {
      enabled: true,
      maxAge: 5 * 60, // 5 minutes de cache
    },
  },
});
```

✅ Résultat : pendant 5 minutes, Better Auth lit directement depuis le cookie → **aucune requête DB**.

***

### 🔄 **Forcer un refresh (ignorer le cache)**

Si tu veux **forcer la récupération en DB** (par ex. si tu soupçonnes que la session a changé) :

#### 📱 Côté client :

```ts
const session = await authClient.getSession({
  query: { disableCookieCache: true },
});
```

#### 🖥️ Côté serveur :

```ts
await auth.api.getSession({
  query: { disableCookieCache: true },
  headers: req.headers, // indispensable pour la vérification
});
```

***

### ⚡ Quand utiliser ?

* ✅ Pour la majorité des requêtes : garder le cache actif (booste la perf).
* 🔄 Pour des actions critiques (sécurité, admin, changement de mot de passe) : forcer le refresh (`disableCookieCache`).

## 🔑 Customizing Session Response

### 🚀 But

Par défaut, les méthodes `getSession` et `useSession` renvoient un objet avec les infos de l’utilisateur (`user`) et de la session (`session`).\
Avec le plugin **`customSession`**, tu peux enrichir cette réponse (par exemple en ajoutant des rôles, permissions ou des champs supplémentaires).

***

### ⚙️ Configuration côté serveur

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { customSession } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    customSession(async ({ user, session }) => {
      // Exemple : récupérer les rôles depuis ta DB
      const roles = await findUserRoles(session.session.userId);

      return {
        roles,
        user: {
          ...user,
          newField: "newField", // champ personnalisé
        },
        session,
      };
    }),
  ],
});
```

***

### ⚡ Inférence côté client

Pour que TypeScript connaisse tes champs ajoutés (`roles`, `newField`), tu dois utiliser le plugin `customSessionClient` en liant ton client à ton instance serveur `auth`.

```ts
// auth-client.ts
import { createAuthClient } from "better-auth/react";
import { customSessionClient } from "better-auth/client/plugins";
import type { auth } from "@/lib/auth"; // ⚠️ importer comme type

export const authClient = createAuthClient({
  plugins: [
    customSessionClient<typeof auth>() // ✅ permet l’inférence
  ],
});
```

***

### 📱 Utilisation côté client

#### Avec `useSession`

```tsx
import { authClient } from "@/lib/auth-client";

export function UserInfo() {
  const { data: session } = authClient.useSession();

  if (!session) return <p>Not logged in</p>;

  return (
    <div>
      <p>User: {session.user.email}</p>
      <p>Roles: {session.roles.join(", ")}</p>
      <p>Custom Field: {session.user.newField}</p>
    </div>
  );
}
```

#### Avec `getSession`

```ts
const { data: sessionData } = await authClient.getSession();

console.log(sessionData.roles);        // ["admin", "editor", ...]
console.log(sessionData.user.newField); // "newField"
```

## ⚠️ Caveats on Customizing Session Response

Lorsque tu utilises le plugin `customSession`, il y a quelques points importants à connaître pour éviter les pièges.

***

### 🧩 1. Inférence TypeScript avec les plugins

👉 Problème :\
Les champs ajoutés par d’autres plugins (ex: `twoFactor`, `multiSession`, etc.) ne sont **pas automatiquement inférés** dans le `session` passé au callback.

👉 Solution :\
Tu peux contourner ça en créant un objet `options` typé avec `BetterAuthOptions` et en le transmettant au plugin `customSession`.\
De cette façon, les champs ajoutés par les autres plugins **et tes champs custom** seront bien inférés.

```ts
// auth.ts
import { betterAuth, BetterAuthOptions } from "better-auth";
import { customSession } from "better-auth/plugins";

const options = {
  //...tes autres options
  plugins: [
    //...autres plugins
  ]
} satisfies BetterAuthOptions;

export const auth = betterAuth({
  ...options,
  plugins: [
    ...(options.plugins ?? []),
    customSession(
      async ({ user, session }, ctx) => {
        // ✅ maintenant user et session incluent les champs des plugins + custom
        return {
          user,
          session
        };
      },
      options // ⚠️ passer options ici pour l’inférence
    ),
  ],
});
```

***

### 🏗️ 2. Séparation frontend / backend

👉 Si ton **frontend et backend sont dans des repos séparés** (monorepo ou projets distincts), tu ne peux pas importer directement ton `auth` serveur en tant que type.\
Dans ce cas, **l’inférence TypeScript côté client ne fonctionnera pas** pour les champs ajoutés par `customSession`.

📌 Solution possible :\
Tu dois alors **redéfinir manuellement les types** côté client (par exemple avec `zod` ou une interface TS), ou exposer ton schéma via une lib partagée.

***

### 🍪 3. Cache de session

⚠️ Attention :\
Le **session caching** (via **cookie cache** ou **secondary storage**) **n’inclut pas les champs custom**.

👉 Conséquence :\
À chaque fois que la session est récupérée (`getSession`, `useSession`), la fonction `customSession` sera appelée pour reconstruire la réponse personnalisée.

***

### 📱 4. Mutation du endpoint `/multi-session/list-device-sessions`

Si tu utilises le plugin **multi-session**, il expose un endpoint `/multi-session/list-device-sessions` pour lister les devices connectés.

Par défaut, `customSession` **ne modifie pas la réponse de ce endpoint**.\
Tu peux forcer cette mutation en passant l’option `shouldMutateListDeviceSessionsEndpoint: true`.

```ts
// auth.ts
import { customSession } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    customSession(
      async ({ user, session }, ctx) => {
        return { user, session };
      },
      {}, // pas besoin d’options custom ici
      { shouldMutateListDeviceSessionsEndpoint: true } // ✅ mutation activée
    ),
  ],
});
```

***

✅ En résumé :

* Passe `options` pour garder l’inférence TypeScript des plugins.
* Attention si ton front et back sont séparés : pas d’inférence automatique.
* Le cache de session n’inclut jamais tes champs custom.
* Active `shouldMutat`
