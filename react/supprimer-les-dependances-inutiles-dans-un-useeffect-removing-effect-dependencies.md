# 📌 Supprimer les dépendances inutiles dans un useEffect(Removing Effect Dependencies)

#### 🚨 Pourquoi c’est important ?

Quand tu écris un `useEffect`, **le linter React** (`react-hooks/exhaustive-deps`) vérifie que tu as bien inclus toutes les valeurs réactives (props, state, variables calculées dans le composant) utilisées dans ton effet.

* ✅ Cela garantit que ton effet reste **synchronisé avec l’état et les props les plus récents**.
* ❌ Mais si tu ajoutes trop de dépendances (ou des mauvaises), ton effet :
  * peut **s’exécuter trop souvent**,
  * ou même provoquer une **boucle infinie** 🔄.

***

### 🔑 Ce que tu vas apprendre

1. 🌀 Comment corriger une **boucle infinie** de dépendances.
2. ✂️ Quoi faire si tu veux **supprimer une dépendance**.
3. 👀 Comment **lire une valeur dans un effet sans réagir** à ses changements.
4. 🧩 Pourquoi éviter d’avoir des **objets et fonctions** dans les dépendances.
5. 🚫 Pourquoi il est dangereux de **désactiver le linter** et quoi faire à la place.

***

### 🧠 Idée clé

👉 Tu ne peux pas _choisir arbitrairement_ tes dépendances.

* Si une valeur est **réactive** (prop, state, ou variable calculée dans le corps du composant), elle doit être listée.
* Si tu **ne veux pas** qu’elle déclenche de re-synchronisation, tu dois **repenser ton code** :
  * la sortir du composant,
  * la stocker dans un `useRef`,
  * ou utiliser un **Effect Event** (nouvelle API expérimentale).

***

📌 Dans les prochaines sous-sections, on va voir **des cas concrets** :

* les boucles infinies,
* la suppression correcte d’une dépendance,
* la lecture de valeurs sans réactivité,
* et la gestion des objets/fonctions.

## 📌 Les dépendances d’un `useEffect` doivent correspondre au code

#### 🔑 Principe

Quand tu écris un **Effet (`useEffect`)**, tu définis deux choses :

1. **Comment démarrer la synchronisation** (ex. se connecter à un serveur).
2. **Comment arrêter la synchronisation** (ex. se déconnecter).

Exemple :

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // 🚨 Erreur ici
}
```

Ici, tu as laissé le tableau de dépendances vide (`[]`).\
Le **linter React** (`react-hooks/exhaustive-deps`) détecte alors un problème :

```
React Hook useEffect has a missing dependency: 'roomId'.
Either include it or remove the dependency array.
```

***

#### ✅ La solution

Ton `useEffect` **lit la valeur `roomId`**.\
Comme `roomId` est **une valeur réactive** (elle peut changer via un re-render), tu dois l’indiquer dans la liste des dépendances :

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Dépendance correctement déclarée
}
```

***

#### 🚀 Résultat

* Si `roomId` change (ex. l’utilisateur choisit une autre salle dans un `<select>`),\
  👉 ton `useEffect` est relancé automatiquement.
* React **déconnecte l’ancienne salle** puis **se connecte à la nouvelle**.

Console de logs :

```
✅ Connecting to "general" room at https://localhost:1234...
❌ Disconnected from "general" room at https://localhost:1234
✅ Connecting to "travel" room at https://localhost:1234...
```

***

#### 🧠 À retenir

* Les **Effets “réagissent” aux valeurs réactives** (props, state, variables locales calculées).
* Si ton code utilise une valeur réactive, **elle doit apparaître dans le tableau de dépendances**.
* Le linter est ton allié : il t’indique **exactement** quelles dépendances ajouter pour éviter des bugs de synchronisation.

## 📌 Supprimer une dépendance : prouver qu’elle n’en est pas une

#### 🔑 Principe

