# 🔖 Référencer des valeurs avec les Refs(Referencing Values with Refs)

👉 **Problème** : parfois, un composant doit _mémoriser_ une information **sans déclencher de re-render**.\
Exemple :

* un compteur qui s’incrémente mais dont la valeur ne change pas l’affichage
* l’ID d’un `setTimeout`
* une référence vers un élément DOM

Dans ces cas-là, on utilise un **ref** au lieu du **state**.

***

### 🔹 Créer un ref

Tu utilises le Hook `useRef` :

```jsx
import { useRef } from "react";

export default function Counter() {
  const ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert("You clicked " + ref.current + " times!");
  }

  return <button onClick={handleClick}>Click me!</button>;
}
```

***

### 🔹 Comment ça marche ?

* `const ref = useRef(initialValue)` crée un objet `{ current: initialValue }`.
* La valeur est **persistante entre les re-renders** (comme le state).
* **Différence avec le state** :
  * `setState` → déclenche un re-render.
  * `ref.current = ...` → ne déclenche pas de re-render.

👉 Un `ref` est donc comme une **poche secrète** dans ton composant.

***

### 🔹 Différence entre **state** et **ref**

| Aspect                                       | `useState`                                          | `useRef`                                                                         |
| -------------------------------------------- | --------------------------------------------------- | -------------------------------------------------------------------------------- |
| Stocke une valeur persistante ?              | ✅ Oui                                               | ✅ Oui                                                                            |
| Déclenche un re-render quand on le modifie ? | ✅ Oui                                               | ❌ Non                                                                            |
| Usage typique                                | Données qui affectent le rendu (ex: formulaire, UI) | Données qui n’affectent pas le rendu (ex: timer ID, compteur interne, accès DOM) |

***

### 🔹 Quand utiliser `refs`

* Stocker des **valeurs temporaires** qui n’affectent pas l’UI.
* Conserver des **objets mutables** (ex: WebSocket, lecteur vidéo).
* Référencer un **élément DOM** pour le manipuler.

⚠️ Attention : **n’utilise pas les refs comme une alternative au state**.\
Si ta donnée doit **affecter l’affichage**, elle doit être dans le **state**, pas dans un `ref`.

***

### ✅ À retenir

1. `useRef` → crée un objet persistant entre les re-renders.
2. Modifier `ref.current` → **ne redessine pas** ton composant.
3. Utilise `refs` pour :
   * stocker des valeurs hors rendu,
   * garder des références à des éléments DOM,
   * gérer des identifiants (timer, animations…).



## 🔖 Ajouter une ref à ton composant

👉 Tu peux ajouter une ref à ton composant en important le Hook `useRef` depuis React :

```jsx
import { useRef } from 'react';
```

À l’intérieur de ton composant, appelle `useRef` et passe la valeur initiale que tu veux mémoriser comme argument.\
Par exemple, voici une ref pointant sur la valeur **0** :

```jsx
const ref = useRef(0);
```

`useRef` retourne un objet comme celui-ci :

```js
{ 
  current: 0 // La valeur passée à useRef
}
```

***

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

***

Tu peux accéder à la valeur actuelle d’une ref via sa propriété **`ref.current`**.\
👉 Cette valeur est **mutable**, donc tu peux la lire **et** la modifier.\
C’est comme une **poche secrète de ton composant** que React ne suit pas.

⚡ C’est ce qui en fait une **"échappatoire" (escape hatch)** au flux de données unidirectionnel de React (on en reparlera plus loin).

***

### Exemple : incrémenter une ref au clic

```jsx
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('Tu as cliqué ' + ref.current + ' fois !');
  }

  return (
    <button onClick={handleClick}>
      Clique-moi !
    </button>
  );
}
```

***

#### 🔍 Points importants :

* Ici, la ref pointe vers un **nombre**, mais tu pourrais aussi stocker :
  * une chaîne (`string`),
  * un objet (`object`),
  * une fonction,
  * ou même un élément du DOM.
* Contrairement au **state**, une ref est un **simple objet JavaScript** avec une propriété `current`.
* **Le composant ne se re-render pas** à chaque incrément.
* Comme le state, les refs sont **retenues entre les re-renders**.
* ⚠️ Mais contrairement au state, **changer une ref ne déclenche pas de nouveau rendu** !

***

## 🕒 Exemple pratique : un chronomètre (stopwatch)

👉 Tu peux combiner **state** et **ref** dans un même composant.

Ici, on veut créer un chronomètre que l’utilisateur peut démarrer ou arrêter avec un bouton.

#### 1. Démarrage du chronomètre

On a besoin de suivre :

* Quand l’utilisateur a cliqué sur **Start** → `startTime`
* L’heure actuelle → `now`

Ces infos sont utilisées pour l’affichage → donc on les met dans le **state** :

```jsx
const [startTime, setStartTime] = useState(null);
const [now, setNow] = useState(null);
```

Ensuite, quand l’utilisateur clique sur Start, on met à jour toutes les 10ms avec `setInterval` :

