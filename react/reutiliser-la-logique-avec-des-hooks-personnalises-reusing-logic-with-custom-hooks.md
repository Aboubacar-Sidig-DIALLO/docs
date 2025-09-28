# 🔄 Réutiliser la logique avec des Hooks personnalisés(Reusing Logic with Custom Hooks)

### 📌 Introduction

React fournit déjà plusieurs Hooks intégrés comme `useState`, `useContext` ou `useEffect`.\
Cependant, il arrive souvent que tu souhaites avoir un Hook adapté à un besoin plus spécifique :

* récupérer des données (fetch)
* suivre si l’utilisateur est en ligne ou hors ligne
* se connecter à une salle de chat
* gérer une logique de formulaire, etc.

👉 Ces Hooks n’existent pas dans React par défaut, mais la bonne nouvelle est que tu peux créer tes **propres Hooks personnalisés** adaptés aux besoins de ton application.

***

### 🎯 Ce que tu vas apprendre

* ✅ Ce qu’est un **Hook personnalisé** et comment l’écrire.
* ✅ Comment **réutiliser la logique** entre plusieurs composants.
* ✅ Comment **nommer et structurer** tes Hooks.
* ✅ Quand et pourquoi il est pertinent d’extraire la logique dans un Hook.

***

### 💡 Définition

Un **Hook personnalisé** est simplement une **fonction JavaScript** qui :

1. commence par `use` (ex : `useChat`, `useFetch`, `useOnlineStatus`),
2. appelle d’autres Hooks React (`useState`, `useEffect`, etc.),
3. encapsule une logique réutilisable que tu peux partager entre composants.

***

### 📌 Exemple concret

Imaginons que tu veuilles gérer une connexion à une salle de chat.\
Sans Hook personnalisé, ton code risque d’être répété dans plusieurs composants.\
Avec un **Hook personnalisé**, tu encapsules cette logique dans une fonction :

```jsx
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";

export function useChatRoom(roomId) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();

    connection.on("message", (msg) => {
      setMessages((prev) => [...prev, msg]);
    });

    return () => connection.disconnect();
  }, [roomId]);

  return messages;
}
```

👉 Ensuite, dans n’importe quel composant, tu peux réutiliser cette logique simplement :

```jsx
function ChatRoom({ roomId }) {
  const messages = useChatRoom(roomId);

  return (
    <div>
      <h1>Chat room: {roomId}</h1>
      <ul>
        {messages.map((msg, i) => <li key={i}>{msg}</li>)}
      </ul>
    </div>
  );
}
```

***

### 📌 Bonnes pratiques pour créer un Hook

1. **Nommer toujours avec `use...`** → `useForm`, `useOnlineStatus`, etc.
2. **Retourner ce qui est utile** → des valeurs, des états ou des fonctions.
3. **Extraire uniquement la logique réutilisable** → si le code est propre à un composant unique, pas besoin de Hook.
4. **Éviter la sur-abstraction** → crée un Hook seulement si le code est vraiment utilisé à plusieurs endroits ou s’il améliore la lisibilité.

***

### 📌 Quand extraire un Hook personnalisé ?

* 🔄 Quand **plusieurs composants** partagent la même logique.
* 📦 Quand tu veux **isoler une logique complexe** pour simplifier un composant.
* 🎯 Quand tu veux **rendre ton code plus lisible et réutilisable**.

***

👉 En résumé :\
Les **Hooks personnalisés** te permettent d’encapsuler et de réutiliser la logique entre composants. Ils sont une **boîte à outils** que tu construis pour répondre aux besoins spécifiques de ton application.

## 🔄 Hooks personnalisés : partager de la logique entre composants

### 📌 Contexte

La plupart des applications dépendent fortement du réseau.\
Supposons que tu veuilles avertir l’utilisateur si sa connexion tombe pendant qu’il utilise ton app.\
Pour cela, il faut :

1. Un **état (`isOnline`)** qui garde la trace de l’état du réseau (connecté ou non).
2. Un **Effet (`useEffect`)** qui écoute les événements globaux `online` et `offline`, et met à jour l’état.

***

