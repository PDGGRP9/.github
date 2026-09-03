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