```jsx
import { useState } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);

  function handleStart() {
    // Lancer le chronomètre
    setStartTime(Date.now());
    setNow(Date.now());

    setInterval(() => {
      // Mettre à jour l’heure actuelle toutes les 10 ms
      setNow(Date.now());
    }, 10);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Temps écoulé : {secondsPassed.toFixed(3)} s</h1>
      <button onClick={handleStart}>Start</button>
    </>
  );
}
```

***

#### 2. Arrêt du chronomètre avec une ref

Quand l’utilisateur clique sur **Stop**, on doit **annuler l’intervalle**.\
Mais pour faire `clearInterval`, on a besoin de l’ID retourné par `setInterval`.

👉 Cet ID **n’est pas utilisé pour le rendu**, donc on peut le stocker dans une **ref** :

```jsx
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null); // Stocke l’ID de l’intervalle

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current); // Nettoie l’ancien intervalle
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Temps écoulé : {secondsPassed.toFixed(3)} s</h1>
      <button onClick={handleStart}>Start</button>
      <button onClick={handleStop}>Stop</button>
    </>
  );
}
```

***

### ✅ Règle d’or

* Si une donnée est **utilisée pour l’affichage** → utilise **state**.
* Si une donnée est **utile seulement dans la logique interne** (timer, référence DOM, WebSocket, etc.) → utilise une **ref**.

## 🔍 Différences entre **refs** et **state**

Peut-être que tu trouves que les refs semblent moins “strictes” que le state :

* Avec une ref tu peux **muter (modifier)** directement `ref.current`.
* Avec le state tu dois obligatoirement passer par la fonction de mise à jour (`setState`).

👉 Mais dans la majorité des cas, **tu voudras utiliser le state**.\
Les **refs** sont un **“échappatoire” (escape hatch)** que tu n’utiliseras pas souvent.

***

### ⚖️ Tableau comparatif

| **refs**                                                                 | **state**                                                                          |
| ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| `useRef(initialValue)` retourne `{ current: initialValue }`              | `useState(initialValue)` retourne `[value, setValue]`                              |
| **Ne déclenche pas** de re-render quand on le modifie                    | **Déclenche** un re-render quand on le modifie                                     |
| **Mutable** : tu peux modifier `current` en dehors du processus de rendu | **“Immutable”** : tu dois utiliser la fonction setter pour programmer un re-render |
| ❌ Tu ne devrais pas **lire ou écrire `ref.current`** pendant le rendu    | ✅ Tu peux lire le state à tout moment (chaque rendu a son snapshot cohérent)       |

***

### Exemple 1 : compteur avec **state**

Ici, la valeur est affichée → donc **state** est le bon choix.

```jsx
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      Vous avez cliqué {count} fois
    </button>
  );
}
```

👉 Quand tu cliques, `setCount()` met à jour la valeur et React **re-render** le composant → l’écran reflète le nouveau compteur.

***

### Exemple 2 : compteur avec **ref**

Ici, la valeur n’est jamais ré-affichée car **modifier une ref ne déclenche pas de re-render** :

```jsx
import { useRef } from 'react';

export default function Counter() {
  let countRef = useRef(0);

  function handleClick() {
    // ⚠️ Ça ne déclenche PAS de re-render
    countRef.current = countRef.current + 1;
  }

  return (
    <button onClick={handleClick}>
      Vous avez cliqué {countRef.current} fois
    </button>
  );
}
```

👉 Ici, le nombre ne change jamais à l’écran, même si la valeur interne de la ref s’incrémente.\
C’est pourquoi **lire `ref.current` pendant le rendu donne du code non fiable**.

⚠️ Si tu as besoin d’afficher une valeur, **utilise le state** !

***

### 🧠 Plongée plus profonde : comment `useRef` fonctionne

En réalité, `useRef` peut être vu comme une **variante spéciale du state**.\
On pourrait l’implémenter ainsi :