### ✅ Première implémentation : un composant `StatusBar`

On commence par un composant simple qui affiche si la connexion est active ou non :

```jsx
import { useState, useEffect } from "react";

export default function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);

  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }

    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);

    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);

  return <h1>{isOnline ? "✅ En ligne" : "❌ Hors ligne"}</h1>;
}
```

👉 Si tu désactives ta connexion Internet, l’UI passe de **En ligne** à **Hors ligne** automatiquement.

***

### ⚡ Réutilisation de la logique : un bouton `SaveButton`

Maintenant, on veut un bouton **Enregistrer** qui :

* reste cliquable quand on est en ligne,
* devient désactivé et affiche _“Reconnexion…”_ quand le réseau est coupé.

On pourrait copier-coller la même logique :

```jsx
import { useState, useEffect } from "react";

export default function SaveButton() {
  const [isOnline, setIsOnline] = useState(true);

  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }

    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);

    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);

  function handleSaveClick() {
    console.log("✅ Progression sauvegardée !");
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? "Sauvegarder" : "Reconnexion..."}
    </button>
  );
}
```

👉 Cela fonctionne, mais on a **dupliqué** toute la logique de connexion réseau.

***

### 🔧 Problème

Même si `StatusBar` et `SaveButton` affichent des choses différentes, ils partagent la **même logique** pour savoir si l’utilisateur est en ligne ou non.\
La duplication rend le code plus difficile à maintenir.

💡 **Solution : extraire cette logique dans un Hook personnalisé (`useOnlineStatus`).**

## 🛠️ Extraire sa propre logique dans un Hook personnalisé

### 📌 Problème

On avait deux composants (`StatusBar` et `SaveButton`) qui utilisaient la même logique pour savoir si l’utilisateur est en ligne ou hors ligne.\
Cette logique était **dupliquée** dans chacun d’eux → pas optimal.

***

### ✅ Solution : créer un Hook personnalisé `useOnlineStatus`

Imaginons qu’il existe un Hook intégré comme `useOnlineStatus`. On pourrait simplifier nos composants :

```jsx
function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? "✅ En ligne" : "❌ Hors ligne"}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log("✅ Progression sauvegardée");
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? "Sauvegarder" : "Reconnexion..."}
    </button>
  );
}
```

👉 Problème : ce Hook n’existe pas par défaut.\
Bonne nouvelle : on peut le **créer nous-mêmes** !

***

### ✨ Implémentation de `useOnlineStatus`

```jsx
import { useState, useEffect } from "react";

export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);

  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }

    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);

    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);

  return isOnline; // ✅ le Hook retourne juste l'état
}
```

***

### 🚀 Utilisation du Hook dans les composants

```jsx
import { useOnlineStatus } from "./useOnlineStatus.js";

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? "✅ En ligne" : "❌ Hors ligne"}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log("✅ Progression sauvegardée");
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? "Sauvegarder" : "Reconnexion..."}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
```

***

### 🎯 Avantages

* Plus de **duplication** → la logique est centralisée dans un Hook.
* Les composants expriment leur **intention** (_utiliser l’état de la connexion_) au lieu de détailler **comment** ça marche.
* Plus facile à **maintenir** et à **réutiliser** dans toute l’application.

## 🪝 Règles de nommage des Hooks personnalisés

### 📌 Rappel : composants vs Hooks

* **Composants React** → commencent toujours par une **majuscule** (ex. `StatusBar`, `SaveButton`) et **retournent du JSX** ou quelque chose que React peut afficher.
* **Hooks** (intégrés ou personnalisés) → commencent toujours par le préfixe **`use`** suivi d’une majuscule (ex. `useState`, `useEffect`, `useOnlineStatus`) et peuvent **retourner n’importe quelle valeur** (pas forcément du JSX).

👉 Cette convention permet de savoir **d’un coup d’œil** si une fonction utilise l’API des Hooks (donc doit respecter les règles des Hooks) ou si c’est juste une fonction utilitaire.

***

### ✅ Exemple correct

