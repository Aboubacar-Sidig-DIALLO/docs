# 🚀 Afficher des listes dans React

En JavaScript, quand tu as un tableau de données, tu peux utiliser ses méthodes comme **`map()`** pour transformer chaque élément en quelque chose de nouveau.\
En React, on utilise cette logique pour transformer chaque élément en **composant JSX**.

### 🔹 Exemple simple avec `.map()`

Imaginons que tu as une liste de produits :

```js
const products = ['Pommes', 'Bananes', 'Oranges'];
```

Tu peux utiliser `.map()` pour générer une liste React :

```jsx
export default function ShoppingList() {
  const products = ['Pommes', 'Bananes', 'Oranges'];

  return (
    <ul>
      {products.map(product => (
        <li>{product}</li>
      ))}
    </ul>
  );
}
```

👉 Résultat :

* React parcourt chaque élément du tableau `products`.
* Pour chaque élément, il renvoie un `<li>`.
* Tu obtiens une liste affichée dynamiquement.

***

### 🔹 Exemple avec objets + props

Avec des objets, tu peux passer des données comme props à un composant enfant :

```js
const scientists = [
  { id: 1, name: 'Albert Einstein', field: 'Physique' },
  { id: 2, name: 'Marie Curie', field: 'Chimie' },
  { id: 3, name: 'Ada Lovelace', field: 'Maths' },
];
```

```jsx
function Scientist({ name, field }) {
  return <li>{name} — {field}</li>;
}

export default function ScientistList() {
  return (
    <ul>
      {scientists.map(scientist => (
        <Scientist 
          key={scientist.id} 
          name={scientist.name} 
          field={scientist.field} 
        />
      ))}
    </ul>
  );
}
```

👉 Ici :

* Chaque `Scientist` est un **composant enfant**.
* La prop `key` (unique par élément) permet à React de suivre chaque item (super important pour les performances et éviter des bugs).

***

### 🔹 Exemple avec `.filter()`

Si tu veux n’afficher **que certains éléments**, utilise `.filter()` avant `.map()`.

Exemple : afficher seulement les scientifiques en physique :

```jsx
export default function PhysicsList() {
  const scientists = [
    { id: 1, name: 'Albert Einstein', field: 'Physique' },
    { id: 2, name: 'Marie Curie', field: 'Chimie' },
    { id: 3, name: 'Isaac Newton', field: 'Physique' },
  ];

  return (
    <ul>
      {scientists
        .filter(scientist => scientist.field === 'Physique')
        .map(scientist => (
          <li key={scientist.id}>{scientist.name}</li>
        ))}
    </ul>
  );
}
```

👉 Résultat : seule la liste des physiciens est affichée.

***

## ⚡ À retenir

✅ **`map()`** → transformer un tableau de données en un tableau de JSX (composants).\
✅ **`filter()`** → choisir quels éléments afficher avant de les mapper.\
✅ **clé `key`** → toujours donner une clé unique quand tu affiches une liste. (Le plus souvent un `id`).

## Afficher des données à partir de tableaux  <a href="#rendering-data-from-arrays" id="rendering-data-from-arrays"></a>

## 🔹 Étape 1 : Mettre les données dans un tableau

Tu pars de données écrites en dur dans le HTML :

```html
<ul>
  <li>Creola Katherine Johnson : mathématicienne</li>
  <li>Mario José Molina-Pasquel Henríquez : chimiste</li>
  <li>Mohammad Abdus Salam : physicien</li>
  <li>Percy Lavon Julian : chimiste</li>
  <li>Subrahmanyan Chandrasekhar : astrophysicien</li>
</ul>
```

On les met dans un **tableau JavaScript** :

```js
const people = [
  'Creola Katherine Johnson : mathématicienne',
  'Mario José Molina-Pasquel Henríquez : chimiste',
  'Mohammad Abdus Salam : physicien',
  'Percy Lavon Julian : chimiste',
  'Subrahmanyan Chandrasekhar : astrophysicien',
];
```

