# ⚡Mettre en file une série de mises à jour d’état

### 🔎 Le problème

Quand on appelle plusieurs fois une fonction de mise à jour d’état (`setX`), on pourrait croire que chaque appel modifie directement la variable d’état.\
👉 Mais en réalité, **React ne change pas la variable immédiatement**.

➡️ Chaque appel à `setX` **met en file (queue)** une mise à jour.\
➡️ React attend la fin de l’événement (clic, saisie, etc.) pour **traiter toutes les mises à jour en lot** → c’est le **batching**.

***

### 🧩 Exemple simple

```jsx
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 1);
          setNumber(number + 1);
          setNumber(number + 1);
        }}
      >
        +3
      </button>
    </>
  );
}
```

👉 Résultat : `+3` affiche **1** au lieu de **3**.

#### Pourquoi ?

* Lors de ce rendu, `number = 0`.
* Chaque `setNumber(number + 1)` → équivaut à `setNumber(1)`.
* React regroupe ces 3 mises à jour → **dernier ordre appliqué : 1**.

***

### ✅ La bonne approche : mise à jour fonctionnelle

Si la **nouvelle valeur dépend de l’ancienne**, il faut utiliser la **version fonctionnelle** de `setState` :

```jsx
<button
  onClick={() => {
    setNumber(n => n + 1);
    setNumber(n => n + 1);
    setNumber(n => n + 1);
  }}
>
  +3
</button>
```

👉 Résultat : compteur = **3**.

#### Comment ça marche ?

* `setNumber(n => n + 1)` prend toujours **la valeur la plus récente dans la file**.
* React applique les updates les unes après les autres :
  * `0 → 1`
  * `1 → 2`
  * `2 → 3`

***

### 🧠 Métaphore

Imagine une file d’attente dans un restaurant 🍽️ :

* Tu donnes **3 tickets** au serveur (`+1`, `+1`, `+1`).
* Le serveur les applique **dans l’ordre** → ton compteur arrive bien à **3**.

Mais si tu donnes **3 fois le même ticket “mettre à 1”** → résultat = **1**.

***

### 📦 À retenir

1. **Batching (traitement par lots)** :\
   React regroupe les mises à jour d’un même cycle pour optimiser les performances.
2. **Mise à jour directe (`setX(valeur)`)** :
   * Utilise l’instantané du rendu actuel.
   * Écrase → ⚠️ attention aux updates successives.
3. **Mise à jour fonctionnelle (`setX(prev => ...)`)** :
   * Utilise la valeur **la plus récente** disponible.
   * Permet d’appliquer plusieurs updates correctement.

## ⚛️ React regroupe les mises à jour d’état (Batching)

Vous pourriez vous attendre à ce qu’un clic sur le bouton « +3 » incrémente le compteur trois fois, puisqu’il appelle `setNumber(number + 1)` trois fois :

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```

***

Cependant, comme vous vous en souvenez peut-être de la section précédente, les valeurs d’état d’un rendu sont **figées**.\
👉 Cela signifie que la valeur de `number` dans le gestionnaire d’événement du premier rendu est toujours `0`, peu importe combien de fois vous appelez `setNumber(1)` :

```js
setNumber(0 + 1);
setNumber(0 + 1);
setNumber(0 + 1);
```

***

Mais un autre facteur entre en jeu.\
➡️ React attend que **tout le code du gestionnaire d’événement** soit exécuté avant de traiter vos mises à jour d’état.\
👉 C’est pourquoi le nouveau rendu n’a lieu **qu’après** tous les appels à `setNumber()`.

***

#### 🍽️ Métaphore du restaurant

Cela ressemble à un serveur qui prend une commande au restaurant.

* Le serveur ne court pas en cuisine dès que vous mentionnez votre premier plat !
* Il vous laisse finir votre commande, y apporter des modifications, et même prendre les commandes des autres personnes à table.

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

***

#### ✅ Pourquoi c’est utile ?

Ce comportement permet :

* de mettre à jour plusieurs variables d’état — même venant de plusieurs composants — **sans déclencher trop de re-rendus**,
* et d’éviter des affichages partiels ou incohérents où seuls certains états seraient à jour.

Ce regroupement de mises à jour est appelé **batching**.\
Il rend vos applis React **beaucoup plus rapides** et évite les rendus « à moitié terminés ».

***

⚠️ Remarque : React **ne regroupe pas** les mises à jour à travers plusieurs événements distincts (comme deux clics séparés).\
Chaque clic est traité séparément.\
👉 Cela garantit par exemple que si le premier clic désactive un formulaire, le deuxième clic ne pourra pas le soumettre à nouveau.

## 🔄 Mettre à jour plusieurs fois le même état avant le prochain rendu

C’est un cas relativement rare, mais si vous voulez mettre à jour la **même variable d’état plusieurs fois avant le prochain rendu**, au lieu de passer directement la valeur suivante comme `setNumber(number + 1)`, vous pouvez passer une fonction qui calcule la valeur suivante **à partir de l’état précédent**, comme `setNumber(n => n + 1)`.

👉 C’est une façon de dire à React :\
« Fais quelque chose avec la valeur d’état actuelle » plutôt que de simplement la remplacer.

Essayez maintenant d’incrémenter le compteur :

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
```

***

Ici, `n => n + 1` est appelée une **fonction de mise à jour** (updater function).\
Lorsque vous la passez à un setter d’état :