```jsx
// Un Hook personnalisé qui utilise d’autres Hooks
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  // ...
  return isOnline;
}

function StatusBar() {
  const isOnline = useOnlineStatus(); // 👌 clair : c’est un Hook
  return <h1>{isOnline ? "✅ En ligne" : "❌ Hors ligne"}</h1>;
}
```

***

### ⚠️ Exemple incorrect

Si on enlève le préfixe `use` :

```jsx
function getOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true); // 🔴 Erreur
  return isOnline;
}
```

👉 Le linter React **refusera** que `useState` ou `useEffect` soient utilisés ici, car `getOnlineStatus` n’est pas reconnu comme un Hook.

***

### 🎯 Bonne pratique :

* Si votre fonction **n’utilise pas** de Hooks → ce n’est pas un Hook, inutile de mettre `use`.
* Si elle **utilise** (ou utilisera à l’avenir) un ou plusieurs Hooks → son nom **doit** commencer par `use`.

***

### 📝 Exemple avec une fonction utilitaire

Mauvaise approche :

```jsx
// 🔴 Mauvais : ce n’est pas un vrai Hook car il n’appelle aucun Hook
function useSorted(items) {
  return items.slice().sort();
}
```

Bonne approche :

```jsx
// ✅ Correct : fonction utilitaire classique
function getSorted(items) {
  return items.slice().sort();
}

function List({ items, shouldSort }) {
  let displayedItems = items;
  if (shouldSort) {
    displayedItems = getSorted(items); // 👍 peut être appelé de manière conditionnelle
  }
  return <ul>{displayedItems.map(i => <li key={i}>{i}</li>)}</ul>;
}
```

***

### 📌 Cas où un Hook peut ne pas encore appeler d’autres Hooks

Il est parfois pertinent de préparer un Hook qui n’utilise pas encore de Hooks, mais qui en utilisera plus tard :

```jsx
function useAuth() {
  // TODO : remplacer par un vrai Hook quand l’auth sera prête
  // return useContext(Auth);
  return { user: "TEST_USER" };
}
```

👉 Ici, on garde `useAuth` car dans le futur, ce Hook utilisera probablement `useContext` ou `useState`.

***

✅ **Résumé :**

* Les Hooks **doivent commencer par `use`**.
* Les fonctions utilitaires classiques n’ont **pas** ce préfixe.
* Le linter React applique cette règle et évite les erreurs d’utilisation des Hooks.

## 🪝 Les Hooks personnalisés partagent la logique avec état, pas l’état lui-même

### ⚡ Rappel

Un Hook personnalisé ne permet pas de partager une **même variable d’état** entre plusieurs composants.\
👉 Il permet uniquement de **réutiliser la logique** qui gère cet état.

***

### 🔍 Exemple simple : suivi du statut en ligne

```jsx
function StatusBar() {
  const isOnline = useOnlineStatus(); // 👌
  // ...
}

function SaveButton() {
  const isOnline = useOnlineStatus(); // 👌
  // ...
}
```

➡️ Chaque composant appelle **son propre `useOnlineStatus()`**.\
Cela revient à écrire :

```jsx
function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    // logique connexion réseau…
  }, []);
}

function SaveButton() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    // logique connexion réseau…
  }, []);
}
```

👉 Résultat : ce sont **deux états distincts**.\
Ils ont la même valeur au même moment uniquement car ils sont **synchronisés avec la même source externe** (le statut réseau).

***

### 📝 Exemple Formulaire

```jsx
export default function Form() {
  const [firstName, setFirstName] = useState("Mary");
  const [lastName, setLastName] = useState("Poppins");

  return (
    <>
      <label>
        Prénom :
        <input value={firstName} onChange={e => setFirstName(e.target.value)} />
      </label>
      <label>
        Nom :
        <input value={lastName} onChange={e => setLastName(e.target.value)} />
      </label>
      <p><b>Bonjour, {firstName} {lastName}.</b></p>
    </>
  );
}
```

⚠️ Ici, il y a de la logique répétitive pour chaque champ (`useState`, gestionnaire `onChange`, binding JSX).

***

### ✨ Extraction dans un Hook personnalisé

