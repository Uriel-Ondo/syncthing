
# 🔄 Syncthing avec Docker (macOS & Ubuntu)

Configuration propre de **Syncthing via Docker Compose** pour synchroniser un dossier entre **macOS et Ubuntu** ou autre, en évitant les dossiers parasites.

---

## 🚨 Problème rencontré

Lorsque la **configuration Syncthing** et les **données à synchroniser** utilisent le **même volume**, Syncthing écrit ses fichiers internes (`config.xml`, `cert.pem`, etc.) directement dans le dossier partagé.

👉 Résultat :
- Apparition de dossiers parasites
- Mélange entre fichiers utilisateurs et fichiers internes
- Synchronisation désordonnée

---

## ✅ Solution

👉 **Séparer strictement les volumes Docker** :

- 📁 **Un volume pour les données synchronisées**
- ⚙️ **Un volume pour la configuration Syncthing**

Schéma logique :

```

Hôte
├── Partage/              → fichiers à synchroniser
└── syncthing-config/     → configuration interne Syncthing

```

Dans le conteneur :

```

/var/syncthing/data    → données
/var/syncthing/config  → configuration

````

---

## 🐳 Docker Compose – Ubuntu

```yaml
services:
  syncthing:
    image: syncthing/syncthing:latest
    container_name: syncthing
    ports:
      - 8384:8384        # Interface Web
      - 22000:22000/tcp  # Sync
      - 22000:22000/udp
      - 21027:21027/udp  # Découverte LAN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Africa/Dakar
    volumes:
      - /home/uriel/Partage:/var/syncthing/data
      - /home/uriel/syncthing-config:/var/syncthing/config
    restart: unless-stopped
````

---

## 🍎 Docker Compose – macOS

```yaml
services:
  syncthing:
    image: syncthing/syncthing:latest
    container_name: syncthing
    hostname: my-syncthing
    environment:
      - PUID=501        # UID macOS
      - PGID=20         # GID macOS (staff)
      - TZ=Africa/Dakar
    volumes:
      - /Users/uriel/Partage:/var/syncthing/data
      - /Users/uriel/syncthing-config:/var/syncthing/config
    ports:
      - 8384:8384
      - 22000:22000/tcp
      - 22000:22000/udp
      - 21027:21027/udp
    restart: unless-stopped
```

---

## 📁 Création des dossiers

### Ubuntu

```bash
mkdir -p ~/Partage
mkdir -p ~/syncthing-config
```

### macOS

```bash
mkdir -p /Users/uriel/Partage
mkdir -p /Users/uriel/syncthing-config
```

---

## ▶️ Lancer Syncthing

```bash
docker compose down
docker compose up -d
```

Accès à l’interface Web :

```
http://localhost:8384
```

---

## 🖥️ Configuration dans l’interface Syncthing (GUI)

### 💻 Sur macOS

* **Label** : `Partage`
* **Folder ID** : `laisser par defaut`
* **Chemin** :

  ```
  /var/syncthing/data
  ```
* Appareil partagé : **Ubuntu**

---

### 🖥️ Sur Ubuntu

* Accepter le dossier partagé
* **Folder ID** : `laisser par defaut` (⚠️ identique)
* **Chemin** :

  ```
  /var/syncthing/data
  ```

---

## ⚠️ Bonnes pratiques (IMPORTANT)

* ✅ Toujours utiliser `/var/syncthing/data` dans le GUI
* ❌ Ne jamais utiliser `/var/syncthing/config` comme dossier synchronisé
* ✅ Même `Folder ID` sur toutes les machines
* ❌ Ne jamais synchroniser la configuration Syncthing

---

## 🎯 Résultat final

* 📁 `Partage` contient **uniquement tes fichiers**
* ⚙️ `syncthing-config` contient la configuration interne
* 🔁 Synchronisation propre entre macOS et Ubuntu
* 🚫 Plus aucun dossier parasite

---

## 📌 Technologies utilisées

* Docker & Docker Compose
* Syncthing
* macOS
* Ubuntu Linux

---

## 🧑‍💻 Auteur

**Uriel Désiré**
🚀 DevOps / Systèmes & Réseaux
📦 Docker • 🔄 Syncthing • ☁️ Infrastructure

---

## 📄 Licence

MIT – libre d’utilisation et de modification
