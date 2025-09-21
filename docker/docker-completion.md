# ⌨️ Docker Completion

La complétion permet d’utiliser la touche **`Tab`** dans ton terminal pour compléter automatiquement :

* les **commandes Docker** (`docker run`, `docker ps`, etc.),
* les **options/flags** (`--name`, `--volume`, etc.),
* les **noms d’objets** Docker (conteneurs, volumes, images...).

C’est très pratique pour gagner du temps et éviter les erreurs de frappe.

***

### 🔹 1. Générer le script de complétion

La commande de base est :

```bash
docker completion [SHELL]
```

Où `[SHELL]` peut être :

* `bash`
* `zsh`
* `fish`

***

### 🔹 2. Exemple pour **Bash**

```bash
docker completion bash > ~/.docker-completion.sh
echo "source ~/.docker-completion.sh" >> ~/.bashrc
source ~/.bashrc
```

👉 Après ça, dans ton terminal Bash, tu peux écrire :

```bash
docker ru<Tab>
```

➡️ et ça va se compléter en `docker run`.

***

### 🔹 3. Exemple pour **Zsh**

```bash
docker completion zsh > "${fpath[1]}/_docker"
```

Puis recharge ta configuration :

```bash
exec zsh
```

***

### 🔹 4. Exemple pour **Fish**

```bash
docker completion fish | source
docker completion fish > ~/.config/fish/completions/docker.fish
```

***

✅ **En résumé :**

* `docker completion [bash|zsh|fish]` → génère le script.
* Il faut l’**ajouter au fichier de configuration de ton shell** (`.bashrc`, `.zshrc`, etc.).
* Une fois activée, la complétion t’assiste pour toutes les commandes et objets Docker.

## 🐳 **Activer la complétion Docker avec Bash**

### 🔹 1. Installer le paquet `bash-completion`

En fonction de ton système d’exploitation :

#### 📌 Sur **Debian/Ubuntu**

```bash
sudo apt update
sudo apt install bash-completion
```

#### 📌 Sur **Arch Linux**

```bash
sudo pacman -S bash-completion
```

#### 📌 Sur **macOS avec Homebrew**

* Pour **Bash ≥ 4** :

```bash
brew install bash-completion@2
```

* Pour les versions plus anciennes de Bash :

```bash
brew install bash-completion
```

***

### 🔹 2. Activer `bash-completion` dans ton shell

#### 📌 Sur **Linux**

Ajoute ce bloc dans ton `~/.bashrc` :

```bash
if [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
fi
```

#### 📌 Sur **macOS (Homebrew)**

Ajoute ce bloc dans ton `~/.bash_profile` :

```bash
[[ -r "$(brew --prefix)/etc/profile.d/bash_completion.sh" ]] && . "$(brew --prefix)/etc/profile.d/bash_completion.sh"
```

Recharge ta configuration :

```bash
source ~/.bashrc   # ou ~/.bash_profile
```

***

### 🔹 3. Générer le script de complétion Docker

Ensuite, installe la complétion spécifique pour Docker :

```bash
mkdir -p ~/.local/share/bash-completion/completions
docker completion bash > ~/.local/share/bash-completion/completions/docker
```

***

### 🔹 4. Tester la complétion

Ferme puis rouvre ton terminal (ou recharge avec `source ~/.bashrc`).\
Maintenant, tape :

```bash
docker ru<Tab>
```

➡️ Ça se complète automatiquement en `docker run`.\
Tu auras aussi la complétion pour les **options** et les **conteneurs/images existants**.

## 🐳 **Activer la complétion Docker avec Zsh**

### 🔹 1. Avec **Oh My Zsh**

Si tu utilises **Oh My Zsh**, il suffit de stocker la complétion dans le bon répertoire sans toucher au `~/.zshrc` :

```bash
mkdir -p ~/.oh-my-zsh/completions
docker completion zsh > ~/.oh-my-zsh/completions/_docker
```

➡️ Redémarre ton terminal et la complétion Docker fonctionnera directement 🎉.

***

### 🔹 2. Sans Oh My Zsh

Si tu n’utilises pas **Oh My Zsh**, crée un dossier dédié aux complétions et ajoute-le à ton `FPATH` :

```bash
mkdir -p ~/.docker/completions
docker completion zsh > ~/.docker/completions/_docker
```

Puis ajoute ceci à la fin de ton `~/.zshrc` :

```zsh
FPATH="$HOME/.docker/completions:$FPATH"
autoload -Uz compinit
compinit
```

Recharge ta configuration :

```bash
source ~/.zshrc
```

***

### 🔹 3. Tester

Une fois activé, tape par exemple :

```bash
docker im<Tab>
```

➡️ Ça se complète automatiquement en `docker images`.\
Tu auras aussi l’auto-complétion pour toutes les **commandes**, **options** et même pour les **conteneurs/images existants** 🚀.

## 🐳 Activer la complétion Docker avec Fish

Le shell **fish** prend en charge la complétion **nativement**.\
Pour activer la complétion de Docker, il suffit d’ajouter le script de complétion dans le répertoire dédié.

***

### 🔹 Étapes à suivre

1. Crée le dossier des complétions s’il n’existe pas déjà :

```bash
mkdir -p ~/.config/fish/completions
```

2. Génère le script de complétion Docker et place-le dans ce dossier :

```bash
docker completion fish > ~/.config/fish/completions/docker.fish
```

3. Recharge Fish (ou ouvre un nouveau terminal) :

```bash
exec fish
```

***

### 🔹 Vérification

Tape une commande Docker et appuie sur **Tab** :

```fish
docker ru<Tab>
```

➡️ Tu verras les suggestions comme `docker run`, `docker rmi`, etc.