👉 Tu ne peux pas “choisir” arbitrairement les dépendances d’un `useEffect`.\
**Chaque valeur réactive utilisée dans le code de l’Effet doit être déclarée comme dépendance.**

* Les **valeurs réactives** incluent :
  * les **props**,
  * le **state**,
  * et toutes les **variables/fonctions définies dans le corps du composant**.

Exemple :

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) { // roomId est une valeur réactive (prop)
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // l’Effet lit roomId
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ roomId doit être une dépendance
}
```

***

#### 🚨 Si tu tentes de l’enlever

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // 🔴 Erreur : dépendance manquante (roomId)
}
```

👉 Le **linter a raison** : si `roomId` change, ton composant restera connecté à la mauvaise salle.

***

#### ✅ Comment supprimer une dépendance correctement

La seule façon d’enlever une dépendance est de **prouver qu’elle n’est pas réactive**.\
Exemple : tu peux déplacer `roomId` en dehors du composant pour montrer qu’elle **ne changera jamais lors d’un re-render**.

```jsx
const serverUrl = 'https://localhost:1234';
const roomId = 'music'; // ⚡ Valeur fixe, non réactive

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ Plus besoin de roomId dans les dépendances
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

***

#### 🚀 Résultat

Puisque `roomId` est **constant**, ton `useEffect` :

* s’exécute uniquement **au montage** du composant,
* et se nettoie uniquement **au démontage**.

Console :

```
✅ Connecting to "music" room at https://localhost:1234...
❌ Disconnected from "music" room at https://localhost:1234
✅ Connecting to "music" room at https://localhost:1234...
```

***

#### 🧠 À retenir

* Si une valeur est **réactive**, elle doit être listée comme dépendance.
* Pour supprimer une dépendance, rends cette valeur **non réactive** (ex. la sortir du composant ou la définir à l’intérieur de l’Effet).
* Un `[]` vide est valide **uniquement si ton Effet ne dépend d’aucune valeur réactive**.

## 🔄 Modifier les dépendances en modifiant le code

#### 📌 Le workflow habituel avec un `useEffect`

1. Tu écris ton Effet et utilises des valeurs réactives (props, state, variables locales).
2. Le **linter** vérifie si toutes ces valeurs apparaissent dans la liste de dépendances.
3. Si tu n’aimes pas la liste proposée → **tu ne modifies pas directement les dépendances**.\
   👉 Tu modifies d’abord **le code autour de l’Effet**, puis tu laisses le linter recalculer les dépendances correctes.

⚡ En résumé : **tu ne choisis pas tes dépendances, ton code les détermine**.\
La liste des dépendances est comme une _photo_ des valeurs réactives que l’Effet utilise.\
Pour changer la liste, il faut changer la logique du code.

***

#### ⚠️ Erreur fréquente : supprimer le linter

Beaucoup de développeurs utilisent ce “hack” :

```jsx
useEffect(() => {
  // ...
  // 🔴 Mauvais : on ment à React
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, []);
```

👉 Ici, tu dis à React :\
« Mon Effet ne dépend de rien. »\
Mais en réalité, il dépend de valeurs réactives… Résultat : **bugs cachés garantis**.

***

#### 💥 Exemple d’un bug causé par la suppression du linter

```jsx
export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  function onTick() {
    setCount(count + increment);
  }

  useEffect(() => {
    const id = setInterval(onTick, 1000);
    return () => clearInterval(id);
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []); // ⚠️ On ignore le linter
}
```

**Ce qui devait se passer :**

* Le compteur s’incrémente chaque seconde du pas (`increment`) choisi avec les boutons.

**Ce qui se passe réellement :**

* `onTick` est celui de la première exécution → il “voit” toujours `count = 0` et `increment = 1`.
* Donc, il fera à jamais `setCount(0 + 1)`.
* Résultat : le compteur reste bloqué à **1**.

***

#### ✅ La bonne solution

On ne triche pas avec le linter.\
Il faut **ajouter la vraie dépendance** (`onTick`) à la liste :

```jsx
useEffect(() => {
  const id = setInterval(onTick, 1000);
  return () => clearInterval(id);
}, [onTick]); // ✅ onTick est bien une dépendance
```

Et si on veut éviter que l’Effet soit relancé trop souvent, on peut transformer `onTick` en **Effect Event** (non réactif, mais toujours à jour).

***

#### 🧠 Règle d’or

* ⚠️ **Ne jamais supprimer ou ignorer le linter**.
* ✅ Considère un avertissement du linter comme une **erreur de compilation**.
* ✨ Il existe toujours une meilleure solution (réécrire le code, extraire un Effect Event, déplacer une valeur hors du composant, etc.).

## 🧹 Supprimer les dépendances inutiles

Quand tu ajustes les dépendances d’un `useEffect`, demande-toi toujours :\
👉 _Est-ce logique que l’Effet se relance si **telle dépendance** change ?_

Dans certains cas, la réponse est **non** :

* tu veux que certaines parties du code ne se déclenchent que dans un contexte précis (pas à chaque re-rendu),
* tu veux seulement lire la **dernière valeur** d’une dépendance sans “réagir” à ses changements,
* ou bien ta dépendance change trop souvent parce que c’est un objet/fonction recréé à chaque rendu.

➡️ La bonne pratique consiste à vérifier d’abord si ton code doit vraiment être dans un `useEffect`.

***

### ❌ Mauvaise approche : logique d’événement dans un Effet

Exemple : un formulaire.\
Lorsqu’on soumet le formulaire (`submitted = true`), on envoie une requête POST et on affiche une notification.

Beaucoup écrivent ça dans un Effet :

```jsx
function Form() {
  const [submitted, setSubmitted] = useState(false);

  useEffect(() => {
    if (submitted) {
      // 🔴 Mauvais : logique spécifique à un événement dans un Effet
      post('/api/register');
      showNotification('Inscription réussie !');
    }
  }, [submitted]);

  function handleSubmit() {
    setSubmitted(true);
  }

  // ...
}
```

Au départ, ça semble fonctionner…\
Mais si plus tard tu ajoutes un **thème réactif** (`theme`), tu dois l’ajouter aux dépendances :

```jsx
function Form() {
  const [submitted, setSubmitted] = useState(false);
  const theme = useContext(ThemeContext);

  useEffect(() => {
    if (submitted) {
      // 🔴 Mauvais : déclenche encore la notif si le thème change !
      post('/api/register');
      showNotification('Inscription réussie !', theme);
    }
  }, [submitted, theme]);

  function handleSubmit() {
    setSubmitted(true);
  }

  // ...
}
```

👉 Problème : si tu soumets **une seule fois**, puis tu passes du mode clair au mode sombre,\
l’Effet va se rejouer… et réafficher une **nouvelle notification** alors que tu n’as rien fait.

***

### ✅ Bonne approche : mettre la logique dans un event handler

La soumission d’un formulaire est une **interaction précise** (un clic sur “Envoyer”).\
Donc la logique doit rester **dans le handler**, pas dans un `useEffect`.

```jsx
function Form() {
  const theme = useContext(ThemeContext);

  function handleSubmit() {
    // ✅ Bon : logique spécifique à l’événement dans le handler
    post('/api/register');
    showNotification('Inscription réussie !', theme);
  }

  return (
    <form onSubmit={(e) => { e.preventDefault(); handleSubmit(); }}>
      {/* ... champs ... */}
      <button type="submit">Envoyer</button>
    </form>
  );
}
```

Ici :

* L’appel à `post` et `showNotification` n’est **pas réactif**.
* Il s’exécute **uniquement** quand l’utilisateur déclenche l’action.

***

### 📌 Règle clé

* 🎯 **Si une logique répond à une interaction utilisateur → event handler** (`onClick`, `onSubmit`, etc.).
* 🔄 **Si une logique doit rester synchronisée avec des valeurs réactives → useEffect**.

## ⚖️ Ton Effet fait-il plusieurs choses sans rapport ?

Une question clé à se poser :\
👉 _Est-ce que mon `useEffect` essaie de synchroniser plusieurs processus différents en même temps ?_

***

### ❌ Exemple : un seul Effet pour deux synchronisations

Imaginons un formulaire d’expédition où l’utilisateur doit :

1. choisir un **pays** → récupérer les **villes** correspondantes
2. choisir une **ville** → récupérer les **zones** (areas) correspondantes

Tu pourrais être tenté de tout mettre dans **un seul Effet** :

```jsx
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);

  useEffect(() => {
    let ignore = false;

    // Synchronisation 1 : récupérer les villes
    fetch(`/api/cities?country=${country}`)
      .then(res => res.json())
      .then(json => { if (!ignore) setCities(json); });

    // Synchronisation 2 : récupérer les zones
    if (city) {
      fetch(`/api/areas?city=${city}`)
        .then(res => res.json())
        .then(json => { if (!ignore) setAreas(json); });
    }

    return () => { ignore = true; };
  }, [country, city]); // 🔴 Dépendances mélangées
}
```

👉 Problème :

* Tu veux que `fetchCities` dépende seulement du **pays**.
* Tu veux que `fetchAreas` dépende seulement de la **ville**.\
  Mais comme les deux sont dans le même Effet, changer la ville (`city`) relance aussi la requête pour les villes → requête inutile.

***

### ✅ Bonne pratique : séparer en deux Effets indépendants

Chaque Effet doit représenter **un seul processus de synchronisation indépendant**.

```jsx
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);

  // Effet 1 : synchroniser les villes selon le pays
  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(res => res.json())
      .then(json => { if (!ignore) setCities(json); });
    return () => { ignore = true; };
  }, [country]); // ✅ Dépend uniquement du pays

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);

  // Effet 2 : synchroniser les zones selon la ville
  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(res => res.json())
        .then(json => { if (!ignore) setAreas(json); });
      return () => { ignore = true; };
    }
  }, [city]); // ✅ Dépend uniquement de la ville
}
```

👉 Avantages :

* Si **le pays change**, seules les villes sont rechargées.
* Si **la ville change**, seules les zones sont rechargées.
* Les deux processus sont séparés et ne s’interfèrent pas.

***

### 📌 Règle d’or

* **Un Effet = un processus de synchronisation indépendant.**
* Si tu peux supprimer un Effet sans casser l’autre, c’est qu’ils sont bien séparés.
* Si tu trouves le code long, tu peux extraire la logique répétée dans un **Hook personnalisé** (ex. `useFetchData`).

## 📩 Lire un état pour calculer le prochain état

Imaginons ton composant de chat :

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();

    connection.on('message', (receivedMessage) => {
      // Ajoute le nouveau message à la liste
      setMessages([...messages, receivedMessage]);
    });

    return () => connection.disconnect();
  }, [roomId, messages]); // 👈 messages ajouté en dépendance
}
```