```jsx
import { useState } from "react";

export function useFormInput(initialValue) {
  const [value, setValue] = useState(initialValue);

  function handleChange(e) {
    setValue(e.target.value);
  }

  return {
    value,
    onChange: handleChange
  };
}
```

Utilisation :

```jsx
function Form() {
  const firstNameProps = useFormInput("Mary");
  const lastNameProps = useFormInput("Poppins");

  return (
    <>
      <label>
        Prénom :
        <input {...firstNameProps} />
      </label>
      <label>
        Nom :
        <input {...lastNameProps} />
      </label>
      <p><b>Bonjour, {firstNameProps.value} {lastNameProps.value}.</b></p>
    </>
  );
}
```

***

### 🎯 Conclusion

* Chaque appel à un Hook (ex. `useFormInput`) **crée un état indépendant**.
* Les Hooks personnalisés servent à **factoriser la logique** (comment gérer l’état), mais **pas à partager un même état**.
* Si vous devez réellement partager un état **entre plusieurs composants**, il faut **remonter cet état dans un parent** et le passer via les props (ou utiliser un contexte).

## 🔄 Passer des valeurs réactives entre Hooks

### ⚡ Principe

* Le code d’un **Hook personnalisé** s’exécute à chaque re-rendu du composant.
* Comme les composants, les Hooks personnalisés doivent rester **purs**.
* Ils reçoivent donc toujours les **dernières valeurs de props et d’état**.

***

### 🔍 Exemple sans Hook personnalisé

```jsx
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";
import { showNotification } from "./notifications.js";

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useEffect(() => {
    const options = { serverUrl, roomId };
    const connection = createConnection(options);

    connection.on("message", (msg) => {
      showNotification("Nouveau message : " + msg);
    });

    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        URL du serveur :
        <input
          value={serverUrl}
          onChange={(e) => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Bienvenue dans le salon {roomId} !</h1>
    </>
  );
}
```

👉 Ici, l’**Effect** se re-synchronise quand `roomId` ou `serverUrl` changent (on le voit dans la console avec les connexions/déconnexions).

***

### ✨ Extraction en Hook personnalisé

On peut déplacer la logique de connexion dans un Hook réutilisable :

```jsx
import { useEffect } from "react";
import { createConnection } from "./chat.js";
import { showNotification } from "./notifications.js";

export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = { serverUrl, roomId };
    const connection = createConnection(options);

    connection.connect();
    connection.on("message", (msg) => {
      showNotification("Nouveau message : " + msg);
    });

    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

Et dans le composant :

```jsx
import { useState } from "react";
import { useChatRoom } from "./useChatRoom.js";

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useChatRoom({ roomId, serverUrl });

  return (
    <>
      <label>
        URL du serveur :
        <input
          value={serverUrl}
          onChange={(e) => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Bienvenue dans le salon {roomId} !</h1>
    </>
  );
}
```

***

### 🎯 Ce qu’il faut retenir

* Les Hooks personnalisés **reçoivent les dernières valeurs réactives** (props, state).
* Tu peux **chaîner** les Hooks : par ex. `useState` → fournit `serverUrl` → passé en entrée de `useChatRoom`.
* Résultat : dès que `roomId` ou `serverUrl` changent, le Hook personnalisé réagit automatiquement et reconnecte le chat.

## 🎛️ Passer des gestionnaires d’événements (event handlers) à des Hooks personnalisés

### ⚡ Problème initial

Ton Hook `useChatRoom` gère lui-même ce qui se passe lorsqu’un message est reçu :

```jsx
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = { serverUrl, roomId };
    const connection = createConnection(options);
    connection.connect();

    connection.on("message", (msg) => {
      showNotification("Nouveau message : " + msg);
    });

    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

👉 Ici, la logique (afficher une notification) est **codée en dur** dans le Hook.\
Si tu veux que chaque composant décide quoi faire avec un message, il faut **déplacer cette logique dans le composant**.

***

### 🛠️ Passer un gestionnaire depuis le composant

```jsx
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useChatRoom({
    roomId,
    serverUrl,
    onReceiveMessage(msg) {
      showNotification("Nouveau message : " + msg);
    }
  });

  return (
    <>
      <label>
        URL du serveur :
        <input
          value={serverUrl}
          onChange={(e) => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Bienvenue dans le salon {roomId} !</h1>
    </>
  );
}
```

