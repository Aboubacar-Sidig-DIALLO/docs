---
description: >-
  React est une bibliothèque JavaScript qui sert à créer et afficher des
  interfaces utilisateur (UI).
---

# 🎨⚛️Décrire l’UI en React

👉 Une interface utilisateur, c’est tout ce que tu vois à l’écran :

* un **bouton** sur lequel tu cliques,
* un **texte** qui s’affiche,
* une **image** qui illustre ton contenu,
* ou même des éléments plus complexes comme un **formulaire** ou un **tableau**.

En résumé : **chaque petit morceau visuel de ton appli est une partie de l’UI**.

***

### La magie de React ✨

Ce que fait React, c’est prendre tous ces petits morceaux (boutons, textes, images, etc.) et te permettre de les organiser sous forme de **composants (**&#x55;n composant peut être minuscule "un simple bouton", Ou énorme "une page entière"**)** :

* 🔹 **Réutilisables** → tu peux employer le même bouton ou la même carte plusieurs fois dans ton app.
* 🔹 **Imbriqués** → tu peux mettre un composant dans un autre (par exemple, une `Card` qui contient une `Image` et un `Titre`).

Que tu développes :

* un **site web**,
* ou une **application mobile**,

👉 **tout ton écran peut être décomposé en composants**.

***

### 🌟 Qu’est-ce qu’un composant en React ?

Un **composant** est comme une **brique de LEGO** 🧩 pour construire ton interface (UI).

* Avec le HTML, tu as déjà des briques toutes faites : `<h1>`, `<p>`, `<ul>`, etc.
* Avec React, tu peux fabriquer **tes propres briques** personnalisées : `<Profile />`, `<Gallery />`, `<Button />`, etc.

Chaque composant :\
✅ est **réutilisable** (tu l’écris une fois, tu l’utilises partout)\
✅ est **composable** (tu peux les imbriquer les uns dans les autres)\
✅ combine **markup (HTML)**, **style (CSS)** et **logique (JS)** dans un même bloc

***

### 🏗 Exemple avec HTML pur

Sans React, pour afficher un article avec un titre et une liste, tu écris :

```html
<article>
  <h1>Mon Premier Composant</h1>
  <ol>
    <li>Introduction aux composants</li>
    <li>Définir un composant</li>
    <li>Utiliser un composant</li>
  </ol>
</article>
```

C’est bien, mais ça reste figé.\
👉 Avec React, tu vas transformer ça en **composant réutilisable**.

***

### ⚛️ Définir ton premier composant React

En React, un composant est juste une **fonction JavaScript** qui retourne du JSX (un mélange de HTML + JS).

Exemple :

```jsx
export default function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3Am.jpg"
      alt="Katherine Johnson"
    />
  );
}
```

#### Étape par étape :

1. **Export**\
   `export default` → permet d’utiliser ce composant ailleurs.
2. **Définition de la fonction**\
   `function Profile() { ... }` → c’est une fonction classique JS.\
   ⚠️ Son **nom doit commencer par une majuscule** (`Profile`) sinon React le prend pour une balise HTML.
3. **Return JSX**\
   Le `return (...)` contient du **JSX** : ça ressemble à du HTML, mais ça reste du JavaScript.

👉 Résultat : quand tu utilises `<Profile />`, ça affiche l’image de Katherine Johnson.

***

### 🎨 Utiliser ton composant

Une fois ton composant créé, tu peux l’utiliser comme une balise HTML personnalisée :

```jsx
function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

👉 Ici :

* `<section>` et `<h1>` = balises HTML natives
* `<Profile />` = ton composant personnalisé
* Chaque `<Profile />` affiche la même image (réutilisation ♻️)

Le navigateur, lui, ne voit que du HTML classique :

```html
<section>
  <h1>Amazing scientists</h1>
  <img src="..." alt="Katherine Johnson" />
  <img src="..." alt="Katherine Johnson" />
  <img src="..." alt="Katherine Johnson" />
</section>
```

***

### 👨‍👩‍👧 Parent & enfants

* **Gallery** est un **parent**
* **Profile** est un **enfant**\
  Un parent peut **réutiliser** ses enfants autant qu’il veut.

⚠️ **Erreur à éviter** : ne définis jamais un composant _à l’intérieur_ d’un autre, sinon ça casse les performances et crée des bugs.\
Toujours définir les composants **au niveau supérieur** (top-level).

***

### 📦 Organisation des composants

* Tu peux garder plusieurs petits composants dans un même fichier.
* Si ça devient trop gros → sépare-les dans des fichiers différents (`Profile.js`, `Gallery.js`, etc.).

***

### 🚀 Petit challenge pour toi

Corrige ce composant qui ne marche pas :

```jsx
function Profile() {
  return (
    <img
      src="https://i.imgur.com/lICfvbD.jpg"
      alt="Aklilu Lemma"
    />
  );
}
```

👉 Solution : il manque **l’export** !

```jsx
export default function Profile() {
  return (
    <img
      src="https://i.imgur.com/lICfvbD.jpg"
      alt="Aklilu Lemma"
    />
  );
}
```

***

✨ Voilà, tu sais écrire ton premier composant React. À partir de là, tu peux construire **toute une interface** rien qu’en assemblant des briques (tes composants).