***

### ⚠️ Le problème

Ici, tu lis `messages` dans ton Effet.\
Donc logiquement, le linter t’oblige à l’ajouter dans la liste des dépendances (`[roomId, messages]`).

👉 Mais conséquence :

* Chaque fois que tu ajoutes un message → `setMessages` déclenche un nouveau rendu.
* Comme `messages` change → l’Effet se ré-exécute.
* Et donc la **connexion se recrée à chaque message**. 💥

Résultat : ton chat se reconnecte en boucle → expérience horrible pour l’utilisateur.

***

### ✅ La bonne solution : passer un **updater function**

Au lieu de **lire directement `messages` dans l’Effet**, tu donnes à React une fonction qui sait calculer le prochain état à partir du précédent.

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();

    connection.on('message', (receivedMessage) => {
      // Utilise la version fonctionnelle
      setMessages(msgs => [...msgs, receivedMessage]);
    });

    return () => connection.disconnect();
  }, [roomId]); // ✅ Plus besoin de mettre `messages`
}
```

***

### 🔍 Pourquoi ça marche ?

* `setMessages` accepte **deux formes** :
  1. `setMessages(nouvelleValeur)` → écrase l’état.
  2. `setMessages((prev) => nouvelleValeur)` → calcule à partir de l’ancien état.
*   Ici, on utilise la **forme fonctionnelle** :

    ```js
    setMessages(msgs => [...msgs, receivedMessage])
    ```

    où `msgs` est toujours la version la plus à jour de l’état, donnée par React.

👉 Donc plus besoin de dépendre de `messages` dans l’Effet, et tu évites les re-créations de connexion.

***

### 📌 Règle d’or

Quand tu veux **mettre à jour un état en fonction de sa valeur actuelle**,\
👉 utilise **toujours la fonction de mise à jour** au lieu de lire directement l’état.

Cela évite :

* les dépendances inutiles,
* les reconnections,
* les boucles infinies.

## 📌 Lire une valeur sans “réagir” à ses changements

Imaginons ton composant de chat :

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
      if (!isMuted) {
        playSound();
      }
    });
    return () => connection.disconnect();
  }, [roomId, isMuted]); // ✅ Toutes les dépendances déclarées
}
```

