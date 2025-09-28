# Vous n’avez peut-être pas besoin d’un Effet (useEffect)-(You Might Not Need an Effect)

Les **Effets (`useEffect`)** sont une **porte de sortie** (_escape hatch_) du paradigme React.\
Ils permettent de **sortir de React** pour synchroniser vos composants avec un système **externe** (widget non-React, API réseau, navigateur DOM, etc.).

👉 **Mais si aucun système externe n’est impliqué** (par exemple : mettre à jour un `state` en fonction d’un `prop` ou d’un autre `state`), vous **ne devriez pas utiliser `useEffect`**.

⚡ Enlever les Effets inutiles rend votre code :

* plus **simple à lire**,
* plus **rapide à exécuter**,
* moins **sujet aux bugs**.

***

### Dans ce chapitre, vous allez apprendre :

✅ Pourquoi et comment **supprimer les Effets inutiles** de vos composants.\
✅ Comment **mémoriser** des calculs coûteux sans `useEffect`.\
✅ Comment **réinitialiser** ou **ajuster** un état local sans `useEffect`.\
✅ Comment **partager de la logique** entre gestionnaires d’événements.\
✅ Quelle logique doit être déplacée dans des **event handlers** plutôt que dans des Effets.\
✅ Comment **notifier les composants parents** d’un changement.

***

👉 En résumé : **réservez `useEffect` uniquement aux cas où vous devez interagir avec l’extérieur de React** (réseau, DOM direct, librairies tierces…).\
Pour le reste, React offre déjà des solutions **plus simples et plus sûres**.

## Comment supprimer les Effets inutiles (`useEffect`)

Il existe **deux cas très fréquents** où vous **n’avez pas besoin d’Effets** dans vos composants React.

***

### 1. 🚫 Pas besoin d’Effet pour transformer des données avant affichage

Mauvaise intuition :\
On pense parfois qu’il faut un `useEffect` + un `useState` pour recalculer une liste filtrée ou triée.

Exemple incorrect :

