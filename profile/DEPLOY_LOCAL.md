# Lancer le projet en local

Instructions reproductibles pour faire tourner le projet en local : la DB en Docker, l'app Android, et le firmware sur une carte ESP32.

**Table des matières**
- [Firmware](#firmware)
- [Backend](#backend)
- [App Android](#app-android)

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

L'app Android peut être installée directement via l'APK fourni.

### Installer l'APK
## App Android

L'app Android s'installe directement via l'APK fourni. L'installation se fait toujours sur le téléphone ; depuis un ordinateur, il faut d'abord transférer le fichier.

### Récupérer l'APK

Téléchargez la dernière release : [bracelet-connecte-0.3.2.apk](https://github.com/PDGGRP9/webapp-app-android/releases/download/0.3.2/bracelet-connecte-0.3.2.apk).

**Depuis le téléphone**
1. Sur le téléphone, ouvrir le fichier `.apk` (via la notification de téléchargement ou un gestionnaire de fichiers).
2. À la première installation, Android bloque l'APK et propose d'autoriser la source : suivre l'invite, activer l'autorisation, puis revenir en arrière.
   > La procédure d'autorisation peut varier selon le modèle de téléphone et d'Android
3. Confirmer l'installation, puis ouvrir l'application. 
4. Confirmer l'installation, puis ouvrir l'application.

**Depuis un ordinateur :** 
télécharger le fichier, puis le transférer sur le téléphone par l'un de ces moyens :
- Câble USB : brancher le téléphone, l'autoriser en tant que transfert de fichiers, puis copier l'APK (p. ex. dans `Download`).
- Cloud / e-mail : s'envoyer le fichier (Drive, e-mail, etc.) et l'ouvrir depuis le téléphone.

Puis reprendre la procédure vu ci-dessus (depuis le téléphone).

