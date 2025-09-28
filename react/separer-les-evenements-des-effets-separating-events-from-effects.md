# 🎭 Séparer les Événements des Effets(Separating Events from Effects)

### 🔹 Différence fondamentale

*   **Event Handlers (gestionnaires d’événements)**\
    👉 Ils s’exécutent **uniquement quand l’utilisateur interagit** (clic, submit, scroll, etc.).\
    👉 Ils **ne se relancent pas automatiquement** après un re-render.\
    👉 Exemple :

    ```jsx
    function Form() {
      function handleSubmit(e) {
        e.preventDefault();
        sendForm();
      }
      return <form onSubmit={handleSubmit}>...</form>;
    }
    ```

***

*   **Effects (`useEffect`)**\
    👉 Ils s’exécutent **quand les valeurs réactives changent** (props, state, variables calculées dans le composant).\
    👉 Ils servent à **synchroniser ton composant avec un système externe** (API, WebSocket, animation, DOM…).\
    👉 Exemple :

    ```jsx
    function ChatRoom({ roomId }) {
      useEffect(() => {
        const conn = createConnection(roomId);
        conn.connect();
        return () => conn.disconnect();
      }, [roomId]);
    }
    ```

***

### 🔹 Pourquoi ça coince parfois ?

👉 Tu veux un effet qui réagit à certains changements, **mais pas à d’autres**.

Exemple :

```jsx
useEffect(() => {
  // ❌ Mauvais : toute cette logique est réactive
  if (isLoggedIn) {
    postAnalyticsEvent(userId);
  }
  sendWelcomeEmail(userId);
}, [isLoggedIn, userId]);
```

👉 Ici, `postAnalyticsEvent` devrait **se lancer uniquement au moment du login** (événement utilisateur).\
👉 Mais `useEffect` va le rejouer chaque fois que `userId` change → donc trop souvent.

***

### 🔹 La solution : **séparer l’événement de l’effet**

#### 1. La partie réactive (dans `useEffect`)

Elle s’exécute **uniquement quand il faut synchroniser** avec des dépendances (API, socket, etc.).

#### 2. La partie événementielle (dans un handler ou un "Effect Event")

Elle contient la logique qui doit s’exécuter **au bon moment**, pas à chaque re-render.

***

### 🔹 Exemple avec **Effect Event**

React propose un pattern appelé **Effect Event** : tu déclares une fonction spéciale **dans l’effet** mais **non réactive**, que tu peux appeler sans que `useEffect` se resynchronise à chaque fois.

```jsx
import { useEffect, useState } from "react";

function WelcomeMessage({ userId, isLoggedIn }) {
  // ✅ "Effect Event" : non réactif par défaut
  function sendWelcomeEmail(id) {
    console.log(`📧 Email envoyé à ${id}`);
  }

  useEffect(() => {
    if (isLoggedIn) {
      sendWelcomeEmail(userId);
    }
  }, [isLoggedIn]); // dépend seulement de l'état "isLoggedIn"
}
```

👉 Ici, `sendWelcomeEmail` lit bien `userId`, mais il **n’est pas réactif** :

* L’effet ne se déclenche que quand `isLoggedIn` change.
* `userId` sera toujours lu avec sa **dernière valeur à jour**.

***

### 🔹 Quand utiliser quoi ?

| Situation                                                                          | À utiliser        |
| ---------------------------------------------------------------------------------- | ----------------- |
| L’utilisateur fait une action spécifique (clic, submit, drag…)                     | **Event Handler** |
| Le composant doit rester synchronisé avec un système externe (API, WebSocket, DOM) | **Effect**        |
| Tu veux mixer : effet réactif **+ lecture de valeurs actuelles sans re-sync**      | **Effect Event**  |

***

### 🔑 Points à retenir

* Les **Effets réagissent aux valeurs réactives** → ils se rejouent dès que ça change.
* Les **Événements ne sont pas réactifs** → ils s’exécutent uniquement sur interaction.
* Si tu veux lire une valeur dans un `useEffect` **sans la mettre en dépendance**, isole cette partie en **Effect Event**.

## 🎯 Choisir entre **Event Handlers** et **Effects**

***

### 🔹 1. Rappel des rôles

* **Event Handler**\
  👉 Code déclenché **par une interaction utilisateur spécifique** (clic, submit, scroll…).\
  👉 Ne se relance **pas tout seul** après un re-render.