👉 Désormais, c’est le **composant** qui décide quoi faire quand un message arrive.

***

### 📝 Adapter le Hook

```jsx
export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  useEffect(() => {
    const options = { serverUrl, roomId };
    const connection = createConnection(options);
    connection.connect();

    connection.on("message", (msg) => {
      onReceiveMessage(msg);
    });

    return () => connection.disconnect();
  }, [roomId, serverUrl, onReceiveMessage]); // ✅ Toutes les dépendances
}
```

⚠️ Problème : si `onReceiveMessage` change à chaque re-render (par ex. recréé par le parent), cela va **forcer une reconnexion du chat** à chaque fois.

***

### ✨ Solution avec `useEffectEvent`

En utilisant l’API expérimentale `useEffectEvent`, on peut **éviter de rendre `onReceiveMessage` réactif** :

```jsx
import { useEffect, useEffectEvent } from "react";

export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  const onMessage = useEffectEvent(onReceiveMessage);

  useEffect(() => {
    const options = { serverUrl, roomId };
    const connection = createConnection(options);
    connection.connect();

    connection.on("message", (msg) => {
      onMessage(msg);
    });

    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ Pas besoin d’ajouter onReceiveMessage
}
```

👉 Résultat :

* `roomId` et `serverUrl` restent des dépendances réactives.
* `onReceiveMessage` est encapsulé, donc pas de reconnexion inutile.
* Le composant peut passer **n’importe quelle logique personnalisée**.

***

### 🎯 À retenir

1. Un Hook personnalisé peut accepter des **handlers** pour laisser les composants définir le comportement.
2. Si un handler change souvent, il faut éviter de le mettre comme dépendance directe.
3. `useEffectEvent` (API expérimentale) permet de **séparer la partie réactive** (connexion au chat) de la **partie non réactive** (gestion des messages).
4. Le Hook reste **réutilisable** et **flexible** : chaque composant peut décider quoi faire lorsqu’un événement survient.

## 🧩 Quand utiliser des Hooks personnalisés

### ⚡ Pas besoin pour chaque petit bout de code

Tu n’as pas besoin d’extraire un Hook personnalisé pour chaque minuscule duplication.\
👉 Un peu de duplication est parfois préférable à une abstraction inutile.

Par exemple, créer un `useFormInput` juste pour encapsuler un simple `useState` est probablement excessif.

***

### ✅ Quand réfléchir à un Hook personnalisé ?

Chaque fois que tu écris un **Effect (`useEffect`)**, demande-toi :\
➡️ Est-ce que ce serait plus clair de le mettre dans un Hook personnalisé ?

Tu n’utilises pas les Effects si souvent. Donc, quand tu en écris un, c’est probablement pour :

* **synchroniser avec un système externe** (API, WebSocket, localStorage, navigateur…),
* ou **faire quelque chose que React n’a pas prévu en standard**.

Mettre ça dans un Hook personnalisé rend ton intention explicite et évite que d’autres développeurs fassent des erreurs dans la gestion des dépendances.

***

### 📦 Exemple : `ShippingForm`

Tu veux gérer deux listes dépendantes :

* la liste des **villes** d’un pays,
* la liste des **zones** d’une ville.

Version de base (avec deux Effects séparés, ce qui est correct ✅) :

```jsx
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(res => res.json())
      .then(json => { if (!ignore) setCities(json); });
    return () => { ignore = true; };
  }, [country]);

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(res => res.json())
        .then(json => { if (!ignore) setAreas(json); });
      return () => { ignore = true; };
    }
  }, [city]);

  // ...
}
```

👉 Ici, tu fais **deux synchronisations différentes**, donc c’est bien de garder **deux Effects séparés**.

***

### 🎯 Extraire un Hook générique `useData`

On voit de la duplication (fetch, ignore, cleanup).\
On peut l’extraire dans un Hook réutilisable :