1. React met cette fonction **dans une file d’attente** à exécuter une fois que tout le code du gestionnaire d’événement est terminé.
2. Lors du prochain rendu, React parcourt cette file et vous donne l’état final calculé.

Exécution de ces lignes :

```js
setNumber(n => n + 1);
setNumber(n => n + 1);
setNumber(n => n + 1);
```

➡️ Voici ce que fait React :

| Mise à jour en file d’attente | `n` reçu | Résultat |
| ----------------------------- | -------- | -------- |
| `n => n + 1`                  | 0        | 1        |
| `n => n + 1`                  | 1        | 2        |
| `n => n + 1`                  | 2        | 3        |

React stocke **3** comme résultat final et le renvoie via `useState`.

👉 C’est pourquoi le bouton « +3 » incrémente bien de **3**.

***

## 🔁 Et si on mélange “remplacer” et “mettre à jour” ?

Regardez ce gestionnaire d’événement. Que pensez-vous que `number` vaudra au prochain rendu ?

```jsx
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
}}>
  Increase the number
</button>
```

Exemple complet :

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
      }}>Increase the number</button>
    </>
  )
}
```

***

Voici ce que ce gestionnaire dit à React :

1. `setNumber(number + 5)` : ici `number` vaut `0`, donc React ajoute **“remplacer par 5”** dans la file d’attente.
2. `setNumber(n => n + 1)` : React ajoute la fonction `n => n + 1` dans la file d’attente.

***

Lors du prochain rendu, React exécute la file d’attente :

| Mise à jour en file d’attente | `n` reçu    | Résultat |
| ----------------------------- | ----------- | -------- |
| “remplacer par 5”             | (0, ignoré) | 5        |
| `n => n + 1`                  | 5           | 6        |

➡️ Résultat final : **6**

***

📌 **Remarque** :\
Vous avez peut-être remarqué que `setState(5)` se comporte en réalité comme `setState(n => 5)`.\
La différence est que dans ce cas, `n` est simplement ignoré.

## 🔄 Que se passe-t-il si vous **remplacez l’état après l’avoir mis à jour** ?

Essayons encore un exemple. D’après vous, quelle sera la valeur de `number` au **prochain rendu** ?

```jsx
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
  setNumber(42);
}}>
  Increase the number
</button>
```

***

### Exemple complet

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
        setNumber(42);
      }}>
        Increase the number
      </button>
    </>
  )
}
```

***

### 🔎 Comment React exécute ces lignes dans ce gestionnaire :

1. `setNumber(number + 5)` :\
   `number` vaut **0**, donc React ajoute **“remplacer par 5”** dans la file d’attente.
2. `setNumber(n => n + 1)` :\
   React ajoute la fonction `n => n + 1` dans la file d’attente.
3. `setNumber(42)` :\
   React ajoute **“remplacer par 42”** dans la file d’attente.

***

### 🚀 Lors du prochain rendu, React parcourt la file d’attente :

| Mise à jour en file d’attente | `n` reçu   | Résultat |
| ----------------------------- | ---------- | -------- |
| “remplacer par 5”             | 0 (ignoré) | 5        |
| `n => n + 1`                  | 5          | 6        |
| “remplacer par 42”            | 6 (ignoré) | 42       |

➡️ Résultat final : **42** ✅

***

### 📝 À retenir

* Une **fonction de mise à jour** (ex. `n => n + 1`) est ajoutée dans la file d’attente.
* Toute autre valeur (ex. `42`) ajoute **“remplacer par 42”** à la file, ce qui écrase les calculs précédents.
* Une fois le gestionnaire terminé, React déclenche un **nouveau rendu** et traite toute la file.
* ⚠️ Les fonctions de mise à jour s’exécutent **pendant le rendu** → elles doivent être **pures** (sans effets de bord).
  * ❌ Ne pas déclencher de `setState` ou d’effet secondaire depuis une fonction updater.
  * ✅ En Mode Strict, React exécute chaque fonction de mise à jour **deux fois** (en jetant le deuxième résultat) pour aider à détecter les erreurs.

***

### 🧑‍💻 Conventions de nommage

Il est courant de nommer l’argument de la fonction updater par les premières lettres du state correspondant :

```js
setEnabled(e => !e);
setLastName(ln => ln.reverse());
setFriendCount(fc => fc * 2);
```

Si vous préférez un code plus **verbeux et lisible**, vous pouvez :

*   répéter le nom complet du state :

    {% code fullWidth="true" %}
    ```js
    setEnabled(enabled => !enabled);
    ```
    {% endcode %}
*   ou utiliser un préfixe comme `prev` :

    ```js
    setEnabled(prevEnabled => !prevEnabled);
    ```

### 📌 En résumé

1. **Définir l’état ≠ changer immédiatement la variable**
   * Appeler `setState` ne modifie pas la valeur utilisée dans le rendu en cours.
   * Cela demande simplement à React de faire un **nouveau rendu** avec la nouvelle valeur.
2. **Traitement par lots (batching)**
   * React attend la fin d’un gestionnaire d’événement avant d’appliquer toutes les mises à jour d’état en une seule fois.
   * Ça évite des re-rendus inutiles et rend l’app plus rapide.
3. **Mises à jour multiples d’un même état**
   *   Si vous voulez enchaîner plusieurs mises à jour, utilisez une **fonction de mise à jour** :

       ```js
       setNumber(n => n + 1);
       ```
   * Ainsi, React applique chaque fonction sur la valeur la plus récente de la file d’attente, et pas sur l’« instantané » du rendu en cours.
