# Réagir aux entrées utilisateur avec l’état

React propose une manière **déclarative** de manipuler l’interface utilisateur (UI).\
Au lieu de modifier directement chaque élément de l’UI, vous décrivez les **différents états** dans lesquels votre composant peut se trouver, puis vous passez de l’un à l’autre en réponse aux actions de l’utilisateur.

👉 C’est exactement comme la manière dont les **designers** imaginent l’UI : ils définissent des écrans et des transitions entre eux.

***

### ✅ Vous allez apprendre

* En quoi la programmation déclarative d’UI diffère de la programmation impérative.
* Comment énumérer les différents états visuels de votre composant.
* Comment déclencher les changements entre ces états à partir du code.

***

### 🔄 Déclaratif vs Impératif

Quand vous concevez des interactions d’UI, vous pensez probablement à **comment l’interface doit réagir aux actions de l’utilisateur**.\
Prenons l’exemple d’un formulaire qui permet à l’utilisateur de soumettre une réponse :

1. Quand vous tapez quelque chose dans le formulaire, le bouton **« Submit »** devient actif.
2. Quand vous cliquez sur **« Submit »**, le formulaire et le bouton deviennent désactivés, et un **spinner** (chargement) apparaît.
3. Si la requête réseau réussit, le formulaire disparaît et un message **« Merci »** s’affiche.
4. Si la requête échoue, un **message d’erreur** apparaît, et le formulaire redevient actif.

***

#### 🛠 Programmation impérative

En programmation impérative, ce scénario correspond **exactement à la façon dont vous implémentez l’interaction** : vous écrivez les instructions pas à pas qui manipulent l’UI en fonction de ce qu’il s’est passé.

👉 C’est comme si vous étiez assis à côté d’une personne anxieuse au volant d’une voiture, et que vous deviez lui dire **tour par tour** où aller.

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

_Dans une voiture conduite par une personne anxieuse représentant JavaScript, un passager donne au conducteur une série compliquée d’instructions de navigation, étape par étape._

***

Ils ne savent pas où vous voulez aller, ils suivent juste vos commandes. Et si vos instructions sont mauvaises ➝ vous arrivez au mauvais endroit.

C’est « impératif » car vous devez **commander** chaque élément, du bouton au spinner, et dire à l’ordinateur comment mettre à jour l’UI.

Voici un exemple de programmation impérative sans React, uniquement avec le DOM du navigateur :

```js
async function handleFormSubmit(e) {
  e.preventDefault();
  disable(textarea);
  disable(button);
  show(loadingMessage);
  hide(errorMessage);
  try {
    await submitForm(textarea.value);
    show(successMessage);
    hide(form);
  } catch (err) {
    show(errorMessage);
    errorMessage.textContent = err.message;
  } finally {
    hide(loadingMessage);
    enable(textarea);
    enable(button);
  }
}

function handleTextareaChange() {
  if (textarea.value.length === 0) {
    disable(button);
  } else {
    enable(button);
  }
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}
```

***

👉 Manipuler l’UI de manière impérative fonctionne bien pour des **cas isolés**, mais devient **exponentiellement plus difficile** dans des systèmes complexes.\
Imaginez une page remplie de formulaires comme celui-ci :\
ajouter un nouvel élément ou une interaction oblige à relire tout le code existant pour s’assurer qu’on n’a pas oublié de cacher ou de montrer quelque chose.

***

#### 🎨 Programmation déclarative avec React

React a été conçu pour résoudre ce problème.

Avec React :

* Vous **ne manipulez pas directement** l’UI (pas de `show`, `hide`, `enable`, `disable`).
* Vous déclarez **ce que vous voulez afficher**, et React se charge de mettre à jour l’UI.

👉 C’est comme monter dans un taxi et **dire la destination** au chauffeur, plutôt que de lui dicter chaque tournant.\
C’est au chauffeur (React) de trouver le chemin — et parfois même il connaît des raccourcis auxquels vous n’auriez pas pensé !

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

_Dans une voiture conduite par React, un passager demande à être conduit à un endroit précis sur la carte. React se charge de trouver l’itinéraire._

## Penser l’UI de manière déclarative

Vous avez vu plus haut comment implémenter un formulaire de manière **impérative**.\
Pour mieux comprendre comment penser avec React, voyons ensemble comment **réimplémenter cette UI en React** pas à pas :