```jsx
function useData(url) {
  const [data, setData] = useState(null);

  useEffect(() => {
    if (url) {
      let ignore = false;
      fetch(url)
        .then(res => res.json())
        .then(json => { if (!ignore) setData(json); });
      return () => { ignore = true; };
    }
  }, [url]);

  return data;
}
```

Puis simplifier `ShippingForm` :

```jsx
function ShippingForm({ country }) {
  const cities = useData(`/api/cities?country=${country}`);
  const [city, setCity] = useState(null);
  const areas = useData(city ? `/api/areas?city=${city}` : null);

  // ...
}
```

👉 Ici, la logique est claire : tu donnes une `url` ➡️ tu récupères des `data`.

***

### 🔎 Bonnes pratiques pour créer un Hook personnalisé

#### ✅ Nom clair et lié à l’intention

* `useData(url)` → explicite et générique
* `useChatRoom(options)` → spécifique à un système externe (chat)
* `useImpressionLog(eventName, extraData)` → clair pour les analytics

#### ❌ Éviter les Hooks “lifecycle” abstraits

Exemples à éviter :

* `useMount(fn)`
* `useEffectOnce(fn)`
* `useUpdateEffect(fn)`

⚠️ Ces Hooks masquent les dépendances réelles et cassent la logique réactive de React.\
👉 Le linter ne peut plus t’avertir correctement des dépendances manquantes.

***

### ✨ Bon workflow recommandé

1. Commence toujours avec **des Effects bruts** :

```jsx
useEffect(() => {
  const connection = createConnection({ serverUrl, roomId });
  connection.connect();
  return () => connection.disconnect();
}, [serverUrl, roomId]);

useEffect(() => {
  post("/analytics/event", { eventName: "visit_chat", roomId });
}, [roomId]);
```

2. Si tu vois une logique récurrente ou très claire ➡️ **extrait-la en Hook personnalisé** :

```jsx
useChatRoom({ serverUrl, roomId });
useImpressionLog("visit_chat", { roomId });
```

👉 Résultat : le code est plus **déclaratif**, facile à comprendre, et chaque Hook a une **responsabilité claire**.

***

### 🚀 À retenir

* Les Hooks personnalisés servent à **partager de la logique avec état** (stateful logic), pas du state.
* Crée un Hook si :
  * tu manipules un système externe (API, WebSocket, EventListener, etc.),
  * ou si ton Effect devient réutilisable ailleurs.
* Donne toujours un **nom explicite** → reflète ton intention, pas l’implémentation.
* ❌ Évite les Hooks abstraits type `useMount`.
* ✅ Concentre-toi sur des Hooks **hauts-niveaux et concrets**.

## 🚀 Custom Hooks : t’aider à migrer vers de meilleurs patterns

### ⚡ Les Effects = “échappatoire”

* Tu utilises un `useEffect` quand tu dois **sortir du monde React** (écouter des events du navigateur, synchroniser avec une API externe, etc.).
* Mais avec le temps, l’équipe React essaie de **réduire les besoins d’Effects bruts** en proposant des solutions mieux adaptées (`useSyncExternalStore`, `use`, etc.).

***

### 🎯 Exemple : `useOnlineStatus`

#### Version initiale avec `useEffect`

```jsx
import { useState, useEffect } from 'react';

export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);

  useEffect(() => {
    function handleOnline() { setIsOnline(true); }
    function handleOffline() { setIsOnline(false); }

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}
```

✅ Ça marche, mais il y a des **limites** :

* suppose que l’app démarre en ligne (`true`), même si tu étais déjà offline,
* ne gère pas correctement le rendu côté serveur (SSR),
* beaucoup de logique “bas niveau” dans ton Hook.

***

#### Version améliorée avec `useSyncExternalStore`

React propose une API dédiée pour ce type de problème :

```jsx
import { useSyncExternalStore } from 'react';

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

export function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe,
    () => navigator.onLine, // valeur côté client
    () => true              // valeur côté serveur (par défaut "true")
  );
}
```

✨ Avantages :

* gère mieux les edge cases (offline au démarrage, SSR),
* reste réactif aux événements navigateur,
* plus robuste et idiomatique.

***

### 🔄 Migration sans douleur

