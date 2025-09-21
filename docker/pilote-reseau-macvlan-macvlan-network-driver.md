# 🌐 Pilote réseau Macvlan (Macvlan network driver)

### 🔹 Définition

Certaines applications, notamment **les applications legacy** ou celles qui **analysent le trafic réseau**, s’attendent à être directement connectées au réseau physique.

👉 Dans ces cas, on peut utiliser le **pilote Macvlan**, qui assigne **une adresse MAC unique** à chaque interface réseau virtuelle de conteneur.\
Ainsi, chaque conteneur apparaît comme une **machine physique distincte** directement branchée sur le réseau.

#### ✅ Avantages :

* Chaque conteneur possède sa propre **MAC** et sa propre **IP**, visibles sur le réseau local.
* Idéal pour les **applications qui doivent être sur le même LAN** que d’autres machines physiques.
* Utile si tu veux que les conteneurs **écoutent le trafic** comme un outil de monitoring.

#### ⚠️ Points d’attention :

* Risque d’**épuisement des IP** ou de **VLAN spread** (trop de MAC uniques sur le réseau).
* L’équipement réseau doit supporter le **mode promiscuité** (_promiscuous mode_) → plusieurs MAC sur la même interface physique.
* Si une **bridge network** ou une **overlay network** suffit, elles sont préférables sur le long terme (plus simple à gérer).

***

### ⚙️ Options du driver Macvlan

| Option         | Défaut   | Description                                                       |
| -------------- | -------- | ----------------------------------------------------------------- |
| `macvlan_mode` | `bridge` | Définit le mode Macvlan : `bridge`, `vepa`, `passthru`, `private` |
| `parent`       | _(vide)_ | Spécifie l’interface physique parente à utiliser                  |

***

### 🛠️ Création d’un réseau Macvlan

#### 1️⃣ Mode **bridge**

En **bridge mode**, le trafic passe directement par l’interface physique définie sur l’hôte.

Exemple simple :

```bash
docker network create -d macvlan \
  --subnet=172.16.86.0/24 \
  --gateway=172.16.86.1 \
  -o parent=eth0 pub_net
```

👉 Ici, les conteneurs du réseau `pub_net` auront chacun une IP du sous-réseau `172.16.86.0/24` et seront visibles sur le LAN.

***

#### 2️⃣ Exclure certaines adresses IP (`--aux-addresses`)

Si certaines IP sont déjà utilisées, on peut les **réserver** pour éviter les conflits :

```bash
docker network create -d macvlan \
  --subnet=192.168.32.0/24 \
  --ip-range=192.168.32.128/25 \
  --gateway=192.168.32.254 \
  --aux-address="my-router=192.168.32.129" \
  -o parent=eth0 macnet32
```

***

#### 3️⃣ Mode **802.1Q trunk bridge**

Si on définit une interface avec un **point** (ex: `eth0.50`), Docker crée automatiquement une **sous-interface VLAN**.

Exemple :

```bash
docker network create -d macvlan \
  --subnet=192.168.50.0/24 \
  --gateway=192.168.50.1 \
  -o parent=eth0.50 macvlan50
```

***

### 🔄 Alternative : utiliser **IPvlan** à la place

Au lieu de Macvlan (bridge L3), tu peux utiliser **IPvlan en mode L2**.

```bash
docker network create -d ipvlan \
  --subnet=192.168.210.0/24 \
  --subnet=192.168.212.0/24 \
  --gateway=192.168.210.254 \
  --gateway=192.168.212.254 \
  -o ipvlan_mode=l2 -o parent=eth0 ipvlan210
```

***

### 🌍 Support **IPv6**

Si ton démon Docker est configuré pour l’IPv6, tu peux créer un réseau **dual-stack IPv4/IPv6**.

Exemple :

```bash
docker network create -d macvlan \
  --subnet=192.168.216.0/24 --subnet=192.168.218.0/24 \
  --gateway=192.168.216.1 --gateway=192.168.218.1 \
  --subnet=2001:db8:abc8::/64 --gateway=2001:db8:abc8::10 \
  -o parent=eth0.218 \
  -o macvlan_mode=bridge macvlan216
```

***

### ✅ Résumé

* **Macvlan = chaque conteneur a sa propre MAC & IP → comme une machine physique sur le LAN.**
* Modes disponibles : `bridge` (le plus courant), `vepa`, `passthru`, `private`.
* Utile pour les applis qui doivent **écouter du trafic** ou apparaître comme **machines indépendantes**.
* ⚠️ Attention au **VLAN spread** et à l’**épuisement d’adresses**.
* Alternative moderne → **IPvlan** (moins de complexité, meilleure scalabilité).