***

## 🔹 Étape 2 : Transformer le tableau avec `.map()`

`.map()` permet de **parcourir chaque élément du tableau** et de **le transformer**.\
Ici, chaque `person` devient un `<li>` :

```jsx
const listItems = people.map(person => <li>{person}</li>);
```

***

## 🔹 Étape 3 : Retourner le JSX

On insère `listItems` dans le JSX de ton composant :

```jsx
export default function List() {
  const listItems = people.map(person => <li>{person}</li>);
  return <ul>{listItems}</ul>;
}
```

👉 Résultat : la liste s’affiche correctement.

***

## ⚠️ L’avertissement des clés (`key`)

La console affiche :

```
Warning: Each child in a list should have a unique "key" prop.
```

👉 Pourquoi ?\
Parce que React **a besoin d’une clé unique pour chaque élément d’une liste**.\
C’est comme une étiquette qui lui permet de savoir _quel élément correspond à quoi_ si la liste change (ajout, suppression, réorganisation).

Sans clé : React peut se mélanger et réutiliser le mauvais élément.\
Avec clé : React met à jour la bonne partie de la liste.

***

## 🔹 Corriger avec une clé (`key`)

Tu ajoutes une prop `key` unique à chaque `<li>` :

```jsx
export default function List() {
  const listItems = people.map(person =>
    <li key={person}>{person}</li>
  );
  return <ul>{listItems}</ul>;
}
```

✅ Ici j’utilise directement le texte `person` comme clé (car chaque entrée est unique).\
Mais **en pratique**, il vaut mieux utiliser un `id` quand tu en as un (plus robuste).

***

## ✨ À retenir

1. Mets tes données dans un **tableau ou un objet**.
2. Utilise `.map()` pour **transformer** chaque élément en JSX.
3. Donne une **clé unique (`key`)** à chaque élément de liste.

## Filtrer des tableaux d’éléments <a href="#filtering-arrays-of-items" id="filtering-arrays-of-items"></a>

## 🔹 1. Structurer les données <a href="#filtering-arrays-of-items" id="filtering-arrays-of-items"></a>

Au lieu d’avoir de simples chaînes de texte, on crée des **objets** avec plusieurs propriétés (`id`, `name`, `profession`, etc.) :

```js
const people = [
  { id: 0, name: 'Creola Katherine Johnson', profession: 'mathématicienne' },
  { id: 1, name: 'Mario José Molina-Pasquel Henríquez', profession: 'chimiste' },
  { id: 2, name: 'Mohammad Abdus Salam', profession: 'physicien' },
  { id: 3, name: 'Percy Lavon Julian', profession: 'chimiste' },
  { id: 4, name: 'Subrahmanyan Chandrasekhar', profession: 'astrophysicien' },
];
```

👉 C’est beaucoup plus flexible, car tu peux trier, filtrer ou afficher certaines infos.

***

### 🔹 2. Filtrer les données avec `.filter()`

Si tu veux n’afficher que les chimistes :

```js
const chemists = people.filter(person => person.profession === 'chimiste');
```

👉 `.filter()` garde uniquement les objets qui **satisfont la condition** (ici : profession = "chimiste").

***

### 🔹 3. Transformer en JSX avec `.map()`

Tu prends maintenant `chemists` et tu les transformes en `<li>` :

```jsx
const listItems = chemists.map(person =>
  <li key={person.id}>
    <img
      src={getImageUrl(person)}
      alt={person.name}
    />
    <p>
      <b>{person.name}:</b>
      {' ' + person.profession + ' '}
      célèbre pour {person.accomplishment}
    </p>
  </li>
);
```

⚠️ Ici j’ai ajouté **`key={person.id}`** pour supprimer l’avertissement dans la console.\
C’est super important : chaque élément d’une liste doit avoir une clé unique (id, slug, etc.) pour que React puisse suivre efficacement les changements.

***

### 🔹 4. Renvoyer le JSX