Les composants n’ont **rien à changer** :

```jsx
function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();
  return (
    <button disabled={!isOnline}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}
```

👉 Que ton Hook soit basé sur `useEffect` ou `useSyncExternalStore`, le code composant reste identique.

***

### 🎁 Pourquoi c’est puissant

1. **Abstraction claire** : les composants expriment l’intention (`useOnlineStatus`) sans gérer les détails techniques.
2. **Évolutivité** : quand React sort une meilleure API, tu n’as qu’à changer ton Hook, pas tous tes composants.
3. **Code propre** : moins de duplication, logique centralisée.

***

### 🔮 Vers le futur : `use` et le data fetching

L’équipe React prévoit d’autres APIs, par exemple pour le **data fetching** :

```jsx
import { use } from 'react';

function ShippingForm({ country }) {
  const cities = use(fetch(`/api/cities?country=${country}`));
  const [city, setCity] = useState(null);
  const areas = city ? use(fetch(`/api/areas?city=${city}`)) : null;
  // ...
}
```

👉 Si tu avais déjà encapsulé ton fetch dans un Hook (`useData`), tu migreras facilement.\
👉 Si tu avais éparpillé du `useEffect` partout, la migration sera plus pénible.

***

✅ **En résumé**

* Extraire la logique dans des Hooks personnalisés n’est pas seulement pour **réutiliser du code**.
* C’est surtout une **assurance évolutive** : tu écris ton app avec les APIs d’aujourd’hui, et tu pourras la migrer vers les APIs de demain sans casser tes composants.

## 🎭 Il y a plusieurs façons de le faire (There is more than one way to do it)

Prenons l’exemple d’une animation **fade-in** :

***

### 1. 🔧 Avec un `useEffect` brut

Tu écris toute la logique directement dans l’Effet, en utilisant `requestAnimationFrame`.

👉 C’est simple pour un prototype, mais vite lourd et difficile à relire si ton composant grossit.

***

### 2. 📦 Extraction dans un **Hook personnalisé** `useFadeIn`

Tu crées un Hook qui encapsule la logique d’animation :

```jsx
useFadeIn(ref, 1000);
```

👉 Ton composant reste **lisible** et décrit ton intention (« appliquer un fade-in ») plutôt que les détails techniques.

***

### 3. 🔄 Découpage plus fin avec un Hook générique `useAnimationLoop`

Tu sépares encore plus :

* `useFadeIn` ne fait que calculer la progression et appliquer l’opacité.
* `useAnimationLoop` gère la boucle `requestAnimationFrame`.

👉 Utile si tu veux réutiliser la boucle d’animation pour autre chose (rotation, scaling, etc.).\
👉 Mais si c’est trop tôt, cela complexifie inutilement.

***

### 4. 🏗️ Externaliser totalement dans une **classe JavaScript**

Tu crées par exemple `FadeInAnimation`, et ton Hook se contente de démarrer/arrêter l’animation :

```jsx
useEffect(() => {
  const animation = new FadeInAnimation(ref.current);
  animation.start(duration);
  return () => animation.stop();
}, [ref, duration]);
```

👉 Bonne approche si tu veux un **moteur d’animations centralisé** ou chaîner plusieurs animations.\
👉 Mais ça déplace la logique hors de React (elle devient un « système externe »).

***

### 5. 🎨 Utiliser… du **CSS natif**

Ici, tu n’as même pas besoin de Hook. Une simple animation CSS fait le travail :

```css
.welcome {
  animation: fadeIn 1000ms;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
```

👉 C’est **plus simple, plus performant et plus maintenable** si le besoin est basique.\
👉 Pas de logique JS inutile.

***

## 📝 Récap

* Les **Hooks personnalisés** permettent de **partager de la logique**, pas l’état.
* Tu peux choisir ton **niveau d’abstraction** :
  * brut (`useEffect`),
  * spécialisé (`useFadeIn`),
  * générique (`useAnimationLoop`),
  * externe (`FadeInAnimation`),
  * ou natif (CSS).
* Plus tu factorises, plus tu rends ton code **réutilisable et déclaratif**, mais parfois la solution la plus simple suffit.