***

### ⚠️ Le problème

* Ici, tu lis `isMuted` dans ton `useEffect`.
* Donc tu es obligé de l’ajouter aux dépendances (`[roomId, isMuted]`).
* Résultat : **à chaque fois que tu changes le mute/unmute → l’Effet se re-synchronise → le chat se reconnecte**.
* Expérience utilisateur catastrophique 😬.

👉 Et si tu supprimais `isMuted` de la liste, ce serait pire : la valeur resterait “bloquée” (stale), car React ne saurait pas la mettre à jour.

***

### ✅ La solution : **Effect Event**

Tu peux extraire la partie **non-réactive** (qui lit `isMuted`) en utilisant `useEffectEvent`.

```jsx
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  const onMessage = useEffectEvent((receivedMessage) => {
    setMessages(msgs => [...msgs, receivedMessage]);
    if (!isMuted) {
      playSound();
    }
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ Ne dépend plus de `isMuted`
}
```

***

### 🔍 Pourquoi ça marche ?

* La **partie réactive** (la connexion) dépend de `roomId` → logique, si tu changes de salle, tu veux te reconnecter.
* La **partie non-réactive** (jouer un son en fonction de `isMuted`) est déplacée dans un **Effect Event** → elle voit toujours la valeur actuelle de `isMuted`, mais **sans obliger l’Effet à se re-synchroniser**.

