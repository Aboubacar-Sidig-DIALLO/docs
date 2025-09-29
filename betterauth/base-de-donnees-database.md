# 🗄️ Base de données (Database)

BetterAuth a besoin d’une **connexion à une base de données** pour fonctionner correctement.\
👉 La base est utilisée pour stocker toutes les **données liées à l’authentification** :

* 👤 **Utilisateurs** (users)
* 🔑 **Sessions**
* 📩 Données liées aux inscriptions / vérifications email
* 🔌 **Plugins** → certains plugins ajoutent leurs **propres tables** (ex: 2FA, organisations, etc.)

***

### 📌 Utilisation d’un adaptateur (Adapter)

Pour connecter votre base de données, vous devez fournir à BetterAuth une **instance de base supportée** via l’option **`database`**.

BetterAuth fournit déjà des adaptateurs pour les bases relationnelles les plus courantes (et d’autres via plugins).

***

#### Exemple : passer une instance DB

```ts
// auth.ts
import { betterAuth } from "better-auth"
import Database from "better-sqlite3"

export const auth = betterAuth({
  database: new Database("./sqlite.db"), // 👉 connexion SQLite
})
```

***

### 📌 Adapters supportés

BetterAuth inclut un système d’**adapters** pour différents types de bases :

* **Bases relationnelles** (SQLite, PostgreSQL, MySQL, etc.)
* **ORMs** populaires comme :
  * Prisma
  * Drizzle
  * Kysely
* Et d’autres connecteurs listés dans la documentation des **Other Relational Databases**.

👉 Chaque adaptateur sait comment traduire les besoins de BetterAuth en **tables, migrations et requêtes** adaptées à votre base.

***

### 🚀 Points forts

* ⚡ **Flexibilité** → vous choisissez votre base (relationnelle ou via ORM).
* 🔌 **Extensible** → les plugins peuvent ajouter leurs propres schémas.
* 🧩 **Intégration fluide** → BetterAuth s’adapte à votre stack existante.
* 🔒 **Centralisation des données** → tous les utilisateurs et sessions sont stockés dans une base unique et sécurisée.

***

👉 En résumé :\
BetterAuth a besoin d’une **base de données connectée via un adapter** pour stocker utilisateurs, sessions et données d’auth.\
Vous pouvez utiliser une **DB native** (SQLite, Postgres, MySQL…) ou un **ORM moderne** (Prisma, Drizzle, Kysely, etc.) 🎉.

## 🛠️ CLI – Gestion de la base de données

BetterAuth est livré avec un outil **CLI** permettant de gérer :

* 📦 les **migrations de base de données**,
* 📑 la **génération du schéma** nécessaire à l’authentification.

***

### 📌 Lancer les migrations (Running Migrations)

La CLI vérifie automatiquement votre base et vous propose :

* ➕ d’**ajouter les tables manquantes**,
* 🔄 ou de **mettre à jour les tables existantes** avec les nouvelles colonnes nécessaires.

👉 **Important** : cette fonctionnalité est uniquement supportée avec l’**adaptateur intégré Kysely**.

Pour les autres adaptateurs (**Prisma, Drizzle, etc.**) → utilisez la commande `generate` (voir ci-dessous), puis appliquez la migration avec l’outil de votre ORM.

```bash
npx @better-auth/cli migrate
```

***

### 📌 Générer le schéma (Generating Schema)

BetterAuth fournit aussi une commande **`generate`** pour créer le schéma requis.

* Avec un adaptateur ORM (**Prisma, Drizzle**) → génère automatiquement un **fichier de schéma** compatible avec l’ORM.
* Avec l’adaptateur intégré **Kysely** → génère un **fichier SQL** exécutable directement dans la base.

```bash
npx @better-auth/cli generate
```

***

### 📌 Schéma manuel

Si vous préférez gérer vos tables **manuellement**, vous pouvez le faire ✅.

* Le **schéma de base requis** par BetterAuth est documenté dans la section **Core Schema**.
* Les **plugins** (comme 2FA, organisations, etc.) définissent aussi leurs propres tables, disponibles dans leur documentation respective.

***

### 🚀 Points forts

* ⚡ **Automatisation** des migrations (Kysely).
* 🧑‍💻 **Compatibilité ORM** → Prisma, Drizzle, etc.
* 🔌 **Extensible** → support des tables additionnelles pour plugins.
* 📑 **Flexibilité** → choix entre génération automatique ou création manuelle.

***

👉 En résumé :\
La **CLI BetterAuth** simplifie la gestion de la base en proposant la **migration automatique (Kysely)** ou la **génération de schéma (Prisma, Drizzle, etc.)**.\
Vous pouvez aussi opter pour une gestion **manuelle**, en suivant les schémas fournis 🎉.

## 🗄️ Secondary Storage (Stockage secondaire)

Le **secondary storage** (stockage secondaire) dans BetterAuth permet d’utiliser des **stockages clé-valeur rapides** (ex. Redis, Memcached, en RAM…) pour gérer :

* 🔑 les **sessions** utilisateurs
* ⏱️ les **compteurs de rate limiting**
* 📊 d’autres données temporaires et intensives

👉 L’idée : déléguer le stockage de ces données **à forte charge** vers une solution **rapide et optimisée en mémoire**, plutôt que de tout stocker dans la base relationnelle.

***

### ⚙️ Interface à implémenter

Pour activer un stockage secondaire, vous devez implémenter l’interface **`SecondaryStorage`** :

```ts
interface SecondaryStorage {
  get: (key: string) => Promise<unknown>; 
  set: (key: string, value: string, ttl?: number) => Promise<void>;
  delete: (key: string) => Promise<void>;
}
```