1. Identifier les **différents états visuels** de votre composant
2. Déterminer **ce qui déclenche** ces changements d’état
3. Représenter l’état en mémoire avec `useState`
4. Supprimer les variables d’état non essentielles
5. Connecter les **gestionnaires d’événements** pour modifier l’état

***

### Étape 1 : Identifier les différents états visuels du composant

En informatique, vous avez peut-être entendu parler d’une **« machine à états »** (state machine), qui peut être dans un des différents « états ».\
Si vous travaillez avec un designer, vous avez sans doute déjà vu des maquettes représentant plusieurs **« états visuels »**.

👉 React se situe **à l’intersection du design et de l’informatique**, donc ces deux approches inspirent sa façon de gérer l’UI.

***

#### Les différents états possibles de notre formulaire

* **Vide (Empty)** : Le bouton « Submit » est désactivé.
* **Saisie (Typing)** : Le bouton « Submit » est activé.
* **Envoi (Submitting)** : Le formulaire est entièrement désactivé et un spinner apparaît.
* **Succès (Success)** : Le formulaire disparaît et un message « Merci » apparaît.
* **Erreur (Error)** : Même état que « Saisie », mais avec un message d’erreur en plus.

***

#### Maquettage des états (Mock UI)

Comme un designer, vous pouvez commencer par **maquetter** chaque état visuel avant même d’ajouter de la logique.

Exemple : un composant `Form` qui utilise une prop `status` (par défaut `'empty'`) :

```jsx
export default function Form({
  status = 'empty'
}) {
  if (status === 'success') {
    return <h1>C’est gagné !</h1>
  }
  return (
    <>
      <h2>Quiz : les villes</h2>
      <p>
        Dans quelle ville existe-t-il un panneau publicitaire qui transforme l’air en eau potable ?
      </p>
      <form>
        <textarea />
        <br />
        <button>
          Envoyer
        </button>
      </form>
    </>
  )
}
```

👉 Essayez de modifier `status = 'empty'` en `status = 'success'` pour voir apparaître le message de succès.

Le « mock » permet de tester rapidement l’UI **sans logique**, juste en contrôlant visuellement les états.

***

#### Prototype plus complet avec la prop `status`

Voici une version plus détaillée qui gère plusieurs cas :

```jsx
export default function Form({
  // Essayez : 'submitting', 'error', 'success'
  status = 'empty'
}) {
  if (status === 'success') {
    return <h1>C’est gagné !</h1>
  }
  return (
    <>
      <h2>Quiz : les villes</h2>
      <p>
        Dans quelle ville existe-t-il un panneau publicitaire qui transforme l’air en eau potable ?
      </p>
      <form>
        <textarea disabled={status === 'submitting'} />
        <br />
        <button
          disabled={status === 'empty' || status === 'submitting'}
        >
          Envoyer
        </button>
        {status === 'error' &&
          <p className="Error">
            Bonne tentative, mais mauvaise réponse. Réessayez !
          </p>
        }
      </form>
    </>
  );
}
```

***

### 🔎 Approfondissement : afficher plusieurs états visuels à la fois

Si un composant possède beaucoup d’états visuels, il peut être pratique de les afficher **tous en même temps sur une page**.

`App.js` :

```jsx
import Form from './Form.js';

let statuses = [
  'empty',
  'typing',
  'submitting',
  'success',
  'error',
];

export default function App() {
  return (
    <>
      {statuses.map(status => (
        <section key={status}>
          <h4>Formulaire ({status}) :</h4>
          <Form status={status} />
        </section>
      ))}
    </>
  );
}
```

👉 Ce type de pages est souvent appelé **styleguide vivant** (_living styleguide_) ou **storybook**.

## Étape 2 : Déterminer ce qui déclenche les changements d’état

Vous pouvez déclencher des mises à jour d’état en réponse à deux types d’entrées :

* **Entrées humaines**, comme cliquer sur un bouton, taper dans un champ, ou suivre un lien.
* **Entrées de l’ordinateur**, comme la réception d’une réponse réseau, l’expiration d’un délai, ou le chargement d’une image.

🖐️\
**Entrées humaines**

1️⃣0️⃣\
**Entrées de l’ordinateur**\
&#xNAN;_&#x49;llustration de Rachel Lee Nabors_

***

Dans les deux cas, vous devez **définir des variables d’état** pour mettre à jour l’UI.

👉 Pour le formulaire que nous développons, voici les déclencheurs :