👉 Résultat :

* Tu peux toggler mute/unmute sans reconnecter le chat.
* Tu joues le son uniquement quand c’est nécessaire.
* Pas besoin de tricher avec le linter.

***

### 📝 Règle d’or

* Tout ce qui doit **synchroniser un état externe** (API, socket, DOM, etc.) reste dans un `useEffect`.
* Tout ce qui doit juste **réagir localement aux dernières valeurs** (comme lire `isMuted`) peut aller dans un **Effect Event**.

## 📌 Envelopper un gestionnaire d’événement venant des props

Imaginons ce composant :

```jsx
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onReceiveMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId, onReceiveMessage]); // ✅ Toutes les dépendances déclarées
}
```

***

### ⚠️ Le problème

Si le parent fait ça :

```jsx
<ChatRoom
  roomId={roomId}
  onReceiveMessage={(receivedMessage) => {
    // ...
  }}
/>
```

➡️ Ici, **une nouvelle fonction** `onReceiveMessage` est créée à chaque re-render du parent.\
➡️ Donc ton `useEffect` considère que `onReceiveMessage` a changé → il se re-synchronise → ton chat se **reconnecte à chaque re-render du parent**, même si `roomId` n’a pas changé.

Pas du tout ce que tu veux.

***

### ✅ La solution : **Effect Event**