* **Effect (`useEffect`)**\
  👉 Code déclenché **quand une valeur réactive change** (props, state, contexte…).\
  👉 Sert à **synchroniser un composant** avec un système externe (API, WebSocket, DOM).

***

### 🔹 2. Exemple : Chat Room

#### ✅ Règle 1 : Connexion automatique à une room

👉 Ce comportement doit se déclencher **dès que `roomId` change**, sans interaction utilisateur.\
👉 Ici, c’est une **synchronisation avec un système externe (serveur de chat)** → donc un **Effet**.

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();

    return () => connection.disconnect();
  }, [roomId]); // Dépend de roomId
}
```

***

#### ✅ Règle 2 : Envoyer un message quand je clique "Send"

👉 Ici, le déclencheur est **l’action de l’utilisateur (clic sur un bouton)**.\
👉 Donc il faut un **Event Handler**, pas un effet.

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState("");

  function handleSendClick() {
    sendMessage(roomId, message);
    setMessage(""); // reset input
  }

  return (
    <>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>Send</button>
    </>
  );
}
```

***

### 🔹 3. Résumé de la logique

| Besoin                                        | Où mettre le code ?                        | Exemple             |
| --------------------------------------------- | ------------------------------------------ | ------------------- |
| **Se connecter automatiquement** à la room    | `useEffect` (synchronisation avec serveur) | Connexion WebSocket |
| **Envoyer un message en cliquant** sur "Send" | Event Handler (interaction utilisateur)    | `onClick` bouton    |

***

### ✅ Règle d’or

👉 Pose-toi toujours la question :\
&#xNAN;**“Pourquoi ce code doit-il s’exécuter ?”**

* Si c’est parce que **l’utilisateur fait quelque chose** → **Event Handler**.
* Si c’est parce que **un état/prop a changé** et qu’il faut **synchroniser avec un système externe** → **Effect**.

## ⚡ Les **Event Handlers** en React

***

### 🔹 1. Définition

👉 Un **event handler** est une fonction qui s’exécute **uniquement** en réponse à une interaction utilisateur **spécifique** (clic, saisie, scroll, soumission de formulaire, etc.).

Il ne se relance **jamais tout seul** lors d’un re-render — il dépend **100 % de l’action de l’utilisateur**.

***

### 🔹 2. Exemple : ChatRoom avec bouton "Send"

Ici, l’envoi du message doit se produire **uniquement quand l’utilisateur clique sur le bouton** :

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function handleSendClick() {
    sendMessage(message); // ✅ Ne s'exécute QUE si l'utilisateur clique
  }

  return (
    <>
      <input 
        value={message} 
        onChange={e => setMessage(e.target.value)} 
        placeholder="Écris ton message..." 
      />
      <button onClick={handleSendClick}>Send</button>
    </>
  );
}
```

***

### 🔹 3. Pourquoi **pas** dans un `useEffect` ?

Si tu mettais `sendMessage(message)` dans un `useEffect` dépendant de `message`, chaque frappe de clavier ou re-render déclencherait l’envoi. Résultat : le message partirait **sans clic**, ce qui est une mauvaise UX.

👉 Exemple à **éviter** :

```jsx
// ❌ Mauvais : déclenché automatiquement à chaque changement
useEffect(() => {
  sendMessage(message);
}, [message]);
```

***

### ✅ Règle d’or

* ❌ **Pas dans un `useEffect`** → sinon il s’exécute à cause d’un changement d’état.
* ✅ **Dans un Event Handler** → il ne s’exécute que sur **l’action explicite** de l’utilisateur.

xactement ✅ Là tu mets le doigt sur la différence fondamentale entre **event handlers** et **effects**.

***

## ⚡ Différence clé : Event Handlers vs Effects

### 🔹 Event Handlers

* S’exécutent uniquement en réponse à une **interaction précise** (clic, saisie, soumission…).
* Exemple : envoyer un message quand on clique sur **Send**.\
  👉 C’est l’utilisateur qui déclenche le code.

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function handleSendClick() {
    sendMessage(message); // ✅ Seulement au clic
  }

  return (
    <>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>Send</button>
    </>
  );
}
```

***

### 🔹 Effects

