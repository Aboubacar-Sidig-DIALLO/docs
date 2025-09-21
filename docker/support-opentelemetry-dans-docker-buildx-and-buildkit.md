# 📊 Support OpenTelemetry dans Docker Buildx & BuildKit

👉 **Buildx** et **BuildKit** supportent **OpenTelemetry** pour permettre la traçabilité des builds.\
Tu peux ainsi suivre :

* ⏱️ la durée des différentes étapes,
* 📦 les contextes et couches construites,
* 🕵️‍♂️ diagnostiquer plus facilement les lenteurs ou problèmes de build.

***

### 🐳 1. Démarrer Jaeger

Lance un conteneur Jaeger (outil de traçage OpenTelemetry) :

```bash
docker run -d --name jaeger \
  -p "6831:6831/udp" \
  -p "16686:16686" \
  --restart unless-stopped \
  jaegertracing/all-in-one
```

* `6831/udp` → port pour recevoir les traces.
* `16686` → interface web Jaeger.

🌍 Accès à Jaeger : [http://127.0.0.1:16686/](http://127.0.0.1:16686/)

***

### ⚙️ 2. Créer un builder avec Jaeger activé

On configure un **builder Buildx** pour envoyer ses traces vers Jaeger via la variable `JAEGER_TRACE`.

```bash
docker buildx create --use \
  --name mybuilder \
  --driver docker-container \
  --driver-opt "network=host" \
  --driver-opt "env.JAEGER_TRACE=localhost:6831"
```

* `--driver docker-container` → builder isolé dans un conteneur.
* `--driver-opt network=host` → permet la communication avec Jaeger local.
* `env.JAEGER_TRACE=localhost:6831` → envoie les traces au collecteur Jaeger.

***

### 🔍 3. Initialiser et inspecter le builder

```bash
docker buildx inspect --bootstrap
```

Cela démarre ton builder et initialise la connexion avec Jaeger.

***

### 📈 4. Visualiser les traces

Chaque commande `docker buildx` exécutée sera **tracée automatiquement**.\
👉 Tu pourras consulter les résultats sur l’interface Jaeger :

🌐 [http://127.0.0.1:16686/](http://127.0.0.1:16686/)

***

### 🎯 Cas d’usage

* ✅ **Optimiser tes builds** en identifiant les étapes lentes.
* ✅ **Surveiller les performances** des builds dans un CI/CD.
* ✅ **Intégrer Docker Build dans ton observability stack** (Grafana, Jaeger, OpenTelemetry).

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>