* **Saisie dans le champ texte** (humain) → passe de l’état **Vide** à **Saisie** (ou inversement si le champ redevient vide).
* **Clic sur le bouton “Envoyer”** (humain) → passe à l’état **Soumission (Submitting)**.
* **Réponse réseau réussie** (ordinateur) → passe à l’état **Succès**.
* **Réponse réseau échouée** (ordinateur) → passe à l’état **Erreur**, avec un message correspondant.

⚠️ **Remarque** : Les entrées humaines nécessitent souvent des **gestionnaires d’événements** (`onClick`, `onChange`, etc.).

***

#### Diagramme du flux des états

Pour bien visualiser ce processus, vous pouvez dessiner chaque **état** comme un cercle étiqueté, et chaque changement comme une flèche entre deux cercles.

> C’est une excellente manière d’anticiper les bugs **avant même d’implémenter**.

***

📈 **Diagramme de flux (de gauche à droite)**

* Premier nœud : **empty**
  * Flèche « start typing » → **typing**
* **typing**
  * Flèche « press submit » → **submitting**
* **submitting**
  * Flèche gauche « network error » → **error**
  * Flèche droite « network success » → **success**

_Illustration : Form states_

***

## Étape 3 : Représenter l’état en mémoire avec `useState`

Ensuite, il faut représenter les **états visuels du composant en mémoire** avec `useState`.

👉 La simplicité est essentielle : chaque variable d’état est une **pièce mouvante**, et plus vous en avez, plus vous risquez de bugs.

#### État de départ minimal

On a besoin de :

* stocker la **réponse saisie**
* stocker l’**erreur éventuelle**

```jsx
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
```

Puis, une variable qui indique **quel état visuel** afficher.

#### Première tentative (pas optimale)

```jsx
const [isEmpty, setIsEmpty] = useState(true);
const [isTyping, setIsTyping] = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isError, setIsError] = useState(false);
```

⚠️ Cela fonctionne, mais ce n’est pas la meilleure manière. On peut **simplifier**.

***

## Étape 4 : Supprimer les variables d’état non essentielles

Vous devez éviter les duplications pour que l’état reste **clair et cohérent**.

💡 Quelques questions utiles :

1. **Cet état provoque-t-il un paradoxe ?**\
   Exemple : `isTyping` et `isSubmitting` ne peuvent pas être vrais en même temps.\
   👉 Solution : regrouper ça dans une variable unique comme `status`.
2. **L’information est-elle déjà disponible ailleurs ?**\
   Exemple : `isEmpty` est redondant, car on peut vérifier `answer.length === 0`.
3. **Peut-on l’obtenir par l’inverse d’une autre valeur ?**\
   Exemple : `isError` est inutile car on peut vérifier `error !== null`.

***

#### Résultat après nettoyage : seulement 3 variables essentielles

```jsx
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing'); 
// valeurs possibles : 'typing', 'submitting', 'success'
```

👉 Ces trois variables suffisent pour représenter tous les états sans paradoxe.

***

### 🔎 Approfondissement : Éliminer les « états impossibles » avec un reducer

Même avec 3 variables, certains cas n’ont pas de sens (ex : `error` non nul quand `status === 'success'`).

👉 Pour mieux structurer, vous pouvez utiliser un **reducer**, qui regroupe tout l’état dans un objet et gère les transitions de manière centralisée.

***

## Étape 5 : Connecter les gestionnaires d’événements à l’état

Enfin, vous pouvez écrire les gestionnaires qui **mettent à jour l’état**.

Voici la version finale du formulaire :

```jsx
import { useState } from 'react';

export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  if (status === 'success') {
    return <h1>C’est gagné !</h1>
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>Quiz : les villes</h2>
      <p>
        Dans quelle ville existe-t-il un panneau publicitaire qui transforme l’air en eau potable ?
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === 'submitting'}
        />
        <br />
        <button
          disabled={answer.length === 0 || status === 'submitting'}
        >
          Envoyer
        </button>
        {status === 'error' && (
          <p className="Error">{error.message}</p>
        )}
      </form>
    </>
  );
}
```

***

## ✅ Récapitulatif

* La programmation **déclarative** consiste à décrire l’UI pour chaque état visuel, plutôt que de la manipuler étape par étape (**impérative**).
* Pour développer un composant React :
  1. Identifiez tous les **états visuels**.
  2. Déterminez les **déclencheurs humains et machines**.
  3. Représentez l’état avec `useState`.
  4. Supprimez les états non essentiels pour éviter paradoxes et incohérences.
  5. Connectez les **gestionnaires d’événements** pour mettre à jour l’état.
