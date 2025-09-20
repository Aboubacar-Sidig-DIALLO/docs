# 📸 L’état est un instantané

Les variables d’état peuvent sembler être de simples variables que vous pouvez lire et écrire à volonté.\
⚠️ Mais en réalité, elles se comportent comme une **photo figée à un moment donné**.

👉 Quand vous appelez leur fonction de mise à jour (`setXxx`), React **ne modifie pas la valeur directement**.\
Au lieu de ça :

1. Il planifie un **nouveau rendu**.
2. Lors de ce rendu, votre composant reçoit une **nouvelle valeur** de l’état.

***

### 🔹 À retenir

* Une **variable classique** change immédiatement et garde sa valeur.
* Une **variable d’état** garde sa valeur **seulement le temps d’un rendu**.
* Modifier l’état → **ne change pas la valeur actuelle** → ça déclenche un **nouveau rendu** où l’état mis à jour sera utilisé.

***

### 🎯 Exemple simplifié

```jsx
import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    console.log(count); // ❌ Affiche encore l'ancienne valeur !
  }

  return (
    <>
      <p>Valeur : {count}</p>
      <button onClick={handleClick}>+1</button>
    </>
  );
}
```

👉 Si `count` vaut `0` et que vous cliquez une fois :

* React planifie `setCount(1)`
* Mais **pendant ce rendu**, `console.log(count)` affiche encore `0`.
* Au **rendu suivant**, `count` devient bien `1`.

***

### 🧠 Métaphore

Imaginez que chaque rendu soit une **photo polaroid 📷** de votre composant.

* Quand vous cliquez, vous demandez à React de prendre une **nouvelle photo** avec la nouvelle valeur.
* Mais la photo actuelle ne change jamais : elle reste figée dans le temps.

***

### 🔜 Ce que vous allez voir ensuite

* Comment une modification d’état déclenche un **nouveau rendu**.
* Pourquoi l’état **ne change pas immédiatement** après `setXxx`.
* Comment les gestionnaires d’événements accèdent toujours à **l’instantané du rendu courant**.

## ⚡ Modifier l’état déclenche un rendu

On pourrait croire que l’UI change **directement** après un clic ou une action de l’utilisateur.\
👉 En réalité, dans **React**, le changement d’UI se fait toujours **via une mise à jour d’état**.

Lorsqu’on appelle `setState`, React :

1. **Met à jour la valeur de l’état (en mémoire interne).**
2. **Planifie un nouveau rendu.**
3. **Re-exécute le composant** → qui produit une nouvelle version du JSX.

***

### 🔹 Exemple : un formulaire simple

```jsx
import { useState } from 'react';

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('Salut !');

  if (isSent) {
    return <h1>Votre message est en route !</h1>;
  }

  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      setIsSent(true); // ✅ met à jour l’état
      sendMessage(message); // envoie le message
    }}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Envoyer</button>
    </form>
  );
}

function sendMessage(message) {
  console.log("Message envoyé :", message);
}
```

***

### 🔎 Étapes quand on clique sur **Envoyer**

1. Le gestionnaire `onSubmit` s’exécute.
2. `setIsSent(true)` modifie l’état **et déclenche un nouveau rendu**.
3. React relance `Form()` → qui cette fois **renvoie le `<h1>`** au lieu du `<form>`.
4. L’UI s’actualise automatiquement.

***

✅ À retenir :

* **L’UI n’est jamais directement manipulée.**
* Tu modifies l’**état**, et React **refait le rendu** pour refléter la nouvelle situation.

## 📸 Le rendu prend une photo instantanée

« Faire le rendu » signifie que **React appelle votre composant (qui est une fonction)**.\
👉 Le JSX que votre fonction renvoie est **comme une photo instantanée** de l’UI à ce moment précis.

* Les **props**,
* les **gestionnaires d’événements**,
* et les **variables locales**

sont tous calculés **avec l’état disponible pour ce rendu précis**.

***

#### ⚡ Un instantané interactif

Contrairement à une simple photo figée, cet « instantané » contient aussi de la **logique interactive** (comme les gestionnaires d’événements).

* React affiche l’UI correspondant au JSX.
* Il relie les gestionnaires d’événements.
* Résultat : cliquer sur un bouton déclenche bien la fonction définie dans votre JSX.

***

### 🔄 Quand React refait un rendu

À chaque nouveau rendu :

1. React rappelle votre fonction composant.
2. Votre fonction renvoie un **nouvel instantané JSX**.
3. React met à jour le DOM pour refléter ce nouvel instantané.

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

### 📦 Où vit l’état ?

