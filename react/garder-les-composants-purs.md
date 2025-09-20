# ⚖️Garder les composants purs

## ⚖️ 1. Qu’est-ce qu’une fonction pure ?

En JavaScript (et en informatique en général), une **fonction pure** est une fonction qui :

1. **Ne modifie rien en dehors d’elle-même** (pas de "side effects").\
   → Elle ne change pas une variable globale, un objet externe, ni le DOM.
2. **Renvoie toujours le même résultat pour les mêmes entrées**.\
   → Si tu lui donnes les mêmes arguments, elle retourne la même valeur.

Exemple pur ✅ :

```js
function double(x) {
  return x * 2;
}
```

Exemple impur ❌ :

```js
let total = 0;

function addToTotal(x) {
  total += x;   // ⚠️ modifie une variable externe
  return total;
}
```

***

## ⚛️ 2. Les composants React doivent être purs

👉 Un composant React est une fonction. Donc **il doit rester pur** pendant sa phase de rendu.

Exemple pur ✅ :

```jsx
function Greeting({ name }) {
  return <h1>Bonjour {name} !</h1>;
}
```

* Même `name` donné → toujours le même rendu.
* Pas de modification d’objets ou variables extérieures.

Exemple impur ❌ :

```jsx
let counter = 0;

function Counter() {
  counter++; // ⚠️ mutation externe pendant le rendu
  return <p>{counter}</p>;
}
```

* Chaque rendu modifie `counter`.
* Résultat : rendu imprévisible, bugs si React réutilise le composant.

***

## 🛑 3. Pourquoi c’est important ?

React **rappelle tes composants plusieurs fois** (ex: en Mode Strict, en transitions, en Suspense…).\
Si ton composant n’est pas pur → tu auras :

* Des **données incohérentes** (affichage différent alors que les props sont identiques).
* Des **bugs subtils** : valeurs qui changent sans raison, doublons, effets qui s’exécutent trop de fois.

La pureté garantit que le rendu est comme une **photo** :

* toujours le même rendu pour les mêmes props et état.
* pas d’effet de bord pendant le rendu.

***

## 🕵️ 4. Le Mode Strict de React

React inclut un outil intégré : **StrictMode**.\
Il aide à détecter les composants impurs.

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App.js';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

💡 En Mode Strict (en développement uniquement) :

* React **exécute les composants deux fois** pour vérifier qu’ils donnent le même résultat.
* Si ton composant n’est pas pur → tu verras des incohérences (par ex. compteurs qui doublent).

***

## ✅ Bonnes pratiques pour garder tes composants purs

1. 🚫 **Ne modifie jamais** :
   * les props
   * les objets/variables externes
   * le DOM directement (sauf via `useEffect`)
2. ✅ **Calcule uniquement à partir de tes entrées (props, état)**.
3. 🔄 Si tu dois avoir un effet (API, localStorage, timers…) → fais-le dans un **Hook comme `useEffect`**, pas dans le rendu.

👉 Un **composant React pur = une formule mathématique ou une recette de cuisine**.

***

## 🔬 1. Composant pur = formule mathématique

Une fonction mathématique, comme **y = 2x**, est **prévisible** :

* `x = 2` → `y = 4`
* `x = 3` → `y = 6`
* toujours le même résultat pour la même entrée.

En React, un composant doit suivre le même principe :

```jsx
function Recipe({ drinkers }) {
  return (
    <ol>
      <li>Faire bouillir {drinkers} tasses d’eau.</li>
      <li>
        Ajouter {drinkers} cuillers de thé et {0.5 * drinkers} cuillers d’épices.
      </li>
      <li>
        Ajouter {0.5 * drinkers} tasses de lait jusqu’à ébullition, et du sucre
        selon les goûts de chacun.
      </li>
    </ol>
  );
}
```

➡️ Si tu passes `drinkers={2}`, le JSX renvoyé est toujours identique.\
➡️ Si tu passes `drinkers={4}`, il change logiquement mais toujours de la même façon.

***

## 🍲 2. L’analogie de la recette de cuisine

Imagine un **livre de recettes** :

* Tu suis la recette à la lettre (les ingrédients = les **props**).
* Tu obtiens toujours **le même plat** si tu donnes les mêmes ingrédients.

⚠️ Mais si tu modifies les ingrédients au milieu (par ex. tu ajoutes du sel alors que ce n’est pas prévu), tu romps la pureté → ton composant devient **imprévisible**.

***

## ❌ 3. Exemple de composant impur

```jsx
let milk = 2;

function Recipe({ drinkers }) {
  milk = milk + 1; // ⚠️ modifie une variable externe
  return (
    <p>
      {drinkers} tasses d’eau et {milk} tasses de lait
    </p>
  );
}
```

* Si tu appelles `<Recipe drinkers={2} />` deux fois → tu n’auras pas le même rendu.
* La valeur dépend de **l’état externe** (variable `milk`).
* C’est comme une recette où tu changes la liste des ingrédients pendant la préparation → résultat aléatoire.

***

## ✅ 4. Exemple de composant pur