Enfin, tu les mets dans ton composant :

```jsx
export default function List() {
  const chemists = people.filter(person => person.profession === 'chimiste');
  const listItems = chemists.map(person =>
    <li key={person.id}>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}:</b>
        {' ' + person.profession + ' '}
        célèbre pour {person.accomplishment}
      </p>
    </li>
  );
  return <ul>{listItems}</ul>;
}
```

***

### ⚡ Piège : `return` implicite vs explicite

* Si tu écris une **fonction fléchée sans accolades**, le `return` est **implicite** :

```js
chemists.map(person => 
  <li key={person.id}>{person.name}</li>
);
```

* Si tu ouvres des accolades `{}`, tu dois écrire `return` :

```js
chemists.map(person => {
  return <li key={person.id}>{person.name}</li>;
});
```

👉 Oublier `return` dans la deuxième version est une **erreur fréquente** : rien ne s’affiche !

***

✅ **Résumé** :

1. **`filter()`** → sélectionne les bons éléments.
2. **`map()`** → transforme en JSX.
3. **`key` unique** → obligatoire pour éviter les warnings et améliorer les perfs

### Maintenir l’ordre des éléments d'une liste avec `key`  <a href="#keeping-list-items-in-order-with-key" id="keeping-list-items-in-order-with-key"></a>

### 🚨 Pourquoi React réclame une `key`

Quand tu écris :

```jsx
const listItems = people.map(person =>
  <li>{person.name}</li>
);
```

➡️ React affiche bien la liste, mais il te dit :

> **Warning: Each child in a list should have a unique "key" prop.**

Car sans `key`, il **ne sait pas reconnaître chaque élément de manière fiable**.

👉 Si la liste change (tri, ajout, suppression), React ne peut pas savoir **quel élément correspond à quel ancien élément** et risque de :

* recréer inutilement des nœuds DOM,
* casser l’**état interne** des composants enfants (ex : un `<input>` qui perd son curseur ou sa valeur).

***

### ✅ Comment corriger avec `key`

Ajoute une propriété unique (ex : `id`) sur chaque élément :

```jsx
const listItems = people.map(person =>
  <li key={person.id}>
    {person.name}
  </li>
);
```

Ici :

* `person.id` est **stable** (ne change pas entre les rendus),
* chaque `<li>` est identifié de façon unique.

***

### 🔹 Exemple complet

#### `App.js`

```jsx
import { people } from './data.js';

export default function List() {
  const listItems = people.map(person =>
    <li key={person.id}>
      <img
        src={"https://i.imgur.com/" + person.imageId + "s.jpg"}
        alt={person.name}
      />
      <p>
        <b>{person.name}</b>: {person.profession}  
        <br />
        Célèbre pour {person.accomplishment}
      </p>
    </li>
  );

  return <ul>{listItems}</ul>;
}
```

#### `data.js`

```jsx
export const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'mathématicienne',
  accomplishment: 'ses calculs pour vol spatiaux',
  imageId: 'MK3eW3A'
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'chimiste',
  accomplishment: 'sa découverte du trou dans la couche d’ozone',
  imageId: 'mynHUSa'
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'physicien',
  accomplishment: 'sa théorie de l’électromagnétisme',
  imageId: 'bE7W1ji'
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'chimiste',
  accomplishment: 'ses travaux sur la cortisone et les stéroïdes',
  imageId: 'IOjWm71'
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrophysicien',
  accomplishment: 'son calcul de la masse des naines blanches',
  imageId: 'lrWQx8l'
}];
```

***

### 🔹 Cas particulier : plusieurs nœuds par élément

Tu veux afficher **plusieurs balises** (`<h1>`, `<p>`, …) pour chaque `person`.

👉 Avec `<>...</>` (Fragment court), tu ne peux **pas** mettre de `key`.

Donc tu utilises `Fragment` **avec clé** :

