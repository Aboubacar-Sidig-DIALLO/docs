# 🔄 Préservation et réinitialisation de l’état

En React, **chaque composant a son propre état isolé**.\
React garde la trace de **quel état appartient à quel composant** en fonction de sa position dans l’**arbre de l’UI**.

👉 Cela signifie que lorsqu’un composant reste à la **même place dans l’arbre**, son état est **préservé**.\
👉 Mais si un composant est **remplacé** par un autre, ou que sa **clé (`key`) change**, son état est **réinitialisé**.

***

### 📚 Tu vas apprendre

1. **Quand React choisit de préserver ou réinitialiser l’état**.
2. **Comment forcer React à réinitialiser l’état d’un composant**.
3. **Comment les clés (`key`) et les types de composants influencent la préservation de l’état**.

***

### ✨ Exemple concret

Imaginons une petite app de chat :

```jsx
import { useState } from 'react';

function Chat({ contact }) {
  const [message, setMessage] = useState("");

  return (
    <section>
      <h2>Chat avec {contact.name}</h2>
      <textarea
        value={message}
        onChange={(e) => setMessage(e.target.value)}
      />
    </section>
  );
}

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={setTo}
      />
      <Chat contact={to} />
    </div>
  );
}

const contacts = [
  { name: 'Alice', email: 'alice@mail.com' },
  { name: 'Bob', email: 'bob@mail.com' }
];
```

***

#### 🔹 Problème :

Si tu tapes un message dans la boîte de `Chat`, puis tu passes de _Alice_ à _Bob_, le texte reste !

👉 Pourquoi ?\
Parce que **React considère que c’est le même composant `Chat`** (même type, même position dans l’arbre). Donc, son état (`message`) est **préservé**.

***

#### 🔹 Solution : forcer la réinitialisation avec une clé

Tu peux dire à React : “Considère ce `Chat` comme un nouveau composant à chaque changement de destinataire” → grâce à la prop spéciale **`key`** :

```jsx
<Chat key={to.email} contact={to} />
```

Désormais, si tu changes de contact :

* React démonte l’ancien `Chat` (avec son état).
* Il monte un **nouveau `Chat`**, avec un état réinitialisé.

### ✅ Récap

* React garde l’état des composants **par leur type et leur position dans l’arbre**.
* Pour **forcer la réinitialisation**, change leur `key` ou leur type.
* Utilise cette technique pour éviter des incohérences (ex. un champ texte qui garde la mauvaise valeur).
* Par défaut, **React essaie de préserver l’état** pour éviter de tout réinitialiser inutilement → cela améliore l’expérience utilisateur.

***

### 📌 Règles de préservation / réinitialisation de l’état

1. **Même type + même position = état préservé**
   * Exemple : `<Chat />` remplacé par un autre `<Chat />` → conserve son `useState`.
2. **Type différent = état réinitialisé**
   * Exemple : `<Chat />` remplacé par `<VideoCall />` → l’état est perdu car ce n’est pas le même composant.
3. **Clé différente = état réinitialisé**
   * Même type, mais clé différente → React considère que c’est un composant **nouveau**.

## 📌 L’état est lié à une position dans l’arbre de rendu

React construit des **arbres de rendu** pour représenter la structure des composants dans ton UI.

Quand tu donnes un état à un composant avec `useState`, tu pourrais penser que cet état “vit” **à l’intérieur du composant**.\
👉 En réalité, l’état est stocké **par React lui-même**.

React associe chaque morceau d’état au bon composant en fonction de **sa position dans l’arbre de rendu**.

***

### 🔹 Exemple 1 : même composant, deux positions différentes

```jsx
import { useState } from 'react';

export default function App() {
  const counter = <Counter />;
  return (
    <div>
      {counter}
      {counter}
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

***

<figure><img src="../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

📖 **Description :**\
Un `<div>` racine contient deux enfants `<Counter>`.

* Chaque `<Counter>` a sa propre “bulle d’état” (`count: 0`).

```
div
 ├── Counter (state: count=0)
 └── Counter (state: count=0)
```

👉 Même si c’est le **même JSX `<Counter />`**, comme ils sont à deux positions différentes, **React leur donne deux états distincts**.

***

### 🔹 Exemple 2 : deux compteurs indépendants

```jsx
export default function App() {
  return (
    <div>
      <Counter />
      <Counter />
    </div>
  );
}
```

Chaque `Counter` a son propre `score` et son propre `hover`.

***

<figure><img src="../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

📖 **Description :**\
Si tu incrémentes uniquement le compteur de droite&#x20;

div\
├── Counter (state: count=0)\
└── Counter (state: count=1)  ←  mis à jour

👉 **Chaque état est totalement isolé** : mettre à jour un `Counter` ne touche pas l’autre.

***

### 🔹 Exemple 3 : suppression puis ajout d’un composant

```jsx
import { useState } from 'react';