👉 Les trois méthodes essentielles :

* **`get(key)`** → récupérer une valeur via sa clé
* **`set(key, value, ttl?)`** → enregistrer une valeur avec option TTL (durée de vie ⏳)
* **`delete(key)`** → supprimer une valeur

***

### 📌 Intégration dans BetterAuth

Vous fournissez ensuite votre implémentation directement dans la config :

```ts
betterAuth({
  // ... autres options
  secondaryStorage: {
    // Implémentation personnalisée ici
  },
});
```

***

### 🚀 Exemple concret : Redis

Redis est le **candidat parfait** pour un secondary storage, car il est ultra rapide et supporte le TTL nativement 🔥.

```ts
import { createClient } from "redis";
import { betterAuth } from "better-auth";

const redis = createClient();
await redis.connect();

export const auth = betterAuth({
  // ... autres options
  secondaryStorage: {
    get: async (key) => {
      return await redis.get(key);
    },
    set: async (key, value, ttl) => {
      if (ttl) await redis.set(key, value, { EX: ttl }); 
      // Pour ioredis :
      // if (ttl) await redis.set(key, value, 'EX', ttl)
      else await redis.set(key, value);
    },
    delete: async (key) => {
      await redis.del(key);
    },
  },
});
```

***

### 🎯 Avantages du Secondary Storage

* ⚡ **Performance** : Redis/Memcached > DB relationnelle pour les accès fréquents.
* 🔒 **Séparation claire** : sessions et limites stockées ailleurs → DB principale allégée.
* ⏱️ **TTL automatique** → pratique pour expirer les sessions et limiter les abus (rate limiting).
* 🧩 **Flexibilité** : vous pouvez ajouter des **préfixes de clés** (`auth:session:123`, `auth:rate:ip`) pour mieux organiser vos données.

***

### 🧐 Bonnes pratiques

* Utiliser un **TTL court** pour les sessions en cache → éviter une surcharge inutile.
* Ajouter un **préfixe** aux clés pour éviter les collisions (`better-auth:session:`).
* Sécuriser Redis (mot de passe, réseau privé) → car il contient des **données sensibles**.

***

👉 En résumé :\
Le **Secondary Storage** permet à BetterAuth de déléguer le stockage des **sessions** et du **rate limiting** vers un système optimisé (Redis, RAM, etc.).\
Cela améliore **les performances** ⚡ et soulage votre **base principale** 🗄️.

## 🗄️ Core Schema (Schéma de base)

BetterAuth repose sur un **ensemble minimal de tables** qui doivent exister dans votre base.\
👉 Ces tables permettent de gérer les **utilisateurs**, les **sessions**, ainsi que toutes les **relations essentielles à l’authentification**.

Les types sont donnés en **TypeScript**, mais dans la pratique vous utiliserez les types équivalents dans votre **SGBD (Postgres, MySQL, SQLite, …)** ou via un **ORM (Prisma, Drizzle, Kysely, …)**.

***

### 📌 Tables principales attendues

1. **Users** → informations sur chaque utilisateur 👤
2. **Sessions** → suivi des connexions actives 🔑
3. **Accounts** → gestion des connexions sociales (OAuth, etc.) 🌍
4. **VerificationTokens** → stockage des tokens temporaires (emails, magic links, reset password) ✉️

***

### 📝 Exemple en TypeScript (types logiques)

#### 👤 Table `users`

```ts
type User = {
  id: string;            // Identifiant unique (UUID recommandé)
  email: string;         // Adresse email unique
  password?: string;     // Hash du mot de passe (optionnel si login social)
  name?: string;         // Nom d’affichage de l’utilisateur
  image?: string;        // URL de l’avatar / photo de profil
  emailVerified?: Date;  // Date de vérification de l’email
  createdAt: Date;       // Date de création
  updatedAt: Date;       // Dernière mise à jour
}
```

***

#### 🔑 Table `sessions`

```ts
type Session = {
  id: string;        // Identifiant unique de la session
  userId: string;    // FK vers users.id
  expiresAt: Date;   // Date d’expiration
  createdAt: Date;   // Date de création
  updatedAt: Date;   // Dernière mise à jour
}
```

***

#### 🌍 Table `accounts` (login social)

```ts
type Account = {
  id: string;            // Identifiant unique
  userId: string;        // FK vers users.id
  provider: string;      // ex: "google", "github"
  providerAccountId: string; // ID du compte côté provider
  accessToken?: string;  // Jeton d’accès (optionnel, selon provider)
  refreshToken?: string; // Jeton de rafraîchissement (optionnel)
  expiresAt?: Date;      // Expiration du jeton (optionnel)
}
```

***

#### ✉️ Table `verificationTokens`

```ts
type VerificationToken = {
  id: string;        // Identifiant unique
  token: string;     // Jeton aléatoire généré
  identifier: string; // Email ou identifiant lié au token
  expiresAt: Date;   // Expiration du token
}
```

***

### 🚀 Points importants

* 📑 **Minimal mais extensible** → ce schéma est le socle, mais les **plugins** (2FA, organisations, etc.) ajoutent leurs propres tables.
* 🔑 **Relations principales** :
  * `users` ↔ `sessions` (1 utilisateur → plusieurs sessions)
  * `users` ↔ `accounts` (1 utilisateur → plusieurs comptes sociaux)
* 🔒 **Sécurité intégrée** → mots de passe hashés, tokens temporaires expirables.

***

👉 En résumé :\
Le **Core Schema** de BetterAuth définit les **tables essentielles** pour gérer **utilisateurs, sessions, comptes sociaux et tokens de vérification**.\
C’est la **colonne vertébrale de l’authentification** dans votre application 🔐.