```jsx
// ❌ Mauvais : inutile d'utiliser useEffect
import { useState, useEffect } from "react";

function ProductList({ products, filter }) {
  const [visibleProducts, setVisibleProducts] = useState([]);

  useEffect(() => {
    setVisibleProducts(products.filter(p => p.category === filter));
  }, [products, filter]);

  return (
    <ul>
      {visibleProducts.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

👉 Ici, `useEffect` est **inutile** :

* `React` relance déjà votre fonction composant quand `props` ou `state` changent.
* Vous doublez inutilement le travail : calcul → render → commit → effet → nouveau state → render encore une fois.

✅ Version correcte : transformez directement vos données dans le rendu.

```jsx
// ✅ Bon : calculez directement dans le rendu
function ProductList({ products, filter }) {
  const visibleProducts = products.filter(p => p.category === filter);

  return (
    <ul>
      {visibleProducts.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

***

### 2. 🚫 Pas besoin d’Effet pour gérer des événements utilisateurs

Mauvaise intuition :\
On veut parfois envoyer une requête API ou déclencher une action dans `useEffect` après un clic.

Exemple incorrect :

```jsx
// ❌ Mauvais : l'effet n'est pas déclenché par l’événement
import { useEffect, useState } from "react";

function BuyButton() {
  const [bought, setBought] = useState(false);

  useEffect(() => {
    if (bought) {
      fetch("/api/buy", { method: "POST" });
      alert("Achat confirmé !");
    }
  }, [bought]);

  return <button onClick={() => setBought(true)}>Acheter</button>;
}
```

👉 Ici, `useEffect` ajoute de la **complexité inutile** :

* L’événement utilisateur est déjà connu dans le handler.
* Quand l’Effet se déclenche, vous ne savez plus _quel bouton_ ou _quelle action précise_ a causé le changement.

✅ Version correcte : gérez tout dans le gestionnaire d’événement.

```jsx
// ✅ Bon : traitez tout directement dans le handler
function BuyButton() {
  function handleClick() {
    fetch("/api/buy", { method: "POST" });
    alert("Achat confirmé !");
  }

  return <button onClick={handleClick}>Acheter</button>;
}
```

***

### 3. ✅ Quand un Effet est vraiment nécessaire

Par contre, utilisez `useEffect` lorsqu’il faut :

* **Synchroniser avec un système externe** (ex. widget jQuery, librairie non-React).
* **Appeler un service externe** lié à l’état React (ex. mettre à jour une carte interactive avec le niveau de zoom stocké dans un `state`).
* **Charger ou synchroniser des données distantes** (ex. recherche dynamique avec une API).

⚡ Mais souvenez-vous :

* Les frameworks modernes (Next.js, Remix, etc.) offrent des **mécanismes de data fetching plus performants** que de mettre les requêtes directement dans `useEffect`.

***

👉 Résumé :

* **Transformation de données** → directement dans le rendu.
* **Actions utilisateur** → directement dans les gestionnaires d’événements.
* **Interactions avec l’extérieur (DOM, API, widgets)** → là oui, `useEffect`.

## Mettre à jour un état basé sur des props ou un autre état

Prenons un exemple concret : vous avez deux variables d’état `firstName` et `lastName`. Vous voulez calculer un `fullName` qui les concatène. Et vous souhaitez que `fullName` se mette automatiquement à jour quand `firstName` ou `lastName` changent.

***

### 🚫 Mauvaise approche : redondance d’état et `useEffect` inutile

Votre premier réflexe pourrait être de créer un troisième état `fullName` et de le mettre à jour avec un **Effet** :

```jsx
import { useState, useEffect } from "react";

function Form() {
  const [firstName, setFirstName] = useState("Taylor");
  const [lastName, setLastName] = useState("Swift");

  // ❌ Mauvais : état redondant + useEffect inutile
  const [fullName, setFullName] = useState("");

  useEffect(() => {
    setFullName(firstName + " " + lastName);
  }, [firstName, lastName]);

  return <h1>{fullName}</h1>;
}
```

👉 Problèmes avec cette approche :

* **Redondance** : vous stockez une donnée (`fullName`) qui peut déjà être dérivée de deux autres états.
* **Inefficace** : React fait un premier rendu avec une valeur obsolète de `fullName`, puis relance immédiatement un nouveau rendu après la mise à jour.
* **Source de bugs** : vous risquez d’avoir des états désynchronisés.

***

### ✅ Bonne approche : calculer pendant le rendu

Il n’est pas nécessaire de créer un état supplémentaire. Calculez directement `fullName` à partir de `firstName` et `lastName` pendant le rendu :

```jsx
import { useState } from "react";

function Form() {
  const [firstName, setFirstName] = useState("Taylor");
  const [lastName, setLastName] = useState("Swift");

  // ✅ Bon : calcul direct dans le rendu
  const fullName = firstName + " " + lastName;

  return <h1>{fullName}</h1>;
}
```

***

### 🎯 Règle à retenir

👉 **Si une donnée peut être dérivée de `props` ou `state`, ne la stockez pas dans un `state`.**

* Calculez-la directement dans le rendu.
* Cela rend votre code **plus simple**, **plus rapide** et **moins sujet aux erreurs**.

## Mise en cache des calculs coûteux avec `useMemo`

Prenons un exemple avec une liste de tâches (`todos`). On veut afficher uniquement celles qui correspondent à un filtre (`filter`).

***

### 🚫 Mauvaise approche : état redondant et `useEffect` inutile

On pourrait être tenté de calculer les tâches visibles et de les stocker dans un état, mis à jour via un **Effet** :

```jsx
import { useState, useEffect } from "react";

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState("");

  // ❌ Mauvais : état redondant + useEffect inutile
  const [visibleTodos, setVisibleTodos] = useState([]);

  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);

  return (
    <ul>
      {visibleTodos.map((todo) => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

👉 Problèmes :

* Vous stockez une donnée dérivable (`visibleTodos`).
* Vous déclenchez un rendu supplémentaire inutile.

***

### ✅ Bonne approche : calcul direct dans le rendu

Supprimez l’état et l’Effet, et calculez directement :

```jsx
import { useState } from "react";

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState("");

  // ✅ Calcul direct
  const visibleTodos = getFilteredTodos(todos, filter);

  return (
    <ul>
      {visibleTodos.map((todo) => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

👉 Cette approche est **simple et efficace** tant que `getFilteredTodos()` n’est pas trop coûteux.

***

### ⚡ Optimisation : mémorisation avec `useMemo`

Si `getFilteredTodos()` est une fonction lourde (ex. beaucoup de boucles ou milliers d’éléments), vous pouvez **mémoriser** son résultat avec `useMemo`.

```jsx
import { useState, useMemo } from "react";

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState("");

  // ✅ Calcul uniquement si `todos` ou `filter` changent
  const visibleTodos = useMemo(() => {
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);

  return (
    <ul>
      {visibleTodos.map((todo) => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

👉 Ici, `useMemo` demande à React de **réutiliser** le dernier résultat si `todos` et `filter` n’ont pas changé.

***

### 🧐 Comment savoir si un calcul est coûteux ?

Ajoutez une mesure de temps avec `console.time`:

```jsx
console.time("filter array");
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd("filter array");
```

* Si cela prend **plusieurs ms** de manière répétée, envisagez `useMemo`.
* Sinon, laissez simplement le calcul direct (plus simple, plus lisible).

⚠️ Astuce : testez avec **CPU Throttling** (dans Chrome DevTools) pour simuler une machine plus lente, car votre PC est sûrement plus rapide que celui de vos utilisateurs.

***

### 🎯 Règle à retenir

👉 **Ne stockez pas de données dérivées dans le state.**\
👉 **Utilisez `useMemo` uniquement si un calcul est réellement coûteux.**\
👉 Sinon, un calcul direct dans le rendu est la meilleure solution (plus simple et plus clair).

## 🔄 Réinitialiser tout l’état lorsqu’une prop change

Prenons l’exemple d’une page de profil qui reçoit un `userId` en prop.\
Cette page contient un champ commentaire avec un état `comment`.

***

### 🚫 Mauvaise approche : utiliser un `useEffect` pour réinitialiser l’état

On pourrait être tenté de remettre `comment` à vide quand `userId` change :

```jsx
import { useState, useEffect } from "react";

export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState("");

  // ❌ Mauvais : réinitialisation via un Effet
  useEffect(() => {
    setComment("");
  }, [userId]);

  return (
    <>
      <h1>Profil {userId}</h1>
      <textarea
        value={comment}
        onChange={(e) => setComment(e.target.value)}
      />
    </>
  );
}
```

👉 Problèmes :

* Inefficace : le composant **rend d’abord avec l’ancienne valeur**, puis re-render avec l’état réinitialisé.
* Complexe : si l’UI est imbriquée, il faudrait répéter cette logique dans chaque sous-composant.

***

### ✅ Bonne approche : utiliser la prop comme **clé**

On demande à React de considérer chaque profil comme un **composant différent**, grâce à la prop `key`.

```jsx
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId} // 🟢 Indique à React que chaque userId est un composant distinct
    />
  );
}

function Profile({ userId }) {
  const [comment, setComment] = useState("");

  return (
    <>
      <h1>Profil {userId}</h1>
      <textarea
        value={comment}
        onChange={(e) => setComment(e.target.value)}
      />
    </>
  );
}
```

👉 Avantages :

* L’état (`comment` et tous les autres) est **automatiquement réinitialisé** quand `userId` change.
* Plus simple et plus robuste (React gère tout seul la réinitialisation).
* Fonctionne aussi pour tous les sous-composants imbriqués.

***

### ⚡ Pourquoi ça marche ?

Normalement, React conserve l’état quand **le même composant est rendu au même endroit**.\
Mais avec une **clé différente** (`key={userId}`) :

* React considère qu’il s’agit d’un **nouveau composant**.
* Il démonte l’ancien, monte le nouveau, et donc réinitialise son état.

***

✅ Résultat :\
Quand vous changez d’utilisateur, **toute l’UI du profil redémarre proprement**, sans effet secondaire ni code supplémentaire.

## 🔧 Ajuster une partie de l’état quand une prop change

Il arrive qu’on ne veuille pas réinitialiser **tout l’état**, mais seulement une partie, quand une prop change.

***

### 🚫 Mauvaise approche : ajuster l’état avec un `useEffect`

Prenons un composant `List` qui reçoit un tableau `items` et garde en mémoire un élément sélectionné :

```jsx
import { useState, useEffect } from "react";

function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // ❌ Mauvais : réinitialiser via un Effet
  useEffect(() => {
    setSelection(null);
  }, [items]);

  // ...
}
```

👉 Problèmes :

* Le composant et ses enfants **rendent d’abord avec une sélection périmée**, puis React met à jour le DOM, et enfin `setSelection(null)` déclenche un second rendu.
* Cela entraîne des **re-renders inutiles** et rend le flux de données plus difficile à suivre.

***

### ⚡ Meilleure approche : ajuster l’état **pendant le rendu**

On peut comparer les props actuelles avec les précédentes directement dans le rendu :

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  const [prevItems, setPrevItems] = useState(items);

  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null); // 🟢 Réinitialisation immédiate
  }

  // ...
}
```

👉 Ce qui se passe :

* React détecte que `items` a changé.
* Il réinitialise immédiatement `selection` **avant** de mettre à jour le DOM.
* Ainsi, les enfants ne voient jamais une valeur périmée.

⚠️ Attention : React n’autorise ce type de mise à jour **que dans le même composant**. Sinon, vous auriez une erreur (pour éviter des boucles infinies).

***

### ✅ Encore mieux : éviter complètement les ajustements

Plutôt que de stocker tout l’objet sélectionné et devoir le réinitialiser, stockons uniquement l’**ID** de l’élément sélectionné. Puis calculons `selection` pendant le rendu :

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);

  // ✅ Sélection calculée dynamiquement
  const selection = items.find((item) => item.id === selectedId) ?? null;

  // ...
}
```

👉 Avantages :

* Plus besoin de “réinitialiser” manuellement l’état.
* Si l’élément sélectionné n’existe plus dans `items`, `selection` devient `null` automatiquement.
* Plus simple, plus prévisible et moins de risques de bugs.

***

### 📌 À retenir

1. ❌ Évitez de réinitialiser ou ajuster l’état avec des `useEffect`.
2. ⚡ Si vraiment nécessaire, comparez les props et ajustez directement dans le rendu.
3. ✅ Mais dans la majorité des cas, le mieux est de **calculer dynamiquement** ce dont vous avez besoin (comme avec l’ID).

## 🔄 Partager de la logique entre gestionnaires d’événements

Imaginons une page produit avec deux boutons : **Acheter** et **Passer à la caisse**. Les deux permettent d’ajouter le produit au panier, et vous voulez afficher une notification à chaque ajout.

***

### 🚫 Mauvaise approche : mettre la logique dans un `useEffect`

On pourrait être tenté de surveiller `product.isInCart` et d’afficher une notification dès qu’il passe à `true` :

```jsx
function ProductPage({ product, addToCart }) {
  // ❌ Mauvais : logique spécifique aux événements mise dans un Effet
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`Ajouté ${product.name} au panier !`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
}
```

👉 Problème :

* Si l’application sauvegarde le panier dans le localStorage ou le backend, **au rechargement de la page**, `product.isInCart` est déjà `true`.
* Résultat : une notification apparaît à chaque refresh, même sans action de l’utilisateur.
* L’utilisateur est trompé car l’effet est déclenché par le rendu, pas par son interaction.

***

### ✅ Bonne approche : mettre la logique **dans les gestionnaires d’événements**

L’affichage de la notification doit découler directement d’une action utilisateur (cliquer un bouton), pas d’un rendu.

On crée une fonction partagée :

```jsx
function ProductPage({ product, addToCart }) {
  // ✅ Correct : logique spécifique appelée depuis les gestionnaires d’événements
  function buyProduct() {
    addToCart(product);
    showNotification(`Ajouté ${product.name} au panier !`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
}
```

👉 Avantages :

* La logique est **factorisée** dans `buyProduct()`.
* Pas de duplication entre les boutons.
* La notification est affichée **uniquement quand l’utilisateur agit**, et non au simple rendu de la page.

***

### 📌 À retenir

* **Effets (`useEffect`)** → utilisés pour synchroniser avec un système externe **à cause du rendu**.
* **Gestionnaires d’événements** → utilisés pour tout ce qui doit se produire **à cause d’une action utilisateur**.
* Si vous vous demandez : _« Pourquoi ce code doit-il s’exécuter ? »_
  * Si c’est _parce que le composant est affiché_ → `useEffect`.
  * Si c’est _parce qu’un utilisateur a cliqué, tapé ou interagi_ → gestionnaire d’événements.

## 📩 Envoyer une requête POST

Prenons un exemple avec un formulaire qui doit envoyer deux types de requêtes **POST** :

1. Un **événement analytique** quand le formulaire est affiché.
2. Une **inscription** quand l’utilisateur soumet le formulaire.

***

### 🚫 Mauvaise approche : mélanger les deux dans des Effets

```jsx
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Correct : cet envoi se fait parce que le composant est affiché
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  // ❌ Mauvais : logique spécifique à un événement mise dans un Effet
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
}
```

👉 Problèmes :

* L’inscription (`/api/register`) dépend d’une **interaction utilisateur** (clic sur Submit), pas de l’affichage du composant.
* En passant par un **state + Effect**, on crée une logique inutilement complexe et on introduit un risque de bugs (exécutions imprévues).

***

### ✅ Bonne approche : séparer selon la cause

```jsx
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Correct : logique liée à l’affichage du composant
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    // ✅ Correct : logique déclenchée par une action utilisateur
    post('/api/register', { firstName, lastName });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={firstName} onChange={e => setFirstName(e.target.value)} />
      <input value={lastName} onChange={e => setLastName(e.target.value)} />
      <button type="submit">Register</button>
    </form>
  );
}
```

👉 Avantages :

* L’analytics reste dans un **Effet**, car il est déclenché **par l’affichage** du formulaire.
* L’inscription reste dans un **gestionnaire d’événement**, car elle est déclenchée **par l’action de l’utilisateur**.
* Code plus simple, plus lisible et moins sujet aux erreurs.

***

### 📌 À retenir

* **Effets (`useEffect`)** : utilisés quand une action doit se produire **parce que le composant est affiché** (ex. log analytique, abonnement, synchro externe).
* **Gestionnaires d’événements** : utilisés quand une action doit se produire **parce qu’un utilisateur interagit** (clic, saisie, soumission).

## 🔗 Chaînes de calculs dans les composants React

Il peut être tentant d’écrire une série d’**Effets** qui s’enchaînent et mettent à jour l’état les uns après les autres :

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  // ❌ Mauvais : chaque état dépend d’un autre via une chaîne d’Effects
  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  useEffect(() => {
    alert('Good game!');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    } else {
      setCard(nextCard);
    }
  }
}
```