export default function App() {
  const [showB, setShowB] = useState(true);
  return (
    <div>
      <Counter />
      {showB && <Counter />}
      <label>
        <input
          type="checkbox"
          checked={showB}
          onChange={e => setShowB(e.target.checked)}
        />
        Render the second counter
      </label>
    </div>
  );
}
```

***

<figure><img src="../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

📖 **Description :**\
Quand tu décoches la case, le 2e compteur est supprimé → **son état disparaît aussi**.

```
div
 ├── Counter (state: count=0)
 └── ❌ (Counter supprimé → état détruit)
```

***

<figure><img src="../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

📖 **Description :**\
Quand tu recoches la case, React crée **un nouveau Counter**, avec un état initial (`count = 0`).

```
div
 ├── Counter (state: count=0)
 └── Counter (state: count=0)  ← nouvellement créé
```

***

### ✅ Récap

* L’état est lié à la **position du composant dans l’arbre de rendu**.
* Si un composant est **rendu à la même position**, React préserve son état.
* Si tu **supprimes un composant**, son état est détruit.
* Si tu rends un **autre composant au même endroit**, React réinitialise l’état.

## 📌 Le même composant à la même position conserve son état

Dans React, l’état d’un composant est lié à sa **position dans l’arbre de rendu**, pas à la balise JSX exacte.

Cela signifie que si le **même composant** reste au **même emplacement** dans la structure retournée, React **préserve son état**, même si ses props changent.

***

### 🔹 Exemple : Composant `<Counter />` avec une prop

```jsx
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <Counter isFancy={true} /> 
      ) : (
        <Counter isFancy={false} /> 
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

***

<figure><img src="../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

📖 **Description :**\
Quand tu coches la case :

* `App` change `isFancy` de `false` → `true`.
* Le `Counter` garde son état (`count = 3` par exemple).

**Avant (isFancy = false)**

```
App (state: isFancy=false)
 └── div
     └── Counter (state: count=3, hover=false, props: isFancy=false)
```

**Après (isFancy = true)**

```
App (state: isFancy=true)
 └── div
     └── Counter (state: count=3, hover=false, props: isFancy=true)
```

👉 Le `Counter` garde son état car il reste **le premier enfant du `<div>`**, donc au **même emplacement dans l’arbre**.

***

### ⚠️ Piège : la position dans l’arbre, pas dans le code JSX

Regarde ce code :

```jsx
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  if (isFancy) {
    return (
      <div>
        <Counter isFancy={true} />
        <label>
          <input
            type="checkbox"
            checked={isFancy}
            onChange={e => setIsFancy(e.target.checked)}
          />
          Use fancy styling
        </label>
      </div>
    );
  }
  return (
    <div>
      <Counter isFancy={false} />
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => setIsFancy(e.target.checked)}
        />
        Use fancy styling
      </label>
    </div>
  );
}
```

Tu pourrais croire que l’état devrait se réinitialiser en cochant la case, puisque ce sont **deux `<Counter />` différents** dans le JSX.

Mais React ne “voit” pas ton `if`.\
👉 Tout ce qu’il voit, c’est que dans les deux cas :

* L’arbre retourné a un `<div>`
* Avec un `<Counter />` comme **premier enfant**

Donc il conserve l’état.

***

📖 **Description :**\
Que `isFancy` soit `true` ou `false`, le `<Counter />` reste le premier enfant du `<div>`.

**Cas 1 : isFancy = false**

```
App
 └── div
     └── Counter (props: isFancy=false, state: count=5)
```

**Cas 2 : isFancy = true**

```
App
 └── div
     └── Counter (props: isFancy=true, state: count=5)
```

👉 La **position est la même**, donc React considère que c’est le même composant. Son état est **préservé**.

***

### ✅ Récap

* React conserve l’état d’un composant tant qu’il est rendu **au même emplacement dans l’arbre de rendu**.
* Peu importe si le JSX change avec des `if` ou des conditions.
* Ce qui compte, c’est l’**adresse du composant dans l’arbre**, pas la façon dont tu écris ton JSX.

## 📌 Différents composants au même emplacement réinitialisent l’état

React conserve l’état d’un composant tant que **le même type de composant** est rendu au **même emplacement dans l’arbre**.\
Mais si tu remplaces ce composant par un **type différent**, React **détruit l’ancien composant et son état**, puis crée le nouveau.

***

### 🔹 Exemple 1 : Remplacer `<Counter />` par `<p>`

```jsx
import { useState } from 'react';

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? (
        <p>See you later!</p> 
      ) : (
        <Counter /> 
      )}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={e => {
            setIsPaused(e.target.checked)
          }}
        />
        Take a break
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

***

<figure><img src="../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

📖 **Description :**

* Quand `isPaused = false`, le premier enfant du `<div>` est `<Counter />`.
* Quand `isPaused = true`, il devient `<p>See you later!</p>`.

1️⃣ Avant :

```
div
 └── Counter (state: count=3)
```

2️⃣ Transition (suppression du Counter) :

```
div
 └── 💥 suppression de Counter
```

3️⃣ Après :

```
div
 └── p ("See you later!")
```

👉 Quand tu reviens à `isPaused = false`, le `<p>` est détruit, et un **nouveau `<Counter />` est recréé avec son état réinitialisé (count = 0)**.

***

### 🔹 Exemple 2 : Même composant enfant, mais parent différent

```jsx
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <div>
          <Counter isFancy={true} /> 
        </div>
      ) : (
        <section>
          <Counter isFancy={false} />
        </section>
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

***

<figure><img src="../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

📖 **Description :**\
Même si `<Counter />` reste présent, il change de **parent (`section` → `div`)**.\
Comme React doit détruire l’ancien parent, il supprime aussi tout l’arbre enfant, donc **le compteur est recréé à 0**.

1️⃣ Avant :

```
div
 └── section
     └── Counter (state: count=3)
```

2️⃣ Transition (suppression du section et de ses enfants) :

```
div
 └── 💥 suppression de section et Counter
```

3️⃣ Après :

```
div
 └── div
     └── Counter (state: count=0)
```

👉 Résultat : l’état de `<Counter />` est réinitialisé car **son chemin dans l’arbre a changé**.

***

### ⚠️ Piège : Définir un composant dans un autre

```jsx
import { useState } from 'react';

export default function MyComponent() {
  const [counter, setCounter] = useState(0);

  function MyTextField() {
    const [text, setText] = useState('');
    return (
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
    );
  }

  return (
    <>
      <MyTextField />
      <button onClick={() => {
        setCounter(counter + 1)
      }}>
        Clicked {counter} times
      </button>
    </>
  );
}
```

***

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

📖 **Description :**\
À chaque fois que `MyComponent` est rendu, une **nouvelle fonction `MyTextField` est recréée**.\
Donc React considère que c’est un **nouveau composant**, et son état (`text`) est perdu.

1️⃣ Premier rendu :

```
MyComponent
 └── MyTextField (state: text="Hello")
```

2️⃣ Re-rendu après clic :

```
MyComponent
 └── 💥 suppression de l’ancien MyTextField
 └── Nouveau MyTextField (state: text="")
```

👉 Résultat : le champ texte est **réinitialisé à chaque clic**.

✅ Solution : toujours déclarer les composants **au niveau supérieur** du fichier, pas à l’intérieur d’un autre composant.

***

### ✅ Récap

* Remplacer un composant par un autre **au même emplacement** → état détruit.
* Même composant mais **parent différent** → état détruit (car l’arbre change).
* Ne **jamais définir un composant dans un autre**, sinon React croit que c’est un nouveau composant à chaque rendu.

## 🔄 Réinitialiser l’état au même emplacement

👉 Par défaut, React **préserve l’état** d’un composant tant qu’il reste au même emplacement dans l’arbre du rendu.\
C’est généralement ce que l’on veut.\
Mais parfois, on veut **réinitialiser l’état** (par exemple, un score qui doit repartir de 0 quand un nouveau joueur commence).

***

### 🔹 Exemple initial : deux joueurs, un compteur partagé

```jsx
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter person="Taylor" />
      ) : (
        <Counter person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

***

📌 Problème : quand on clique sur **Next player**, le score **est conservé**.\
Car pour React, c’est toujours le même `<Counter />` au même emplacement.

***

## ⚙️ Deux solutions pour réinitialiser l’état

***

### ✅ Option 1 : Rendre les composants à des **positions différentes**

```jsx
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}
```

***

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

1️⃣ **État initial (Taylor actif)**

```
Scoreboard (isPlayerA=true)
 └── Counter (Taylor, score=0)
