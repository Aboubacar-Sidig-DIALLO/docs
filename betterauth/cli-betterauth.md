# 🛠️ CLI BetterAuth

BetterAuth est livré avec un **outil en ligne de commande (CLI)** intégré.\
Cet outil vous aide à :

* 🗄️ gérer les **schémas de base de données**,
* ⚡ initialiser votre projet,
* 🔑 générer une **clé secrète** pour votre application,
* 🩺 obtenir des **informations de diagnostic** utiles.

***

### 📌 Commandes principales

#### 1. **Generate** – Générer le schéma

Cette commande crée le schéma requis par BetterAuth.

* Avec un ORM comme **Prisma** ou **Drizzle** → génère automatiquement le schéma adapté.
* Avec l’adaptateur **Kysely** intégré → génère un fichier SQL exécutable directement sur votre base.

```bash
npx @better-auth/cli@latest generate
```

**Options :**

* **`--output`** → chemin de sortie du schéma :
  * Prisma → `prisma/schema.prisma`
  * Drizzle → `schema.ts` (racine du projet)
  * Kysely → `schema.sql` (racine du projet)
* **`--config`** → chemin du fichier de config BetterAuth (`auth.ts` recherché par défaut dans `./`, `./utils`, `./lib`, ou sous `src/`).
* **`--yes`** → exécuter sans confirmation.

***

#### 2. **Migrate** – Appliquer le schéma

Cette commande applique directement le schéma BetterAuth dans la base de données.\
👉 Disponible uniquement avec l’adaptateur intégré **Kysely**.\
👉 Pour Prisma ou Drizzle, utilisez l’outil de migration de votre ORM.

```bash
npx @better-auth/cli@latest migrate
```

**Options :**

* **`--config`** → chemin du fichier de config (`auth.ts`).
* **`--yes`** → appliquer directement sans confirmation.

***

#### 3. **Init** – Initialiser un projet

Permet d’initialiser rapidement BetterAuth dans un projet.

```bash
npx @better-auth/cli@latest init
```

**Options :**

* **`--name`** → nom de votre application (par défaut = `package.json.name`).
* **`--framework`** → framework utilisé (actuellement supporté : **Next.js**).
* **`--plugins`** → plugins à installer (séparés par des virgules).
* **`--database`** → base de données (actuellement supportée : **SQLite**).
* **`--package-manager`** → gestionnaire de paquets (npm, pnpm, yarn, bun).

***

#### 4. **Info** – Informations système & configuration

Affiche des infos de diagnostic sur votre environnement et votre configuration BetterAuth.\
Très utile pour **déboguer** ou partager lors d’une demande de support.

```bash
npx @better-auth/cli@latest info
```

**Affiche :**

* **System** → OS, CPU, RAM, version de Node.js
* **Package Manager** → gestionnaire détecté + version
* **BetterAuth** → version + configuration (_les infos sensibles sont masquées_)
* **Frameworks** → frameworks détectés (Next.js, React, Vue, etc.)
* **Databases** → clients DB et ORMs utilisés (Prisma, Drizzle, etc.)

**Options :**

* **`--config`** → chemin du fichier `auth.ts`.
* **`--json`** → sortie JSON (utile pour logs ou usage programmatique).

**Exemples :**

```bash
# Usage simple
npx @better-auth/cli@latest info

# Spécifier un chemin custom
npx @better-auth/cli@latest info --config ./config/auth.ts

# Export en JSON
npx @better-auth/cli@latest info --json > auth-info.json
```

👉 Les données sensibles (secrets, API keys, URLs de DB) sont automatiquement remplacées par **\[REDACTED]** ✅.

***

#### 5. **Secret** – Générer une clé secrète

BetterAuth requiert une **clé secrète** (utilisée pour le hash et le chiffrement).\
Vous pouvez la générer facilement avec :

```bash
npx @better-auth/cli@latest secret
```

***

### ⚠️ Problèmes courants

#### ❌ Erreur : _Cannot find module X_

Cela signifie que la CLI n’arrive pas à résoudre certains modules dans votre fichier de config BetterAuth.

✅ Solutions possibles :

* Supprimez temporairement les **alias d’import** et utilisez des **chemins relatifs** dans `auth.ts`.
* Une fois la CLI exécutée, vous pouvez revenir aux alias.

***

### 🚀 Points forts du CLI BetterAuth

* 🧑‍💻 **DX améliorée** : gestion simple des schémas, migrations et config.
* 🔒 **Sécurité** : génération automatique de secret keys.
* 🩺 **Debug facile** : `info` fournit un diagnostic complet.
* ⚡ **Rapide** : évite la configuration manuelle fastidieuse.

***

👉 En résumé :\
La **CLI BetterAuth** est votre **couteau suisse** 🔧 pour initialiser, configurer, migrer et déboguer votre système d’authentification.\
Elle simplifie au maximum le travail côté dev et garantit une **installation rapide, sécurisée et fiable** 🎉.
