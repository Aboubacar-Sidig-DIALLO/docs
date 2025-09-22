# 🌟 Bonnes pratiques pour travailler avec les variables d’environnement dans Docker Compose

### 🔐 1. Gérez les informations sensibles en toute sécurité

⚠️ Soyez prudent lorsque vous incluez des données sensibles (mots de passe, clés API, tokens, etc.) dans des variables d’environnement.\
👉 Préférez utiliser les **Secrets** de Docker pour gérer ce type d’informations de manière sécurisée.

***

### ⚖️ 2. Comprenez la **priorité des variables d’environnement**

📌 Familiarisez-vous avec la manière dont Docker Compose gère la **précédence** des variables provenant de différentes sources :

* fichiers `.env`,
* variables du shell,
* définitions dans le `Dockerfile`.

Cela vous évitera des comportements inattendus.

***

### 📂 3. Utilisez des fichiers d’environnement spécifiques

Adaptez votre configuration aux différents contextes (développement, test, production).\
👉 Créez des fichiers `.env` séparés (`.env.dev`, `.env.test`, `.env.prod`) et chargez-les en fonction de l’environnement ciblé.

***

### 🔄 4. Maîtrisez l’**interpolation**

Apprenez à utiliser l’**interpolation des variables** dans vos fichiers `compose.yaml`.\
👉 Cela rend vos configurations plus **dynamiques et flexibles**, en vous permettant de réutiliser vos fichiers tout en changeant uniquement les valeurs nécessaires.

***

### 🖥️ 5. Utilisez les surcharges en ligne de commande

N’oubliez pas que vous pouvez **surcharger temporairement** des variables lors du démarrage de vos conteneurs :

```bash
docker compose up -e VARIABLE=valeur_temporaire
```

👉 Très utile pour **tester rapidement** ou appliquer des **changements ponctuels** sans modifier vos fichiers `.env` ou `compose.yaml`.

***

✅ En résumé :

* **Sécurisez vos secrets**,
* **comprenez la hiérarchie des variables**,
* **séparez vos environnements**,
* **utilisez l’interpolation intelligemment**,
* et **profitez des surcharges CLI** pour plus de flexibilité.