```

🖼️ Diagramme : l’arbre montre `Scoreboard` avec une bulle `isPlayerA=true`.\
À gauche, un `Counter` (Taylor) avec `score=0` apparaît en **jaune (ajouté)**.

***

2️⃣ **Après clic (Sarah active)**

```
Scoreboard (isPlayerA=false)
 └── Counter (Sarah, score=0)
```

🖼️ Diagramme : la bulle `isPlayerA` passe à **false (en jaune)**.\
Le `Counter` de gauche est supprimé (💥 poof), et un nouveau `Counter` (Sarah) apparaît à droite en **jaune**.

***

3️⃣ **Encore un clic (retour à Taylor)**

```
Scoreboard (isPlayerA=true)
 └── Counter (Taylor, score=0)
```

🖼️ Diagramme : la bulle `isPlayerA` repasse à **true (en jaune)**.\
Un nouveau `Counter` (Taylor) est ajouté à gauche en **jaune**, et celui de Sarah disparaît (💥 poof).

***

👉 Résultat : chaque fois qu’un joueur change, son compteur est **recréé**, donc le score est remis à 0.

***

### ✅ Option 2 : Utiliser une **clé (`key`)**

⚡ C’est la méthode la plus générique.\
Même si deux composants sont au **même emplacement JSX**, React les distinguera grâce à une clé unique.

```jsx
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}
```

***

1️⃣ **Taylor actif (clé = "Taylor")**

```
Scoreboard (isPlayerA=true)
 └── Counter (key=Taylor, score=0)
