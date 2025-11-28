# Handlebars

### 🔎 Qu’est-ce que Handlebars ?

* Handlebars est un **moteur de templates** simple, utilisé pour générer du HTML (ou tout autre texte) à partir d’un **template** + un **objet de données**. [handlebarsjs.com+2Wikipédia+2](https://handlebarsjs.com/guide)
* Il a été créé en 2010 par Yehuda Katz. [Wikipédia+1](https://fr.wikipedia.org/wiki/Handlebars_\(moteur_de_template\)?utm_source=chatgpt.com)
* Handlebars étend un moteur plus minimaliste, Mustache — en apportant des fonctionnalités additionnelles comme la logique (conditions, boucles…) tout en restant compatible avec les gabarits Mustache. [Wikipédia+1](https://fr.wikipedia.org/wiki/Handlebars_\(moteur_de_template\)?utm_source=chatgpt.com)
* Concrètement, on écrit un template avec des **placeholders** (espaces réservés) et quand on l’exécute avec un objet de données, Handlebars remplace ces placeholders par les valeurs correspondantes. [handlebarsjs.com+1](https://handlebarsjs.com/guide)

Cette approche permet de séparer clairement la **structure/modèle** (layout, HTML, mise en page…) de la **données** — ce qui rend le code plus propre, modulaire, facile à maintenir.

***

### 📝 Syntaxe de base : expressions, contextes, évaluation

#### Expressions (mustaches)

* Une **expression simple** s’écrit entre `{{` et `}}`. Par exemple : `{{nom}}`, ou `{{user.age}}` si `user` est un objet. Lors de l’exécution, Handlebars remplace l’expression par la valeur correspondante dans l’objet de données. [handlebarsjs.com+1](https://handlebarsjs.com/guide)
* Si l’objet de données contient des objets imbriqués (nested), on peut accéder aux sous-propriétés avec la **notation pointée** (dot-notation), par exemple `{{user.name}}`, `{{user.address.city}}`. [handlebarsjs.com](https://handlebarsjs.com/guide)
* Si tu as besoin d’insérer du HTML brut (et non échappé), tu peux utiliser les **triple accolades** `{{{…}}}` — Handlebars insère alors la valeur “telle quelle”, sans échapper les caractères spéciaux. [handlebarsjs.com+1](https://handlebarsjs.com/guide)

#### Contexte d’évaluation (context)

* Le “contexte” (context) correspond à l’objet passé au template. Par défaut, les expressions récupèrent des données dans ce contexte. [handlebarsjs.com+1](https://handlebarsjs.com/guide)
* Grâce aux helpers intégrés (voir plus bas), tu peux **changer le contexte courant** à l’intérieur d’une portion de template — ce qui facilite l’accès à des sous-objets profonds, sans répéter les chemins complets. [handlebarsjs.com+1](https://handlebarsjs.com/guide)

***

### 🧩 Partials — Réutilisation de templates

Quand tu veux **réutiliser** des morceaux de template (comme un header, un footer, un composant, etc.), tu utilises les **partials**. Un partial est simplement un petit template “inclus” dans un autre. [handlebarsjs.com+1](https://handlebarsjs.com/guide/partials.html?utm_source=chatgpt.com)

#### Comment ça marche ?

* Tu définis (et enregistres) un partial via `Handlebars.registerPartial("nomDuPartial", leTemplateOuLaFonction)`. [handlebarsjs.com+1](https://handlebarsjs.com/guide/partials.html?utm_source=chatgpt.com)
* Dans ton template principal, tu l’appelles via `{{> nomDuPartial }}`. À cet endroit, le partial sera rendu avec le contexte courant. [handlebarsjs.com+1](https://handlebarsjs.com/guide/partials.html?utm_source=chatgpt.com)
* Tu peux aussi passer un **contexte personnalisé** au partial (par exemple un sous-objet), ou des **paramètres supplémentaires** (hash params). [handlebarsjs.com](https://handlebarsjs.com/guide/partials.html?utm_source=chatgpt.com)
* Il existe des “dynamic partials”, c’est-à-dire des partials dont le nom est déterminé à l’exécution (utile quand tu voulez choisir quel partial inclure selon des conditions) via des sub-expressions. [handlebarsjs.com](https://handlebarsjs.com/guide/partials.html?utm_source=chatgpt.com)
* Enfin, Handlebars offre la possibilité de définir des **partials inline** (dans le block de template), pour des cas particuliers de portée locale. [handlebarsjs.com](https://handlebarsjs.com/guide/partials.html?utm_source=chatgpt.com)

Les partials sont un outil puissant pour structurer un projet, factoriser le code, éviter les duplications, et construire des templates modulaires.

***

### ⚙️ Helpers — logique, boucles, conditions, etc.

C’est l’un des aspects qui distingue Handlebars d’un simple moteur “statique” : les helpers permettent d’introduire de la **logique** dans les templates (sans pour autant retourner à un code full-on scripting). [handlebarsjs.com+2Wikipédia+2](https://handlebarsjs.com/guide)

#### Helpers intégrés (built-in)

Handlebars fournit quelques helpers de base très utiles :

**`#if` / `#unless` — Conditions**

* `{{#if condition}}…{{/if}}` : affiche le contenu du block si `condition` est “truthy” (non nulle, non vide, non 0, etc.). [handlebarsjs.com+1](https://handlebarsjs.com/guide/builtin-helpers.html?utm_source=chatgpt.com)
* On peut ajouter un `{{else}}` pour gérer le cas “faux”. [handlebarsjs.com](https://handlebarsjs.com/guide/builtin-helpers.html?utm_source=chatgpt.com)
* Il existe une option `includeZero=true` si tu veux que `0` soit considéré comme “vrai” dans la condition. [handlebarsjs.com](https://handlebarsjs.com/guide/builtin-helpers.html?utm_source=chatgpt.com)
* `{{#unless condition}}…{{/unless}}` rend le block si la condition est “falsy”. [handlebarsjs.com+1](https://handlebarsjs.com/guide/builtin-helpers.html?utm_source=chatgpt.com)

**`#each` — Itération (boucle)**

* `{{#each list}} … {{/each}}` : pour parcourir une **liste (array)**. À l’intérieur du block, `this` réfère à l’élément courant. [handlebarsjs.com+1](https://handlebarsjs.com/guide/builtin-helpers.html?utm_source=chatgpt.com)
* Tu peux accéder à des variables de boucle, comme `@index` (index courant), `@first`, `@last`. [handlebarsjs.com](https://handlebarsjs.com/guide/builtin-helpers.html?utm_source=chatgpt.com)
* Si tu itères sur un **objet** (pas un tableau), tu peux récupérer la clé via `@key`. [handlebarsjs.com](https://handlebarsjs.com/guide/builtin-helpers.html?utm_source=chatgpt.com)
* Tu peux aussi fournir un `{{else}}` au `each` : si la liste est vide, la section else sera rendue. [handlebarsjs.com+1](https://handlebarsjs.com/guide/builtin-helpers.html?utm_source=chatgpt.com)

**`#with` — Changer de contexte**

* `{{#with object}} … {{/with}}` : permet de “descendre” dans un sous-objet pour que ses propriétés soient accessibles directement (sans préfixe). [handlebarsjs.com+1](https://handlebarsjs.com/guide/builtin-helpers.html?utm_source=chatgpt.com)
* Utile quand tu as des objets imbriqués, pour éviter d’écrire `{{user.address.street}}` tout le temps — tu peux faire `{{#with user.address}} … {{street}} … {{/with}}`. [handlebarsjs.com+1](https://handlebarsjs.com/guide)

#### Custom Helpers — tes propres fonctions

Quand les helpers intégrés ne suffisent pas, tu peux définir **tes propres helpers** en JavaScript (ou le langage supporté). [handlebarsjs.com+2assemble.io+2](https://handlebarsjs.com/guide)

Principe de base :

```js
Handlebars.registerHelper("monHelper", function(arg1, arg2, /* … */, options) {
  // logic here
  return résultat;  // généralement une string ou une SafeString
});
```

* `this` à l’intérieur du helper correspond au **contexte courant**. [handlebarsjs.com+1](https://handlebarsjs.com/guide)
* Si le helper génère du HTML, tu peux renvoyer un `new Handlebars.SafeString(stringHTML)` pour éviter un double-échappement. [handlebarsjs.com+1](https://handlebarsjs.com/guide)
* Ces helpers peuvent être “inline” (utilisés comme expression simple) ou “block helpers” (similaires à `#if`, `#each`…) selon la façon dont tu les écris. [assemble.io+1](https://assemble.io/docs/Custom-Helpers.html?utm_source=chatgpt.com)

Avec les helpers personnalisés, tu peux ajouter n’importe quelle logique de traitement — formatage, calcul, filtrage, transformation, etc.

***

### 🧱 Block Helpers — blocs avec logique & contexte particulier

Les **block helpers** sont des helpers qui entourent un bloc de template avec une logique (boucle, condition, contexte modifié…). Dans la syntaxe, un block helper commence par `{{#helperName …}}` et se termine par `{{/helperName}}`. [handlebarsjs.com+2handlebarsjs.com+2](https://handlebarsjs.com/guide)

Quelques aspects importants :

* Les **built-in** `#if`, `#unless`, `#each`, `#with` sont tous des block helpers. [handlebarsjs.com+1](https://handlebarsjs.com/guide/builtin-helpers.html?utm_source=chatgpt.com)
* Tu peux créer des block helpers personnalisés — par exemple pour générer une liste HTML, ajouter des balises, manipuler le contexte, etc. [handlebarsjs.com+2handlebarsjs.com+2](https://handlebarsjs.com/guide)
* Le block helper reçoit en paramètre une “options hash” (un objet `options`), dans lequel se trouve notamment une fonction `fn(context)` : c’est cette fonction qui rend (évalue) le contenu du bloc, avec le contexte que tu lui fournis. [handlebarsjs.com+1](https://handlebarsjs.com/guide/block-helpers.html?utm_source=chatgpt.com)
* Si ton block helper génère du texte/HTML, et que tu voulez éviter l’échappement automatique (ou gérer l’échappement toi-même), tu dois renvoyer un `SafeString`. [handlebarsjs.com+1](https://handlebarsjs.com/guide)

Exemple d’un block helper “fait maison” :

```js
Handlebars.registerHelper("bold", function(options) {
  return new Handlebars.SafeString(
    "<strong>" + options.fn(this) + "</strong>"
  );
});
```

Et dans ton template :

```hbs
{{#bold}}Texte en gras{{/bold}}
```

Le contenu “Texte en gras” sera rendu entre `<strong>…</strong>`.

***

### 🔐 Échappement HTML & sécurité

Parce que Handlebars est conçu initialement pour générer du HTML, il **échappe par défaut** les valeurs insérées via `{{…}}`: les caractères spéciaux seront convertis en entités HTML pour éviter les problèmes XSS. [handlebarsjs.com+1](https://handlebarsjs.com/guide)

* Si tu as besoin d’insérer du HTML “brut”, tu utilises la **triple-curly** `{{{…}}}`. [handlebarsjs.com](https://handlebarsjs.com/guide)
* Si tu écris un helper qui génère du HTML, renvoie un `SafeString`. [handlebarsjs.com+1](https://handlebarsjs.com/guide)
* Attention : Handlebars n’échappe pas les chaînes JavaScript — donc si tu génères du code JS inline (événements, etc.), il faut faire attention toi-même aux risques de sécurité. [handlebarsjs.com](https://handlebarsjs.com/guide)

***

### 🧮 Installation, compilation, usage

*   Pour tester rapidement : tu peux charger Handlebars depuis un CDN dans une page HTML, compiler un template, et l’exécuter avec des données. Exemple minimal :

    ````html
    <script src="https://cdn.jsdelivr.net/npm/handlebars@latest/dist/handlebars.js"></script>
    <script>
      var template = Handlebars.compile("Bonjour, {{name}} !");
      console.log(template({ name: "Alice" }));  // → "Bonjour, Alice !"
    </script>
    ``` :contentReference[oaicite:42]{index=42}

    ````
* Pour un usage en production / projet sérieux, il existe d'autres façons (pré-compilation, intégration à des frameworks, bundlers, etc.). [handlebarsjs.com+2handlebarsjs.com+2](https://handlebarsjs.com/guide)
* La pré-compilation des templates (compiling ahead of time) est souvent recommandée pour de meilleures performances, surtout quand le template est utilisé plusieurs fois. [handlebarsjs.com+1](https://handlebarsjs.com/guide)

***

### ✅ Quand (et quand **ne pas**) utiliser Handlebars

Handlebars est très utile quand :

* Tu as des templates HTML ou texte à générer dynamiquement à partir de données.
* Tu veux séparer **structure (layout, mise en page)** et **données**.
* Tu veux garder les choses simples, sans plonger dans un templating trop “code-dense”.

Mais ce n’est **pas toujours** le bon choix :

* Si tu as besoin d’une logique très complexe, de traitements lourds, de manipulations de données poussées — tu feras peut-être mieux avec un vrai moteur de vue (avec code), ou côté backend.
* Si tu injectes beaucoup de code JS inline dans tes templates — attention aux risques de sécurité.

L’idée de base de Handlebars est de rester simple, clair, “template + données” + un peu de logique via helpers au besoin — mais sans faire du “full programming” dans les templates. [handlebarsjs.com+2handlebarsjs.com+2](https://handlebarsjs.com/guide)

***

### ℹ️ Quelques concepts importants résumés

| Concept                        | Description / Utilité                                                                                     |
| ------------------------------ | --------------------------------------------------------------------------------------------------------- |
| Template + données             | Base de Handlebars : du texte “statique” + des expressions `{{…}}` remplacées par des données dynamiques. |
| Mustache / expressions simples | `{{variable}}`, `{{objet.propriété}}` — injection simple de données.                                      |
| Triple-curly `{{{…}}}`         | Pour insérer du HTML “brut” (non échappé).                                                                |
| Contexte (context)             | L’objet de départ +, via helpers, des sous-contextes pour accéder à des sous-objets.                      |
| Partials                       | Fragments de templates réutilisables, pour modulariser le code.                                           |
| Helpers (built-in)             | Logique “light” : boucles (`each`), conditions (`if`, `unless`), changement de contexte (`with`).         |
| Helpers personnalisés          | Pour étendre Handlebars selon tes besoins (formatage, logique, transformation…).                          |
| Block helpers                  | Helpers qui encadrent un bloc de template — très utiles pour logique + contexte.                          |
| Pré-Compilation                | Pour améliorer les performances en production.                                                            |
| Échappement / sécurité         | Sortie HTML échappée par défaut — attention si insertion de HTML brut ou JS inline.                       |

***

### 🧠 Quelques conseils & bonnes pratiques

* Utilise les **partials** pour éviter la duplication et garder des templates clairs, surtout dans des projets moyens à grands.
* Garde la **logique dans les helpers** — les templates doivent rester lisibles : masquer la complexité dans des fonctions JS.
* Quand tu écris un helper qui génère du HTML, renvoie un `SafeString`, mais pense à **nettoyer/échapper les données entrantes** pour éviter les risques XSS.
* Si tu utilises des structures imbriquées (objets dans objets, tableaux dans objets…), structure bien tes données d’entrée pour que la notation reste claire dans le template.
* Précompile tes templates si tu comptes les utiliser souvent — cela accélère le rendu.