```jsx
function Recipe({ drinkers }) {
  return (
    <p>
      {drinkers} tasses d’eau et {0.5 * drinkers} tasses de lait
    </p>
  );
}
```

* Pas de dépendance externe.
* Pas de mutation.
* Même entrée → même sortie.

***

## 📌 5. Pourquoi React insiste sur la pureté

* 🔄 React peut **rappeler un composant plusieurs fois** (ex : StrictMode, Suspense, transitions).
* Si ton composant est impur → tu auras des bugs (valeurs qui changent, calculs doublés, affichage imprévisible).
* En gardant les composants purs → ton UI est **prévisible** et **stable**.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### 🚨 Pourquoi le premier exemple est impur

```jsx
let guest = 0;

function Cup() {
  // ⚠️ Effet de bord : on modifie une variable externe !
  guest = guest + 1;
  return <h2>Tasse de thé pour l’invité #{guest}</h2>;
}
```

* Ici `guest` est **déclaré hors du composant**.
* Chaque appel à `<Cup />` modifie la variable globale.
* Résultat : le rendu dépend de **l’historique des appels**, pas seulement des props.

➡️ Donc, si React appelle plusieurs fois ton composant (par ex. en **StrictMode**), tu n’auras pas le même rendu.

***

### ✅ Correction avec des props (composant pur)

```jsx
function Cup({ guest }) {
  return <h2>Tasse de thé pour l’invité #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

* Chaque `<Cup />` **ne dépend que de ses props**.
* Même entrée → même sortie.
* Pas de dépendance externe → composant **pur** ✅.

***

### 📚 Principe général

Un composant React doit être **prévisible** :

* ❌ Il ne doit **pas modifier** :
  * des variables globales,
  * des objets existants,
  * ou du DOM directement.
* ✅ Il doit se baser uniquement sur :
  * ses **props**,
  * son **state** (via `useState`),
  * ou son **context**.

Ces trois sources sont **en lecture seule pendant le rendu**.

***

### 🛡️ StrictMode à la rescousse

👉 En **développement uniquement**, React appelle chaque composant **deux fois** pour tester sa pureté.

* Si ton composant est pur → pas de problème.
* Si ton composant est impur (ex : il modifie une variable externe) → tu verras des bugs étranges comme des valeurs doublées.

Exemple d’anomalie en Mode Strict avec le code impur :

* Attendu : invité #1, #2, #3
* Affiché : invité #2, #4, #6

Pourquoi ? Parce que chaque rendu double l’incrémentation de `guest`.

⚠️ En prod, StrictMode **ne double pas** les rendus, donc tes utilisateurs ne verront pas ça, mais toi tu auras raté un bug latent.

### ⚡ Rappel : pourquoi éviter les mutations externes

Un composant React doit être **pur pendant le rendu** :

* Il ne doit **pas modifier** des variables globales.
* Il ne doit **pas altérer** des objets créés en dehors de son exécution.\
  Sinon, le rendu devient **imprévisible** (comme avec ton exemple `guest`).

***

### ✅ Mutation locale autorisée

Dans ton exemple :

```jsx
function TeaGathering() {
  let cups = []; // ✅ créé à chaque rendu
  for (let i = 1; i <= 12; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }
  return cups;
}
```

Ici :

* `cups` est **créé à l’intérieur** de `TeaGathering`, à chaque rendu.
* Le tableau n’existe **pas avant** l’appel, donc tu peux y faire des `push`.
* C’est un **secret local** : React s’en fiche, ça reste isolé.

👉 Ce genre de mutation est **inoffensive** car elle ne fuit pas à l’extérieur du composant.

***

### 🌍 Quand les effets de bord sont permis

Même si on évite les effets de bord dans le rendu, il y a des cas où ils sont nécessaires :

1. **Gestionnaires d’événements** (`onClick`, `onChange`…)\
   → Ils s’exécutent **après** le rendu, donc peuvent modifier l’état, lancer une animation, etc.
2. **useEffect**\
   → Pour les cas où il n’existe **pas de gestionnaire d’événement** adéquat (ex. : synchroniser un composant avec une API externe, un timer, du DOM manuel).

Exemple typique :

```jsx
import { useEffect } from "react";

function Timer() {
  useEffect(() => {
    const id = setInterval(() => {
      console.log("tic");
    }, 1000);

    return () => clearInterval(id); // nettoyage
  }, []);

  return <p>Regarde la console !</p>;
}
```

***

### 🚀 Pourquoi React tient tant à la pureté

1. **Exécution n’importe où**\
   → Sur le serveur, dans un worker, ou en streaming.
2. **Optimisations gratuites**\
   → React peut sauter un rendu ou en redémarrer un sans bug.
3. **Nouvelles fonctionnalités**\
   → Tout ce qui touche au rendu concurrent, au chargement de données, aux transitions, etc. repose sur cette garantie de pureté.

En résumé :\
👉 **Pendant le rendu** : pur, pas d’effets de bord.\
👉 **Après le rendu** : libre d’utiliser `useEffect` ou des gestionnaires d’événements.