Au lieu de rendre l’Effet dépendant de `onReceiveMessage`, tu enveloppes son appel dans un **Effect Event** :

```jsx
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  const onMessage = useEffectEvent((receivedMessage) => {
    onReceiveMessage(receivedMessage);
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ Plus besoin d’ajouter onReceiveMessage
}
```

***

### 🔍 Pourquoi ça marche ?

* `onMessage` est un **Effect Event** → il voit toujours la version la plus récente de `onReceiveMessage`.
* Tu n’as plus besoin de mettre `onReceiveMessage` dans les dépendances de ton `useEffect`.
* Résultat : ton chat ne se reconnecte **que** si `roomId` change (ce qui est logique).
* Si le parent passe une nouvelle fonction à chaque render → aucun souci, l’Effet ne se ré-exécute pas inutilement.

***

### 📝 À retenir

* Les **props qui sont des fonctions** sont réactives si elles sont déclarées dans le corps du parent.
* Si tu les utilises dans un `useEffect`, tu risques des re-renders et des re-synchronisations inutiles.
* Solution : **les encapsuler dans un Effect Event** pour qu’elles soient lues avec leur valeur la plus récente **sans être une dépendance**.

## 📌 Séparer le code réactif et non-réactif

Prenons ton exemple :

```jsx
function Chat({ roomId, notificationCount }) {
  const onVisit = useEffectEvent((visitedRoomId) => {
    logVisit(visitedRoomId, notificationCount);
  });

  useEffect(() => {
    onVisit(roomId);
  }, [roomId]); // ✅ Toutes les dépendances déclarées
}
```

***

### ⚠️ Le problème à éviter

* Tu veux **loguer une visite quand roomId change** → donc `roomId` doit être une dépendance.
* Tu veux aussi inclure `notificationCount` dans le log.
* MAIS : si tu mets `notificationCount` dans les dépendances → chaque fois que le compteur change, un nouveau log est déclenché. **Pas logique** → ça créerait de faux “visits”.

***

### ✅ La solution : **Effect Event**

* Tu gardes le code **réactif** (lié à `roomId`) dans ton `useEffect`.
* Tu mets le code **non-réactif** (lié à `notificationCount`) dans un **Effect Event**.

👉 Résultat :

* L’effet se déclenche uniquement quand `roomId` change.
* Mais le log utilise toujours la **valeur la plus récente** de `notificationCount`.

***

### 📝 À retenir

* **Code réactif** → doit être dans le `useEffect`, dépendances incluses.
* **Code non-réactif** (qui doit juste lire la dernière valeur mais ne pas déclencher de re-synchro) → doit aller dans un **Effect Event**.
* Cette séparation permet d’éviter les dépendances inutiles qui déclencheraient des re-exécutions non désirées.

## ⚠️ Quand une valeur réactive change « par erreur » : objets et fonctions

### 🚨 Le problème

Les **objets** et **fonctions** créés dans le corps d’un composant sont **recréés à chaque re-render**.\
Même si leur contenu est identique, JavaScript les considère comme **différents** :

```js
const o1 = { a: 1 };
const o2 = { a: 1 };

console.log(Object.is(o1, o2)); // false
```

👉 Du coup, si tu utilises un objet ou une fonction comme dépendance dans un `useEffect`, React pensera qu’il a changé → il re-synchronise l’effet inutilement.

Exemple :

```jsx
function ChatRoom({ roomId }) {
  const options = { serverUrl: "https://localhost:1234", roomId };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ❌ provoque des reconnexions à chaque re-render
}
```