***

### ⚠️ Problèmes de cette approche

1. **Inefficacité**
   * Chaque `setState` déclenche un nouveau rendu, et donc la chaîne complète peut produire plusieurs re-renders inutiles.
   * Exemple : `setCard → render → setGoldCardCount → render → setRound → render → setIsGameOver → render`.
2. **Rigidité et fragilité**
   * Si tu veux implémenter une nouvelle fonctionnalité (comme revenir à un état précédent du jeu), la chaîne d’Effects peut **modifier des états par erreur**.
   * Le système devient difficile à maintenir, car chaque état dépend implicitement des autres via des Effets.

***

### ✅ Bonne approche : calculer pendant le rendu et regrouper la logique dans les handlers

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ Calcul direct pendant le rendu
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // ✅ Calcul de la logique métier directement dans l’événement
    setCard(nextCard);

    if (nextCard.gold) {
      if (goldCardCount <= 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        const nextRound = round + 1;
        setRound(nextRound);

        if (nextRound === 5) {
          alert('Good game!');
        }
      }
    }
  }

  // ...
}
```

***

### 🎯 Avantages

* **Efficace** : moins de re-renders car plusieurs états sont mis à jour dans le même handler.
* **Lisible** : la logique métier est centralisée dans une seule fonction plutôt que dispersée dans plusieurs Effets.
* **Flexible** : tu peux implémenter l’historique du jeu ou restaurer un état précédent sans déclencher accidentellement toute une chaîne d’updates.

***

### 📌 Quand une chaîne d’Effects est appropriée ?

👉 Seulement quand tu synchronises avec un **système externe**.\
Exemple : un formulaire avec plusieurs sélecteurs dont les options dépendent d’une requête réseau.\
Dans ce cas, chaque `useEffect` sert à **écouter un changement** et à lancer une **synchro externe** (fetch, abonnement, etc.).

## 🚀 Initialiser l’application

Certaines logiques doivent s’exécuter **une seule fois au chargement de l’application** (par exemple : vérifier un token d’authentification, charger des données locales, configurer un service global).

***

### ❌ Mauvaise approche : utiliser un `useEffect` vide

On peut être tenté d’écrire ceci :

```jsx
function App() {
  // ❌ Mauvais : cet effet sera exécuté deux fois en développement
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
  // ...
}
```

👉 Problème : en **mode développement**, React monte puis démonte/remonte le composant une fois (Strict Mode). Résultat : `useEffect` est appelé **deux fois**, ce qui peut causer des effets indésirables (ex. invalider un token).

***

### ✅ Solution 1 : variable de contrôle au niveau du module

On peut utiliser une variable définie **hors du composant** pour vérifier si l’initialisation a déjà eu lieu :

```jsx
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ Ne s’exécute qu’une seule fois par chargement d’app
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);

  return <MainRoutes />;
}
```

***

### ✅ Solution 2 : initialiser directement au niveau du module

Une autre approche consiste à exécuter la logique **au moment du chargement du fichier** (avant même que le composant ne soit monté) :

```jsx
if (typeof window !== 'undefined') { // Vérifie qu’on est bien côté client
  // ✅ S’exécute une seule fois par chargement d’app
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  return <MainRoutes />;
}
```

👉 Avantage :

* Pas affecté par les doubles montages du `StrictMode` en dev.

👉 Inconvénient :

* S’exécute dès l’import du module, même si le composant n’est pas encore rendu.
* À réserver uniquement aux composants racines (`App.js`, `index.js`) ou au point d’entrée de ton application.

***

### 📌 Règle d’or

* Si la logique doit s’exécuter **à chaque fois qu’un composant est monté** ➝ `useEffect`.
* Si la logique doit s’exécuter **une seule fois par chargement d’application** ➝ top-level (variable de garde ou code de module).

## 📊 Comparatif : Initialisation dans `useEffect` vs. Top-level module

| **Cas d’usage**                   | **Avec `useEffect`**                                                                   | **Au niveau du module (top-level)**                                            |
| --------------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| **Quand s’exécute ?**             | À chaque montage du composant                                                          | Une seule fois au chargement du module                                         |
| **Impact du `StrictMode` en dev** | ⚠️ Exécuté 2 fois (montage → démontage → remontage)                                    | ✅ Exécuté 1 fois seulement                                                     |
| **Exemple typique**               | Se connecter à un WebSocket quand le composant est affiché                             | Vérifier un token, charger des préférences locales, configurer une lib globale |
| **Avantage**                      | Respecte le cycle de vie React, clair et standard                                      | Pas de doublon en dev, garanti une seule exécution par app                     |
| **Inconvénient**                  | Peut provoquer des effets indésirables (appel doublé en dev)                           | S’exécute même si le composant n’est jamais rendu                              |
| **À utiliser pour…**              | Effets liés à l’apparition/disparition du composant (connexion, abonnement, animation) | Initialisation globale (auth, config, cache, analytics setup)                  |

***

👉 **Règle pratique** :

* 🔄 **Lié au cycle de vie du composant** → `useEffect`.
* 🌍 **Global, unique à l’application** → code au top-level du module (ou variable de garde).

## 📡 Faire remonter des données au parent

Dans React, le **flux de données est descendant** (du parent vers l’enfant). Il est donc important de **ne pas casser ce flux** en essayant de faire remonter des données via des `useEffect` dans les composants enfants.

***

### ❌ Mauvaise pratique : envoyer les données au parent avec un `useEffect`

Ici, l’enfant récupère des données et les renvoie au parent :

```jsx
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();

  // ❌ Mauvais : faire remonter les données dans un effet
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);

  // ...
}
```

👉 Problème :

* Le flux de données devient **confus** (on ne sait plus si l’état vient du parent ou de l’enfant).
* Debug compliqué : difficile de remonter la source d’un bug car l’enfant “contrôle” le parent.

***

### ✅ Bonne pratique : laisser le parent gérer les données et les passer en props

Le parent doit être responsable du **fetching** et de l’état, puis transmettre le résultat à l’enfant :

```jsx
function Parent() {
  const data = useSomeAPI();
  // ✅ Le parent récupère et gère les données
  return <Child data={data} />;
}

