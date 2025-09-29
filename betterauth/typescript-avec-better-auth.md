# 🟦 TypeScript avec Better Auth

Better Auth est conçu pour être **type-safe** sur le **serveur** et le **client**.\
Tout est basé sur l’inférence, ce qui permet de conserver des types fiables, y compris lorsqu’on ajoute des champs personnalisés.

***

### ✅ 1. Configuration TypeScript

Active le mode strict dans ton projet pour bénéficier de l’inférence complète :

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true
  }
}
```

👉 Si tu ne peux pas activer `strict`, active au minimum :

```json
{
  "compilerOptions": {
    "strictNullChecks": true
  }
}
```

⚠️ Vérifie que `declaration` et `composite` **ne sont pas activés** pour éviter des problèmes d’inférence trop longue.

***

### 🧩 2. Inférer les types (client et serveur)

#### Côté client

Tu peux extraire les types `Session` grâce à `$Infer` :

```ts
// auth-client.ts
import { createAuthClient } from "better-auth/client"

const authClient = createAuthClient()

// ✅ Session inclut { user, session }
export type Session = typeof authClient.$Infer.Session
```

#### Côté serveur

Idem, côté serveur :

```ts
// auth.ts
import { betterAuth } from "better-auth"
import Database from "better-sqlite3"

export const auth = betterAuth({
  database: new Database("database.db"),
})

type Session = typeof auth.$Infer.Session
```

***

### 🏗️ 3. Ajouter des champs personnalisés (Additional Fields)

Tu peux enrichir le modèle `user` ou `session` avec `additionalFields`.\
Ces champs sont **automatiquement inférés** et disponibles partout (serveur + client).

```ts
import { betterAuth } from "better-auth"
import Database from "better-sqlite3"

export const auth = betterAuth({
  database: new Database("database.db"),
  user: {
    additionalFields: {
      role: {
        type: "string",
        input: false, // ⚠️ important pour éviter que l’utilisateur le modifie lui-même
      },
    },
  },
})

// ✅ le type Session inclura automatiquement `user.role`
type Session = typeof auth.$Infer.Session
```

#### ⚠️ Propriété `input`

* `input: true` (par défaut) → le champ peut être fourni par l’utilisateur (inscription, mise à jour).
* `input: false` → empêche l’utilisateur de définir ou modifier ce champ (ex: `role`, `isAdmin`).

👉 C’est **obligatoire** pour les champs sensibles comme `role`, `status`, `permissions`.

***

### 🔄 4. Inférence des champs personnalisés côté client

#### a) Projet **monorepo** (client + serveur dans le même projet)

Tu peux utiliser `inferAdditionalFields` pour importer automatiquement les champs depuis ton serveur.

```ts
import { inferAdditionalFields } from "better-auth/client/plugins";
import { createAuthClient } from "better-auth/react";
import type { auth } from "./auth"; // importe ton auth serveur

export const authClient = createAuthClient({
  plugins: [inferAdditionalFields<typeof auth>()],
});
```

#### b) Projet **séparé** (client et serveur distincts)

Tu dois déclarer manuellement les champs supplémentaires :

```ts
import { inferAdditionalFields } from "better-auth/client/plugins";

export const authClient = createAuthClient({
  plugins: [inferAdditionalFields({
    user: {
      role: {
        type: "string",
      },
    },
  })],
});
```

***

## 🚀 En résumé

* Active `strict` ou `strictNullChecks` pour tirer parti de TypeScript.
* Utilise `$Infer` pour extraire les types `Session` partout.
* Ajoute tes champs via `additionalFields` (pense à `input: false` pour les champs sensibles).
* Côté client :
  * **Monorepo** → `inferAdditionalFields<typeof auth>()`
  * **Repos séparés** → déclarer les champs manuellement.