Ici, chaque saisie dans un input déclenche un re-render → nouvel objet `options` → effet relancé → déconnexion/reconnexion du chat.

***

### ✅ Les solutions

#### 1. Extraire les primitives

Utilise directement les valeurs primitives au lieu de l’objet.

```jsx
useEffect(() => {
  const connection = createConnection({
    serverUrl: "https://localhost:1234",
    roomId
  });
  connection.connect();
  return () => connection.disconnect();
}, [roomId]); // ✅ dépend seulement de roomId
```

***

#### 2. Créer l’objet dans l’effet

Déclare ton objet **dans le `useEffect`** plutôt qu’en dehors.

```jsx
useEffect(() => {
  const options = { serverUrl: "https://localhost:1234", roomId };
  const connection = createConnection(options);
  connection.connect();
  return () => connection.disconnect();
}, [roomId]);
```

***

#### 3. Mémoïser avec `useMemo` ou `useCallback`

Si tu dois absolument réutiliser un objet ou une fonction, rends-les **stables** avec `useMemo` ou `useCallback`.

```jsx
const options = useMemo(() => ({
  serverUrl: "https://localhost:1234",
  roomId
}), [roomId]);

useEffect(() => {
  const connection = createConnection(options);
  connection.connect();
  return () => connection.disconnect();
}, [options]); // ✅ options est stable
```

***

### 📌 À retenir

* ⚠️ **Objets/fonctions sont recréés à chaque re-render**, même avec le même contenu.
* ✅ Évite de les mettre directement dans les dépendances.
* Trois solutions :
  1. **Extraire les primitives**
  2. **Déclarer dans l’effet**
  3. **Stabiliser avec `useMemo` / `useCallback`**

## ✅ Déplacer les objets et fonctions statiques ou dynamiques pour gérer correctement les dépendances

### 1. 🔹 Déplacer les objets/fonctions **statiques** en dehors du composant

Si un objet ou une fonction **ne dépend d’aucune prop ni d’aucun state**, il ne changera jamais après un re-render.\
Dans ce cas, tu peux le déclarer **en dehors du composant**.

Exemple avec un objet :

```jsx
const options = {
  serverUrl: "https://localhost:1234",
  roomId: "music"
};

function ChatRoom() {
  const [message, setMessage] = useState("");

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ aucune dépendance
}
```

👉 Ici, `options` est **non réactif** → il n’a pas besoin d’être ajouté aux dépendances.\
Le linter comprend qu’il ne changera jamais, donc pas de re-synchronisation inutile.

Cela marche aussi avec une fonction :

```jsx
function createOptions() {
  return {
    serverUrl: "https://localhost:1234",
    roomId: "music"
  };
}

function ChatRoom() {
  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ aucune dépendance
}
```

***

### 2. 🔹 Déplacer les objets/fonctions **dynamiques** dans le `useEffect`

Si ton objet ou ta fonction **dépend d’une valeur réactive** (ex. `roomId`), tu **ne peux pas** le sortir du composant.\
Mais tu peux le **déclarer directement dans ton effet**.

Exemple :

```jsx
const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState("");

  useEffect(() => {
    const options = { serverUrl, roomId };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ dépend seulement de roomId
}
```

👉 Ici :

* `roomId` est une valeur **réactive** (prop).
* L’objet `options` est **créé dans l’effet**, donc **non réactif** → pas besoin de l’ajouter aux dépendances.
* Résultat : le chat **se reconnecte uniquement quand `roomId` change**, et non pas à chaque re-render.

Même principe avec une fonction locale :

```jsx
const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  useEffect(() => {
    function createOptions() {
      return { serverUrl, roomId };
    }

    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ dépend seulement de roomId
}
```

***

### 📌 À retenir

