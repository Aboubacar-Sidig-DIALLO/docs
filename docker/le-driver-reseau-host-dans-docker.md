# 🌐 Le driver réseau host dans Docker

## Docker

#### 🔹 Définition

Quand tu utilises le mode réseau **host** pour un conteneur :

* Le conteneur **partage directement la pile réseau de l’hôte** (même espace de noms réseau).
* Le conteneur **n’a pas sa propre adresse IP** attribuée.
* Si ton application écoute sur le port `80` dans le conteneur, elle sera disponible **directement sur le port `80` de l’hôte**.

***

#### ⚠️ Note importante

Comme le conteneur **n’a pas d’IP propre**, le **port mapping est ignoré**.\
👉 Les options `-p`, `--publish`, `-P`, `--publish-all` sont sans effet et affichent un avertissement :

```
WARNING: Published ports are discarded when using host network mode
```

***

#### 🔹 Cas d’utilisation du mode host

* **Optimisation des performances**\
  (pas de NAT, pas de "userland-proxy" créé pour chaque port).
* **Gestion d’un grand nombre de ports**\
  car le conteneur peut directement utiliser les ports de l’hôte.

***

#### 🔹 Compatibilité

* Disponible sur **Docker Engine (Linux uniquement)**.
* Depuis **Docker Desktop 4.34 et plus récent** (Linux containers only).

***

## 🐳 Utiliser host networking avec Swarm

Tu peux aussi utiliser le réseau **host** avec un service swarm :

```bash
docker service create --network host my-service
```

👉 Le **trafic de contrôle** (gestion du swarm) passe toujours par un réseau **overlay**,\
mais les conteneurs du service utilisent le réseau et les ports de l’hôte.

⚠️ Limitation :\
Si un conteneur de service écoute sur le port `80`, **un seul conteneur** peut tourner par nœud swarm (car ils partagent le même réseau hôte).

***

## 🖥️ Docker Desktop et host networking

Pour activer cette option sur Docker Desktop (>= v4.34) :

1. Connecte-toi à ton compte Docker Desktop.
2. Va dans **Settings > Resources > Network**.
3. Active **Enable host networking**.
4. Clique sur **Apply and restart**.

👉 Fonctionne dans les deux sens :

* Un serveur dans un conteneur est accessible depuis l’hôte.
* Un serveur sur l’hôte est accessible depuis le conteneur en mode `host`.
* Supporte TCP et UDP.

***

## 🛠️ Exemples pratiques

#### 1. Lancer un serveur dans un conteneur

Exemple avec `netcat` écoutant sur le port `8000` :

```bash
docker run --rm -it --net=host nicolaka/netshoot nc -lkv 0.0.0.0 8000
```

👉 Le port `8000` est disponible directement sur l’hôte.\
Depuis un autre terminal :

```bash
nc localhost 8000
```

Tout ce que tu tapes ici apparaît dans le terminal du conteneur.

***

#### 2. Accéder à un service de l’hôte depuis le conteneur

Démarre un conteneur en mode host :

```bash
docker run --rm -it --net=host nicolaka/netshoot
```

Puis, depuis le conteneur, tu peux accéder au serveur web de l’hôte (port `80`) :

```bash
nc localhost 80
```

***

## 🚫 Limitations du mode host

* Les processus dans le conteneur **ne peuvent pas se binder aux IP spécifiques de l’hôte** (pas d’accès direct aux interfaces).
* Sur Docker Desktop :
  * Fonctionne uniquement en **couche 4 (TCP/UDP)** → pas de protocoles en dessous (comme ICMP brut).
  * **Incompatible avec Enhanced Container Isolation** (contradiction entre isolation et partage du réseau).
* **Pas de support pour Windows containers** → uniquement Linux containers.

***

✅ En résumé :

* Le mode **host** supprime l’isolation réseau → le conteneur **utilise le réseau de l’hôte directement**.
* Très performant, mais moins sécurisé car pas de NAT.
* Utile pour les applications réseau intensives, ou quand il faut écouter un grand nombre de ports.
* Mais attention : pas de multi-conteneurs sur le même port, pas d’isolation stricte.