* S’exécutent **quand une synchronisation est nécessaire**, indépendamment des actions de l’utilisateur.
* Exemple : se connecter automatiquement à la **room sélectionnée**.\
  👉 Ce n’est pas une interaction qui déclenche le code, mais un **état du composant** (ici `roomId`).

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();

    return () => connection.disconnect(); // Nettoyage
  }, [roomId]); // ✅ Dépend de la salle courante
}
```

***

### 🔹 En résumé

* **Event handler** → logique déclenchée **par une action utilisateur explicite**.
* **Effect** → logique déclenchée **par un état/reactive value** qui change (montage, props, state).

***

📌 Exemple combiné (comme ton code final) :

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // ✅ Synchronisation automatique avec la bonne room
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  // ✅ Interaction explicite de l’utilisateur
  function handleSendClick() {
    sendMessage(message);
  }

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>Send</button>
    </>
  );
}
```

## ⚡ Reactive values et Reactive logic

### 🔹 Qu’est-ce qu’une valeur réactive ?

* **Props**
* **State**
* **Variables déclarées dans le corps du composant** (dérivées de props ou state)

👉 Elles sont dites _réactives_ car elles **changent lors d’un re-render** et participent au flux de données React.

Exemple :

```jsx
const serverUrl = 'https://localhost:1234'; // ❌ pas réactif (fixe)

function ChatRoom({ roomId }) { // ✅ réactif (prop)
  const [message, setMessage] = useState(''); // ✅ réactif (state)

  // ...
}
```

***

### 🔹 Différence dans le comportement

#### 1. **Event Handlers → logique non réactive**

* S’exécutent **uniquement** quand l’utilisateur refait l’action.
* Peuvent lire des valeurs réactives, mais **ne se relancent pas automatiquement** si ces valeurs changent.

```jsx
function handleSendClick() {
  sendMessage(message); // message est lu au moment du clic
}
```

➡️ Même si `message` change dans un re-render, ce code **ne se rejoue pas** tout seul.

***

#### 2. **Effects → logique réactive**

* S’exécutent (et se ré-exécutent) **dès qu’une valeur réactive qu’ils lisent change**.
* Tu dois lister ces valeurs dans les **dépendances** (`[deps]`).

```jsx
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();

  return () => connection.disconnect();
}, [roomId]); // ✅ se relance si roomId change
```

➡️ Ici, si `roomId` change, React rejoue tout l’Effect (connect → disconnect → reconnect).

***

### 🔹 Exemple combiné

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // ✅ Logique réactive : se connecte quand roomId change
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  // ✅ Logique non réactive : se déclenche uniquement au clic
  function handleSendClick() {
    sendMessage(message);
  }

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>Send</button>
    </>
  );
}
```

***

✅ **Résumé clair :**

* Les **event handlers** lisent les valeurs réactives mais **ne “réagissent” pas** automatiquement → ils s’exécutent uniquement sur action utilisateur.
* Les **effects** sont **liés aux valeurs réactives** → ils se rejouent automatiquement quand ces valeurs changent.

## ⚡ Event handlers ≠ réactifs

👉 Dans ton exemple :

```jsx
// Mauvaise interprétation (si c’était réactif)
sendMessage(message);
```

* Si ce code était placé dans un **Effect**, il serait **ré-exécuté** à chaque fois que `message` change.
* Ça voudrait dire qu’à chaque frappe de l’utilisateur (dès qu’il tape une lettre), le message serait envoyé **automatiquement** → ce n’est pas le comportement voulu ! 😅

***

### ✅ Bonne approche → dans un event handler

```jsx
function handleSendClick() {
  sendMessage(message);
}
```

* Ici, `sendMessage(message)` **n’est pas réactif**.
* Il s’exécute **uniquement** quand l’utilisateur clique sur **"Send"**.
* Même si `message` change 100 fois dans le state (pendant la saisie), rien ne se passe tant que le bouton n’est pas cliqué.

***

### 📌 Conclusion

* **Logic inside event handlers is NOT reactive.**
* Un **event handler** se déclenche **seulement** lors de l’action utilisateur (clic, saisie, etc.), pas en réponse aux re-renders.
* Tu dois y mettre les actions qui **ne doivent pas être rejouées automatiquement**.

## 🎯 Extraire la logique non-réactive d’un `useEffect`

### 🚨 Le problème

Quand tu mélanges logique réactive et non-réactive dans un `useEffect`, tu risques d’introduire des re-synchronisations inutiles.\
Exemple :

```jsx
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.on("connected", () => {
    showNotification("Connected!", theme); // ⚡ dépend de theme
  });
  connection.connect();
  return () => connection.disconnect();
}, [roomId, theme]); // <- provoque une reconnexion à chaque changement de thème
```

👉 Ici, **`roomId` doit être une dépendance** (connexion au bon salon), mais **`theme` ne devrait pas déclencher une reconnexion**. Pourtant, il le fait à cause de sa présence dans le tableau des dépendances.

***

### ✅ La solution : séparer logique réactive et logique non-réactive

On utilise un **Effect Event** (`useEffectEvent`) pour extraire la partie non-réactive :

```jsx
import { useEffect } from "react";
import { createConnection } from "./chat.js";
import { showNotification } from "./notifications.js";