* Si un objet ou une fonction est **statique** → 📤 déclare-le **en dehors du composant**.
* Si un objet ou une fonction est **dynamique** (dépend d’une valeur réactive) → 📥 déclare-le **dans l’effet**.
* ⚠️ Évite de mettre des **objets/fonctions directement comme dépendances**, car ils changent à chaque re-render.
* ✅ Utilise uniquement les **valeurs primitives réactives** (string, number, boolean…) comme dépendances.

## ✅ Lire des valeurs primitives depuis des objets et fonctions (pour éviter des dépendances inutiles)

### 1. 🔹 Problème avec les objets passés en props

Imaginons que ton composant reçoive un objet `options` en prop :

```jsx
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ toutes les dépendances déclarées
}
```

👉 Le souci : si le parent recrée un nouvel objet `options` à chaque re-render, React considérera que `options` est **différent**, même si son contenu n’a pas changé → ton `useEffect` se ré-exécutera inutilement (et re-connectera ton chat).

Exemple parent :

```jsx
<ChatRoom
  roomId={roomId}
  options={{
    serverUrl: serverUrl,
    roomId: roomId
  }}
/>
```

Résultat : le chat se reconnecte à chaque re-render du parent → ⚠️ pas optimal.

***

### 2. 🔹 Solution : lire les valeurs primitives en dehors de l’Effect

La bonne pratique consiste à **extraire les valeurs primitives** (`roomId`, `serverUrl`) **avant l’Effect**, puis recréer l’objet **dans l’Effect** uniquement.

```jsx
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  // ✅ Extraction des valeurs primitives (non réactives)
  const { roomId, serverUrl } = options;

  useEffect(() => {
    const connection = createConnection({ roomId, serverUrl });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ dépendances explicites et stables
}
```

👉 Avantage :

* Si `options` est recréé inutilement par le parent → pas de re-connexion.
* Si `options.roomId` ou `options.serverUrl` changent réellement → re-connexion correcte.

***

### 3. 🔹 Cas similaire avec les fonctions

Ton parent peut aussi passer une **fonction** au lieu d’un objet :

```jsx
<ChatRoom
  roomId={roomId}
  getOptions={() => ({
    serverUrl: serverUrl,
    roomId: roomId
  })}
/>
```

👉 Même problème : la fonction est recréée à chaque render du parent → ton `useEffect` se ré-exécuterait inutilement.

#### ✅ Solution : appeler la fonction **avant l’Effect** et n’utiliser que les valeurs primitives :

```jsx
function ChatRoom({ getOptions }) {
  const [message, setMessage] = useState('');

  // ✅ Extraction des valeurs primitives
  const { roomId, serverUrl } = getOptions();

  useEffect(() => {
    const connection = createConnection({ roomId, serverUrl });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ dépend seulement des primitives
}
```

⚠️ Attention :

* Cela ne marche que pour des **fonctions pures** (sans effet secondaire).
* Si c’est une fonction événementielle (ex: `onClick`), utilise plutôt un **Effect Event**.

***

### 4. 📌 Récapitulatif

* Les **dépendances doivent toujours refléter le code réel**.
* Si une dépendance est gênante → change ton code, pas la liste des dépendances.
* **Ne jamais supprimer ou ignorer le linter** → ça crée des bugs cachés.
* Pour **retirer une dépendance**, il faut “prouver” au linter qu’elle n’est pas nécessaire.
* Si ton code doit répondre à une interaction précise → utilise un **event handler** au lieu d’un effet.
* Si ton `useEffect` fait plusieurs choses indépendantes → **sépare-le en plusieurs effets**.
* Si tu mets à jour un state en fonction de son ancien état → utilise une **updater function** (`setState(prev => ...)`).
* Si tu veux lire la dernière valeur sans “réagir” → utilise un **Effect Event**.
* ⚠️ En JS, deux objets/fonctions créés à des moments différents sont toujours différents → **évite les dépendances objets/fonctions**.
* ✅ Préfère les **valeurs primitives** (string, number, boolean) comme dépendances.
