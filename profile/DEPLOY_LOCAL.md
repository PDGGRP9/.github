# Lancer le projet en local

Instructions reproductibles pour faire tourner le projet en local : la DB en Docker, l'app Android, et le firmware sur une carte ESP32.

**Table des matières**
- [Firmware](#firmware)
- [App Android](#app-android)
- [Software (Frontend/Backend/DB)](#software-frontendbackenddb)

## Firmware

Le firmware se lance en le flashant sur une carte ESP32-S3. Si le bracelet est déja monté, il suffit de le connecter via le port usb-c.

### Prérequis

- [PlatformIO](https://platformio.org/) (extension VSCode ou CLI) doit être installé — voir la marche à suivre dans [DEPLOY_FEATURE.md](./DEPLOY_FEATURE.md#Environnement-de-développement-et-commandes-du-terminal-pour-le-firmware).
- Se trouver à la racine du repo [firmware](https://github.com/PDGGRP9/firmware) (présence du fichier `platformio.ini`).
- Une carte ESP32-S3 branchée en USB.
- Le fichier `include/secrets.h` (non commité) — voir [CONTRIBUTING.md](./CONTRIBUTING.md).

### Flasher la carte

```bash
pio run -e esp32-s3 -t upload   # build + flash
```

## App Android

L'app Android s'installe directement via l'APK fourni. L'installation se fait toujours sur le téléphone ; depuis un ordinateur, il faut d'abord transférer le fichier.

### Récupérer l'APK

Téléchargez la dernière release : [bracelet-connecte-0.3.2.apk](https://github.com/PDGGRP9/webapp-app-android/releases/download/0.3.2/bracelet-connecte-0.3.2.apk).

**Depuis le téléphone**
1. Sur le téléphone, ouvrir le fichier `.apk` (via la notification de téléchargement ou un gestionnaire de fichiers).
2. À la première installation, Android bloque l'APK et propose d'autoriser la source : suivre l'invite, activer l'autorisation, puis revenir en arrière.
   > La procédure d'autorisation peut varier selon le modèle de téléphone et d'Android
   > chercher un bouton qui dit "Installer quand même"
3. Confirmer l'installation, puis ouvrir l'application. 

**Depuis un ordinateur :** 
télécharger le fichier, puis le transférer sur le téléphone par l'un de ces moyens :
- Câble USB : brancher le téléphone, l'autoriser en tant que transfert de fichiers, puis copier l'APK (p. ex. dans `Download`).
- Cloud / e-mail / whats'app : s'envoyer le fichier (Drive, e-mail, etc.) et l'ouvrir depuis le téléphone.

Puis reprendre la procédure vu ci-dessus (depuis le téléphone).

## Software (Frontend/Backend/DB)

Cette partie explique comment faire tourner l'ensemble du projet (base de données, backend, frontend) sur votre machine, à l'aide de Docker. Il n'est pas nécessaire d'installer quelconque librairie Node, Python ou PostgreSQL.

### Prérequis

- [Docker](https://www.docker.com/) et Docker Compose installés.
- Les images `ghcr.io/pdggrp9/*` doivent être accessibles. Si elles sont privées, se logger une fois :
  ```bash
  echo <TOKEN_GITHUB> | docker login ghcr.io -u <user-github> --password-stdin
  ```
  (le token doit avoir le scope `read:packages`)

### Récupérer le repo

Le repository [infra-orchestrator](https://github.com/PDGGRP9/infra-orchestrator) orchestre tous les services (DB, backend, frontend, Caddy) à partir des images publiées sur `ghcr.io` : inutile de cloner les autres repos pour un simple lancement.

```bash
git clone https://github.com/PDGGRP9/infra-orchestrator.git
cd infra-orchestrator
```

### Configurer le `.env`

Le `docker-compose.yml` est celui de la **prod** (HTTPS servi par Caddy). Deux variables sont **obligatoires** — `SITE_DOMAIN` et `ACME_EMAIL` — et `docker compose up` échoue tant qu'elles ne sont pas définies. Pour un lancement local, on copie l'exemple et on pointe sur `localhost` :

```bash
cp .env.example .env
```

Puis éditer `.env`. Pour du local, seules ces deux lignes sont à adapter :

```bash
SITE_DOMAIN=localhost         # Caddy émet un certificat via sa CA locale, sans Let's Encrypt
ACME_EMAIL=dev@example.com    # non utilisé avec localhost, mais la variable doit exister
```

Toutes les autres variables (`POSTGRES_*`, `DJANGO_SECRET_KEY`, tags d'images, `DEBUG`…) ont une valeur par défaut qui convient en local : les laisser telles quelles, ou les surcharger dans `.env` au besoin. Le fichier `.env` est dans `.gitignore`, il n'est jamais commité.

### Lancer les services

```bash
docker compose up -d
```

Un `fake-emitter` envoie automatiquement des mesures factices toutes les 5 secondes, pour avoir des données à afficher sans bracelet physique. Le désactiver avec `docker compose up -d --scale fake-emitter=0`.

### Accéder aux services

Tout passe par Caddy en HTTPS : le frontend et le backend ne sont pas publiés sur des ports séparés.

| Service | URL |
|---|---|
| Application (frontend + API) | https://localhost |
| API backend | https://localhost/api/ |
| PostgreSQL | `localhost:5432` (user `bracelet` / mdp `bracelet` / db `bracelet_connecte`, ou les valeurs mises dans `.env`) |

> À la première visite, le navigateur affiche un avertissement de sécurité : le certificat vient de la CA locale de Caddy, pas d'une autorité publique. L'accepter pour continuer. Pour le supprimer, importer la CA racine de Caddy dans le trousseau du système :
> ```bash
> docker compose cp caddy:/data/caddy/pki/authorities/local/root.crt ./caddy-root.crt
> ```