👉 L’état **ne vit pas dans la fonction composant**.\
Il est stocké **dans React lui-même** (comme sur une étagère 🗄️).

À chaque rendu, React **vous redonne la valeur actuelle** de l’état sous forme d’instantané :

* Vous travaillez avec cette valeur pendant ce rendu.
* React génère du JSX basé sur cet état.
* Puis il lie les nouveaux gestionnaires calculés.

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>



***

### 🧪 Expérience : le piège du `+3`

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

#### 🔎 Résultat :

➡️ Le compteur **n’augmente que de +1** par clic, pas de +3.

Pourquoi ? Parce que pendant ce rendu :

* `number` vaut `0`.
* Donc chaque `setNumber(number + 1)` = `setNumber(1)`.
* Après les 3 appels, React prépare juste **une mise à jour vers `1`**.

***

#### 📌 Substitution mentale

Rendu initial (`number = 0`) :

```jsx
setNumber(0 + 1);
setNumber(0 + 1);
setNumber(0 + 1);
// => number sera 1
```

Rendu suivant (`number = 1`) :

```jsx
setNumber(1 + 1);
setNumber(1 + 1);
setNumber(1 + 1);
// => number sera 2
```

👉 C’est pour ça qu’on parle **d’instantané** :\
chaque gestionnaire d’événement « capture » l’état tel qu’il était **au moment du rendu**.

***

⚠️ **À retenir**

* L’état est une **photo instantanée par rendu**.
* `setState` ne change pas immédiatement la valeur utilisée dans ce rendu.
* Les mises à jour sont appliquées **au prochain rendu**.

## 🕒 L’état au fil du temps

En React, **l’état ne change pas à l’intérieur d’un rendu**. Chaque rendu reçoit une **photo figée** (un snapshot 📸) des valeurs d’état au moment où la fonction composant a été appelée.

***

### 🎯 Exemple simple avec alerte directe

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 5);
          alert(number);
        }}
      >
        +5
      </button>
    </>
  );
}
```

👉 Ici :

* `setNumber(number + 5)` **programme un nouveau rendu** avec `number + 5`.
* Mais dans ce **rendu actuel**, `number` vaut encore `0`.
* Donc `alert(number)` affiche **0**.

***

### 🕰️ Exemple avec `setTimeout`

```jsx
<button
  onClick={() => {
    setNumber(number + 5);
    setTimeout(() => {
      alert(number);
    }, 3000);
  }}
>
  +5
</button>
```

👉 Après 3 secondes, l’alerte affiche toujours **0**, même si `number` a été mis à jour dans React.

Pourquoi ? Parce que la fonction passée à `setTimeout` **capture l’instantané de l’état du rendu courant** → ici, `number = 0`.

***

📌 **schéma mental:**

1. Rendu actuel → `number = 0`.
2. On clique → `setNumber(5)` programme un nouveau rendu.
3. Le `setTimeout` garde `number = 0` car il « fige » la valeur.
4. Au bout de 3 secondes → l’alerte utilise encore l’ancien `number`.

***

### ✉️ Exemple du formulaire avec délai

```jsx
import { useState } from 'react';

export default function Form() {
  const [to, setTo] = useState('Alice');
  const [message, setMessage] = useState('Bonjour');

  function handleSubmit(e) {
    e.preventDefault();
    setTimeout(() => {
      alert(`Vous avez dit ${message} à ${to}`);
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Destinataire :{' '}
        <select value={to} onChange={e => setTo(e.target.value)}>
          <option value="Alice">Alice</option>
          <option value="Bob">Bob</option>
        </select>
      </label>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Envoyer</button>
    </form>
  );
}
```

👉 Scénario :

1. Vous écrivez **Bonjour** à Alice et cliquez sur « Envoyer ».
2. Avant 5 secondes, vous changez le destinataire en Bob.
3. L’alerte affiche **« Vous avez dit Bonjour à Alice »** ✅

Pourquoi ? Parce que le `setTimeout` a capturé l’**instantané** des valeurs (`message = Bonjour`, `to = Alice`) au moment du clic.

***

### ✅ Ce qu’il faut retenir

* Chaque rendu = **une photo immuable de l’état**.
* Les gestionnaires d’événements utilisent l’état du rendu où ils ont été créés.
* Même si du temps passe (ex. `setTimeout`, `fetch`, etc.), ils gardent **l’instantané** de cet état.
* 👉 Pour utiliser la **dernière valeur** au moment d’un calcul, il faut passer par une **mise à jour fonctionnelle** (`setState(prev => ...)`) → qu’on verra dans la suite.