```js
// À l’intérieur de React
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

* Au **premier rendu**, `useRef` retourne `{ current: initialValue }`.
* Cet objet est **conservé par React**, et **réutilisé** à chaque nouveau rendu.
* La fonction `setState` existe (car on l’a utilisé avec `useState`), mais on ne l’utilise pas → car une ref doit toujours renvoyer **le même objet**.

👉 On peut donc voir une **ref** comme un **state sans setter**.

***

### ⚙️ Métaphore OOP

Si tu viens de la POO (Programmation Orientée Objet) :

* le **state** ressemble à des **propriétés immuables**, qui déclenchent un re-render quand elles changent.
*   les **refs** ressemblent à des **champs d’instance** (`this.something`), mais ici tu écris :

    ```js
    somethingRef.current
    ```

***

✅ En résumé :

* Utilise **state** pour les données affichées ou qui impactent l’UI.
* Utilise **refs** pour stocker des infos invisibles pour le rendu (timer ID, élément du DOM, cache, etc.).

## 📌 Quand utiliser les refs ?

En général, tu vas utiliser une **ref** quand ton composant doit **“sortir” de React** pour interagir avec une API externe — souvent une **API du navigateur** — et que cette donnée **n’a pas d’impact sur le rendu de ton JSX**.

👉 En d’autres termes :

* Si la valeur influence **l’UI** → utilise **state**.
* Si la valeur est seulement **technique** ou **externe** (et n’affecte pas l’affichage) → utilise **ref**.

***

### ✅ Cas d’usage typiques des refs

1.  **Stocker des IDs de timeout ou d’intervalle**

    ```js
    const timeoutRef = useRef(null);

    function startTimer() {
      timeoutRef.current = setTimeout(() => {
        console.log("Timer terminé");
      }, 1000);
    }

    function clearTimer() {
      clearTimeout(timeoutRef.current);
    }
    ```

    Ici, `timeoutRef` stocke l’ID renvoyé par `setTimeout`.\
    Ce n’est pas une donnée à afficher, donc inutile d’utiliser `state`.

***

2.  **Référencer et manipuler des éléments du DOM**\
    Par exemple, pour mettre le focus sur un input :

    ```js
    const inputRef = useRef(null);

    function focusInput() {
      inputRef.current.focus();
    }

    return <input ref={inputRef} />;
    ```

    👉 React gère le DOM via le rendu, mais parfois tu as besoin d’appeler directement une méthode d’API du navigateur (`focus()`, `scrollIntoView()`, etc.).

***

3.  **Stocker des objets “techniques”** qui n’ont pas d’impact visuel\
    Exemple : une connexion à un WebSocket, une instance de lecteur vidéo, une bibliothèque externe, etc.

    ```js
    const socketRef = useRef(null);

    useEffect(() => {
      socketRef.current = new WebSocket("wss://exemple.com");
      return () => socketRef.current.close();
    }, []);
    ```

    👉 Ici, tu stockes une instance externe que React n’a pas besoin de suivre pour le rendu.

***

### ⚠️ Règle d’or

* Utilise **state** ➝ pour tout ce qui influence **le rendu JSX** (exemple : texte d’un input, compteur affiché, état d’un bouton).
* Utilise **refs** ➝ pour stocker des **valeurs invisibles** au rendu mais nécessaires au fonctionnement (timers, DOM, objets externes).

***

👉 Donc, si ton composant a besoin de mémoriser une valeur **qui n’impacte pas l’affichage**, choisis **refs**.

## ✅ Bonnes pratiques pour les refs

#### 1. Considère les refs comme une **issue de secours** (escape hatch)

* Utilise-les seulement quand tu dois **sortir de React** : accéder à une API du navigateur (focus, scroll, vidéo, canvas…), gérer des timers (`setTimeout`, `setInterval`), ou manipuler un objet externe (WebSocket, player, etc.).
* ⚠️ Si beaucoup de ta logique repose sur les refs → ton architecture n’est peut-être pas la bonne. Privilégie `state` et les props pour la gestion des données.

***

#### 2. 🚫 Ne lis pas / n’écris pas `ref.current` pendant le rendu

*   **Mauvais** :

    ```js
    function MyComponent() {
      if (ref.current) { // ❌ lecture pendant le rendu
        doSomething(ref.current);
      }
      return <div ref={ref} />;
    }
    ```
*   **Exception tolérée** :

    ```js
    if (!ref.current) ref.current = new Thing(); // initialisation une seule fois
    ```

👉 Si une donnée est utilisée pour **afficher** du JSX, elle doit être dans le **state**, pas dans une ref.\
Sinon ton composant devient imprévisible, car React ne sait pas quand `ref.current` change.

***

#### 3. Comprends que les refs n’ont pas les **limitations du state**

* `state` agit comme une **photo instantanée** par rendu.
*   `ref.current` est **mutable et immédiat** :

    ```js
    ref.current = 5;
    console.log(ref.current); // 5 directement
    ```
* Ça marche ainsi car une ref n’est qu’un **objet JS normal** : `{ current: valeur }`.

👉 Tu n’as pas à craindre la mutation (contrairement au state). Tant que ce n’est pas utilisé pour le rendu, React s’en fiche.

***

#### 4. 🖼️ Refs et le DOM

*   Le cas d’usage le plus courant est d’attacher une ref à un **élément DOM** :

    ```js
    const inputRef = useRef(null);

    return <input ref={inputRef} />;
    ```
* Quand l’élément existe → `inputRef.current` pointe dessus.
* Quand il est démonté → React remet `inputRef.current = null`.

👉 Exemple : donner le focus automatiquement à un input :

```js
useEffect(() => {
  inputRef.current.focus();
}, []);
```

***

## 🔑 Récapitulatif

* 🔹 Les refs = une **poche privée** dans ton composant, pour garder des infos **non liées au rendu**.
* 🔹 Ce sont des objets JS `{ current: ... }`, obtenus avec `useRef()`.
* 🔹 Elles survivent aux re-renders, comme le state.
* 🔹 Mais modifier une ref **ne déclenche pas de re-render**.
* 🔹 Évite de lire ou écrire `ref.current` pendant le rendu → mets en state si c’est pour l’affichage.
* 🔹 Principaux cas : timers, DOM, objets techniques externes.