const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId, theme }) {
  // ⚡ Événement non-réactif : il lit toujours la dernière valeur de theme
  const onConnected = useEffectEvent(() => {
    showNotification("Connected!", theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on("connected", onConnected);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // 👈 Ne dépend plus de theme

  return <h1>Welcome to the {roomId} room!</h1>;
}
```

* La **connexion dépend uniquement de `roomId`**.
* La notification lit toujours le **dernier `theme`**, sans provoquer de reconnexion.

***

### 📌 Règle d’or

* **`useEffect`** → pour la logique **réactive** (synchronisation dépendante de props/state).
* **`useEffectEvent`** → pour la logique **non-réactive** (utiliser la dernière valeur sans déclencher de re-sync).

## ⚡ Déclarer un **Effect Event** (`useEffectEvent`)

### 🚨 Le problème

Un `useEffect` est **réactif** : si tu lis une valeur réactive (prop, state, variable interne) dans son corps, tu dois l’ajouter dans les dépendances.\
👉 Cela peut provoquer des **re-synchronisations inutiles** (par ex. se reconnecter au chat juste parce que le `theme` a changé).

***

### ✅ La solution : `useEffectEvent`

React introduit un **Hook expérimental** :

```jsx
import { experimental_useEffectEvent as useEffectEvent } from 'react';
```

Il permet d’extraire la **logique non-réactive** de ton `useEffect`.

Exemple :

```jsx
function ChatRoom({ roomId, theme }) {
  // ⚡ Effect Event : lit toujours la dernière valeur de theme
  const onConnected = useEffectEvent(() => {
    showNotification("Connected!", theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on("connected", () => {
      onConnected(); // 👈 appelé sans rendre theme réactif
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ dépend seulement de roomId
}
```

***

### 🔑 Points importants

* `useEffectEvent` **voit toujours la dernière valeur** des props/state (ici `theme`).
* Il n’est **pas réactif** → tu **ne dois pas** l’ajouter dans le tableau des dépendances.
* Il agit comme un **event handler interne à l’Effect**.

***

### 🎯 Règle d’or

* **Event handler classique (`onClick`, etc.)** → déclenché par une **interaction utilisateur**.
* **Effect Event (`useEffectEvent`)** → déclenché **depuis un `useEffect`** pour isoler une logique **non-réactive**.

👉 Cela permet de **“casser la chaîne de réactivité”** d’un `useEffect` et éviter des comportements indésirables (ex. reconnexion inutile).

## 📌 Lire les dernières props et states avec **Effect Events**

### 🚨 Le problème

Dans un `useEffect`, **toute valeur réactive** utilisée doit être déclarée comme dépendance.\
👉 Mais parfois, on veut **lire une valeur à jour** sans que l’Effect ne re-synchronise à chaque changement.

Exemple classique :

```jsx
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems);
  }, [url]); // ⚠️ Le linter demande aussi numberOfItems
}
```

Ici, `url` doit bien être une dépendance (chaque URL = une nouvelle visite).\
Mais `numberOfItems` n’a rien à voir avec l’événement de visite : si le panier change, cela ne doit pas re-déclencher l’Effect.

***

### ✅ La solution : séparer avec `useEffectEvent`

On déclare une **Effect Event** pour isoler la logique non-réactive :

```jsx
import { experimental_useEffectEvent as useEffectEvent } from "react";

function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  const onVisit = useEffectEvent((visitedUrl) => {
    logVisit(visitedUrl, numberOfItems); // ✅ lit toujours la valeur à jour
  });

  useEffect(() => {
    onVisit(url); // ✅ l’Effect ne dépend que de url
  }, [url]);
}
```

***

### 🔑 Ce qui se passe

* L’**Effect** reste **réactif à `url`** → loguer une visite quand l’URL change.
* L’**Effect Event** lit toujours les valeurs les plus récentes (`numberOfItems`) → pas besoin de dépendance.
* Résultat :
  * On loggue chaque **visite unique**
  * On inclut la valeur **actuelle** du panier au moment du log
  * Mais on ne relance pas l’Effect pour chaque ajout/retrait d’articles.

***

### ⚡ Pourquoi ne pas supprimer le linter ?

Tu pourrais être tenté de faire :

```jsx
useEffect(() => {
  logVisit(url, numberOfItems);
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, [url]);
```

👉 Mauvaise pratique !

* Tu "mens" à React en disant que l’Effect n’a pas besoin de `numberOfItems`.
* Cela mène à des bugs de valeurs périmées (**stale values**).
* Avec `useEffectEvent`, tu n’as plus besoin de tricher : le code reste clair et juste.

***

### 🎯 Résumé pratique

* **Ce qui doit déclencher une re-synchro** → reste dans le `useEffect` (ex. `url`).
* **Ce qui doit juste être lu à jour** → va dans une **Effect Event** (ex. `numberOfItems`).
* Ne **jamais désactiver le linter** → si ça bloque, c’est ton code qui doit être repensé.

## ⚠️ Limitations of Effect Events

#### 🔑 Règles principales

1. **Appelle un Effect Event uniquement à l’intérieur d’un `useEffect`.**\
   👉 Il n’est pas conçu pour être exécuté depuis un handler ou ailleurs dans le code.
2. **Ne le passe jamais à un autre composant ou Hook.**\
   👉 Contrairement à un event handler classique (`onClick`, etc.), un Effect Event n’est **pas une prop** réutilisable.

***

### 🚫 Mauvaise utilisation : passer un Effect Event

```jsx
function Timer() {
  const [count, setCount] = useState(0);

  const onTick = useEffectEvent(() => {
    setCount(count + 1);
  });

  useTimer(onTick, 1000); // ❌ Mauvais : on passe un Effect Event
  return <h1>{count}</h1>;
}

function useTimer(callback, delay) {
  useEffect(() => {
    const id = setInterval(() => {
      callback(); // ❌ callback est réactif ici → bug potentiel
    }, delay);
    return () => clearInterval(id);
  }, [delay, callback]);
}
```

👉 Ici `onTick` est traité comme un callback classique, mais ce n’est pas son rôle.\
Ça casse le modèle réactif et mène à des dépendances incorrectes.

***

### ✅ Bonne utilisation : déclarer l’Effect Event **dans le Hook**

```jsx
function Timer() {
  const [count, setCount] = useState(0);

  useTimer(() => {
    setCount(count + 1);
  }, 1000);

  return <h1>{count}</h1>;
}

function useTimer(callback, delay) {
  const onTick = useEffectEvent(() => {
    callback();
  });

  useEffect(() => {
    const id = setInterval(() => {
      onTick(); // ✅ Appel interne, contrôlé par l’Effect
    }, delay);
    return () => clearInterval(id);
  }, [delay]); // pas besoin d'ajouter onTick
}
```

👉 Ici, `onTick` est **déclaré à côté de l’Effect** qui l’utilise → pas de dépendance inutile, pas de bug.

***

### 📝 Récap visuel

| Concept                                   | Réactif ?                               | Où l’utiliser                                             | Peut être passé en prop ? |
| ----------------------------------------- | --------------------------------------- | --------------------------------------------------------- | ------------------------- |
| **Event handler** (`onClick`, `onSubmit`) | ❌ Non                                   | Réagit aux interactions utilisateur                       | ✅ Oui                     |
| **Effect** (`useEffect`)                  | ✅ Oui                                   | Synchronisation avec l’extérieur                          | ❌ Non                     |
| **Effect Event** (`useEffectEvent`)       | ❌ Non (lit toujours les valeurs à jour) | À l’intérieur d’un Effect pour isoler du code non-réactif | ❌ Non                     |

***

👉 Donc retiens :

* **Event handler** → interactions utilisateur.
* **Effect** → logique réactive liée aux props/state.
* **Effect Event** → morceaux non-réactifs dans un Effect.