```jsx
import { Fragment } from 'react';
import { people } from './data.js';

export default function List() {
  const listItems = people.map(person =>
    <Fragment key={person.id}>
      <h1>{person.name}</h1>
      <p>{person.profession}</p>
    </Fragment>
  );

  return <div>{listItems}</div>;
}
```

⚡ Ici, `Fragment` agit comme un conteneur **invisible** (aucune `<div>` en plus dans le DOM), mais il permet d’associer une `key`.

***

### 📝 À retenir

1. Chaque élément généré dans une liste **a besoin d’une `key` unique**.
2. Utilise un **identifiant stable** (comme un `id` venant des données).
3. ⚠️ Évite d’utiliser l’**index du tableau** (`i`) sauf si :
   * la liste est **statique**,
   * ou les éléments n’ont pas d’ID naturel.\
     Sinon → bugs possibles si tu modifies l’ordre des éléments.
4. Si tu retournes plusieurs nœuds → utilise `<Fragment key={...}>`.

## 🔑 Où récupérer la `key`

👉 Tout dépend d’où viennent tes données :

1.  **📦 Base de données**

    * Utilise directement l’**ID** fourni (ex: `id`, `_id`, `uuid`, …).

    ```jsx
    <li key={person.id}>{person.name}</li>
    ```
2.  **📝 Données locales générées par l’appli** (ex: notes, todo list, etc.)

    * Génère un ID **stable** dès la création :
      * compteur auto-incrémenté,
      * `crypto.randomUUID()`,
      * bibliothèque comme `uuid`.

    ```jsx
    const newNote = { id: crypto.randomUUID(), text: "Nouvelle note" };
    ```

⚠️ Important : **ne génère pas la clé à chaque rendu**, mais bien **au moment de la création de la donnée**.

***

## 📏 Les règles des `key`

1. **Unique dans la liste**
   * Pas besoin qu’elles soient uniques dans tout le projet, juste dans une même liste.
2. **Stables dans le temps**
   * Ne change pas une `key` entre deux rendus, sinon React recrée tout l’élément au lieu de le réutiliser.
3. **Pas d’index du tableau (sauf cas très statiques)**
   *   Exemple **mauvais** :

       ```jsx
       people.map((person, index) =>
         <li key={index}>{person.name}</li>
       )
       ```
   * Ça casse dès que tu supprimes/insères un élément (les index changent → React se trompe d’élément).
4. **Pas de `Math.random()`**
   * Générer une `key` aléatoire → nouvelle valeur à chaque rendu → React détruit et recrée chaque élément → perte des saisies utilisateur, ralentissements, bugs.

***

## 🤔 Pourquoi React a besoin d’une `key`

👉 Imagine ton bureau avec des fichiers **sans nom**.

* Tu dis "ouvre le 2ᵉ fichier".
* Si tu supprimes le 1ᵉʳ fichier → le 2ᵉ devient le 1ᵉʳ → tu perds ta référence.

💡 Les `key` jouent le rôle **des noms de fichiers** : elles permettent à React d’identifier chaque élément indépendamment de sa position dans le tableau.

***

## ⚠️ Attention : la `key` n’est pas une prop

React s’en sert **uniquement en interne** pour faire le suivi des éléments.\
👉 Ton composant **n’a pas accès** à `key`.

Exemple :

```jsx
function Profile({ userId }) {
  return <p>Profil #{userId}</p>;
}

export default function App() {
  return <Profile key="123" userId="123" />;
}
```

* Ici, `key="123"` → utilisé **seulement par React**.
* Si tu veux accéder à l’ID dans ton composant → tu dois le passer explicitement (`userId`).

***

## ✅ Bonnes pratiques en résumé

* Toujours utiliser un **identifiant stable** (id, uuid, compteur).
* **Jamais** `index` (sauf liste fixe jamais modifiée).
* **Jamais** `Math.random()`.
* Générer l’ID **à la création** des données, pas au rendu.
* Si tu as besoin de l’ID dans ton composant → passe-le en **prop séparée**.

\