```

`Counter` avec **clé "Taylor"** ajouté en jaune.

***

2️⃣ **Sarah active (clé = "Sarah")**

```
Scoreboard (isPlayerA=false)
 └── Counter (key=Sarah, score=0)
```

le `Counter` avec clé "Taylor" est supprimé (💥 poof),\
et un nouveau `Counter` avec clé "Sarah" est ajouté en jaune.

***

3️⃣ **Retour à Taylor**

```
Scoreboard (isPlayerA=true)
 └── Counter (key=Taylor, score=0)
```

&#x20;`Counter` "Sarah" est supprimé (💥), et celui de "Taylor" est recréé (jaune).

***

👉 Résultat : grâce à la clé, React **voit ces compteurs comme deux entités distinctes**, donc **leurs états ne sont jamais partagés**.\
À chaque apparition/disparition, l’état est **réinitialisé**.

***

⚠️ **Note importante :**\
Les clés (`key`) ne sont pas globalement uniques. Elles doivent juste être uniques **par rapport à leurs frères dans le même parent**.

***

### 📌 Récap

* Par défaut : React préserve l’état tant que le composant reste au même emplacement.
* ✅ Solution 1 : rendre les composants dans **des positions différentes** → état réinitialisé.
* ✅ Solution 2 : donner une **clé (`key`) unique** → React considère que ce sont des composants différents → état réinitialisé.

## Réinitialiser un formulaire avec une clé

Réinitialiser l’état avec une **clé (`key`)** est particulièrement utile pour les **formulaires**.

Dans cette appli de chat, le composant `<Chat>` contient l’état du champ de saisie :

```jsx
// App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat contact={to} />
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice',  email: 'alice@mail.com'  },
  { id: 2, name: 'Bob',    email: 'bob@mail.com'    }
];
```

🧪 **Essayez** : tapez un message, puis cliquez sur « Alice » ou « Bob » pour changer de destinataire.\
Vous verrez que le contenu du champ **est conservé**, car `<Chat>` est rendu **au même emplacement** dans l’arbre.

Dans une appli de chat, ce n’est **pas** souhaitable : on ne veut pas risquer d’envoyer par erreur un message tapé à la mauvaise personne.\
👉 La solution : **ajouter une clé** au composant :

```jsx
<Chat key={to.id} contact={to} />
```

Ainsi, quand vous sélectionnez un autre destinataire, **React recrée `<Chat>` depuis zéro**, y compris tout son état et les éléments du DOM en dessous.\
Résultat : **le champ de saisie est vidé** à chaque changement de destinataire.

Voici la version complète corrigée :

```jsx
// App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat key={to.id} contact={to} />
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice',  email: 'alice@mail.com'  },
  { id: 2, name: 'Bob',    email: 'bob@mail.com'    }
];
```

***

### Approfondir

**Conserver l’état des composants supprimés**

***

### Récapitulatif

* React conserve l’état **tant que le même composant** est rendu **au même emplacement**.
* L’état n’est pas attaché aux balises JSX elles-mêmes : il est associé à **la position dans l’arbre** où ces balises sont rendues.
* Vous pouvez **forcer la réinitialisation** d’un sous-arbre en lui donnant une **clé différente (`key`)**.
* **N’imbriquez pas** les définitions de composants à l’intérieur d’autres composants, sous peine de **réinitialiser l’état** par inadvertance.