function Child({ data }) {
  // ✅ L’enfant ne fait que consommer les données
  return <div>{data ? data.title : "Chargement..."}</div>;
}
```

***

### 📌 Règle d’or

* 🔄 **Si parent et enfant ont besoin des mêmes données** ➝ le **parent** doit les récupérer.
* 👶 L’enfant ne doit **jamais mettre à jour directement l’état du parent via un `useEffect`**.
* ✅ Le flux reste **descendant et prévisible** : props → enfants.

## 🔌 S’abonner à une source de données externe

Parfois, un composant doit écouter des données qui ne font **pas partie de l’état React** :

* API du navigateur (`navigator.onLine`, `localStorage`, etc.)
* Librairies tierces (ex. Redux, Zustand, etc.)
* Tout store externe qui peut changer sans que React le sache

Dans ce cas, il faut **abonner** le composant à cette source de données pour qu’il se mette à jour automatiquement.

***

### ❌ Mauvaise approche : utiliser `useEffect` avec un état

```jsx
function useOnlineStatus() {
  // ❌ État React + Effet pour synchroniser
  const [isOnline, setIsOnline] = useState(true);

  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener('online', updateState);
    window.addEventListener('offline', updateState);

    return () => {
      window.removeEventListener('online', updateState);
      window.removeEventListener('offline', updateState);
    };
  }, []);

  return isOnline;
}
```

👉 Inconvénients :

* Risque de bugs si on oublie le nettoyage (`removeEventListener`)
* Plus verbeux
* On “duplique” la donnée externe dans l’état React au lieu d’y accéder directement

***

### ✅ Bonne approche : `useSyncExternalStore`

React fournit un hook dédié à ce cas précis :

```jsx
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);

  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe,             // Abonnement aux changements
    () => navigator.onLine, // Valeur côté client
    () => true              // Valeur par défaut côté serveur (SSR)
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  return <p>{isOnline ? "✅ En ligne" : "❌ Hors ligne"}</p>;
}
```

***

### 🎯 Pourquoi c’est mieux ?

* **Moins d’erreurs** : React gère le cycle d’abonnement/désabonnement.
* **Clair et optimisé** : React ne réabonne pas tant que la fonction `subscribe` reste la même.
* **Compatible SSR** : on peut définir une valeur de fallback pour le rendu serveur (`() => true`).
* **Réutilisable** : on peut encapsuler la logique dans un hook personnalisé (`useOnlineStatus`) et l’utiliser partout.

***

### 📌 À retenir

* Si la donnée vient de **React** (props, state, contexte) ➝ pas besoin d’Effet.
* Si la donnée vient d’une **source externe mutable** ➝ `useSyncExternalStore`.
* `useEffect` ne doit être utilisé qu’en dernier recours (DOM impératif, side effects purs).

## 🌐 Récupération de données avec `useEffect`

Beaucoup d’applications utilisent les **Effets** pour déclencher un **fetch** de données côté client. C’est une pratique courante, mais elle cache des pièges si on ne fait pas attention.

***

### ❌ Exemple naïf (sans cleanup)

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // ❌ Mauvais : pas de gestion des réponses obsolètes
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
}
```

👉 Problème : **race condition** ⚡\
Si l’utilisateur tape vite (`h` → `he` → `hel` → `hell` → `hello`), plusieurs requêtes partent en même temps. Rien ne garantit que les réponses arrivent dans le bon ordre.\
Résultat : l’écran peut afficher de vieux résultats.

***

### ✅ Exemple corrigé avec cleanup

On ajoute une variable `ignore` qui permet d’ignorer les réponses arrivées trop tard :

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    let ignore = false;

    fetchResults(query, page).then(json => {
      if (!ignore) {
        setResults(json);
      }
    });

    return () => {
      ignore = true; // 🚫 On ignore les anciennes réponses
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
}
```

👉 Ainsi, seul le **dernier fetch demandé** est pris en compte.\
Même si des requêtes en doublon sont visibles en développement (StrictMode), le state reste cohérent.

***

### 🛠 Aller plus loin

La récupération de données n’est pas si simple. Quelques problématiques récurrentes :

* **Mettre en cache** les réponses pour éviter de refetch inutilement (ex. bouton "Back").
* **Précharger** les données pour éviter les "waterfalls" réseau (fetch d’un parent ➝ puis fetch d’un enfant).
* **Supporter le SSR (Server-Side Rendering)** pour que la page initiale affiche déjà les données au lieu d’un spinner.
* **Gérer les erreurs et le loading state** (`try/catch`, état `isLoading`, `isError`).

C’est pour ça que les **frameworks modernes (Next.js, Remix, etc.)** et les **librairies spécialisées (React Query, SWR, Apollo, etc.)** proposent des solutions intégrées et optimisées.

***

### 🎯 Extraction en Hook personnalisé

Tu peux encapsuler la logique dans un Hook réutilisable :

```jsx
function useData(url) {
  const [data, setData] = useState(null);

  useEffect(() => {
    let ignore = false;

    fetch(url)
      .then(res => res.json())
      .then(json => {
        if (!ignore) {
          setData(json);
        }
      });

    return () => {
      ignore = true;
    };
  }, [url]);

  return data;
}

function SearchResults({ query }) {
  const [page, setPage] = useState(1);
  const params = new URLSearchParams({ query, page });
  const results = useData(`/api/search?${params}`);

  function handleNextPageClick() {
    setPage(page + 1);
  }
}
```

👉 Avantage : le composant est plus clair et la logique de fetch est centralisée.\
Tu peux enrichir `useData` avec :

* `loading` + `error` states
* mise en cache
* déduplication des requêtes

***

### 📌 Récap rapide

* ✅ Si tu peux **calculer** quelque chose pendant le rendu → pas besoin de `useEffect`.
* ✅ Si c’est une logique déclenchée par **l’affichage du composant** → `useEffect`.
* ✅ Si c’est déclenché par **une action utilisateur** → handler d’événement.
* ⚡ Pour le **fetch**, pense toujours à gérer les **race conditions** avec un cleanup.
* 🔮 Préfère des solutions dédiées (React Query, SWR, etc.) pour la mise en cache, le SSR et les requêtes parallèles.
