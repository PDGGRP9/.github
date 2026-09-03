# Déployer une nouvelle fonctionnalité
 
Procédure reproductible pour intégrer et déployer une fonctionnalité via la pipeline CI/CD.

Prendre d'habord des règles général [ici](https://github.com/PDGGRP9/.github/blob/main/profile/CONTRIBUTING.md) 

**Table des matières**
- [Firmware](#firmware)
- [Backend](#backend)
- [App Android](#app-android)

## Firmware

Le firmware ne fait pas de CD. On ne peut pas flasher à distance les ESP32 déjà en circulation.

La CI n'a accès qu'à une seule carte physique sans capteurs connecté (runner auto-hébergé). La pipeline garantit donc deux choses :
 
1. **Intégration continue** — lint, tests, compilation et test sur carte réelle.
2. **Release** — sur un tag `v*`, publication de `firmware.bin` en release GitHub.
Le flash des cartes reste manuel, hors pipeline.
 
### La pipeline
 
Déclenchée sur PR vers `main`, et sur les tags `v*`. Quatre jobs en chaîne :
 
| Job | Rôle | Déclencheur |
|-----|------|-------------|
| `test` | Lint (`cppcheck`) + tests natifs | push + PR |
| `build` | Compile `esp32-s3` et `esp32-s3-ci`, vérifie la taille, publie l'artefact | après `test` |
| `hardware-test` | Flashe et teste sur l'ESP32-S3 physique (local host) | après `build` |
| `release` | Publie `firmware.bin` en release | tag `v*` |

### Environnement de développement et commandes du terminal pour le firmware

- Installer l'extension PlatformIO sur vscode :  [PlatformIO](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide)
- ou chercher "PlatformIO IDE" dans les extensions

### Commande dans le terminal

```bash
pio test -e native        # pour les tests logique (fonctions, ...), se fait en local
pio run -e esp32-s3       # build le firmware (en local)
pio run -e esp32-s3 -t upload             # build et flash la carte
pio run -e esp32-s3 -t upload -t monitor  # flash + serial 115200
pio run -e esp32-s3 -t size               # taille flash/RAM
pio run -e esp32-s3-ci                    # build "sans capteurs"
pio device monitor --baud 115200          # voir les logs
pio check -e esp32-s3 --skip-packages     # cppcheck
pio run -t clean                          # vide .pio/build
pio device list
```

### Adaptation des capteurs
Il y a des environnement dans le fichier platformio.ini
- `esp32-s3` environnement de dev pour les capteurs, il faut les activer !
- `esp32-s3-ci` n'active presque rien -> il permet de tester le bluetouth quand même !
- `native` ne compile aucun code matériel

Pour prendre en compte un capteur, il faut faire #ifdef
```CPP
#ifdef HAS_IMU
  #include "imu.h"
  IMU imu;
#endif

void setup() {
#ifdef HAS_BLUETOOTH
    bluetooth_init();
#endif
#ifdef HAS_IMU
    imu.begin();
#endif
}

void loop() {
#ifdef HAS_IMU
    float accel = imu.readAccel();
#endif
}
```

### Ajouter une librairie

Exemple concret : ajouter l'oxymètre SEN0344 (lib `DFRobot_BloodOxygen_S`).

**1. Déclarer la lib — dans le bon env**

Dans `platformio.ini`, sous `[env:esp32-s3]`

```ini
[env:esp32-s3]
board = seeed_xiao_esp32s3
lib_deps =
    dfrobot/DFRobot_BloodOxygen_S
build_flags =
    -D ARDUINO_USB_CDC_ON_BOOT=1
    -D HAS_BLUETOOTH
    -D HAS_IMU
    -D HAS_OXYGEN      ; <- ton nouveau flag
```

Pourquoi pas sous `[env]` : `[env]` s'applique aussi à `native`, qui compile sur
PC.

**2. Protéger le code par un flag**

Tout ce qui touche le capteur va derrière `#ifdef` — sinon la CI (`esp32-s3-ci`,
sans capteurs) et le esp32 qui n'a pas le module ne compilent plus :

```cpp
#ifdef HAS_OXYGEN
  #include "sensors/oximeter.h"
  Oximeter oxy;
#endif

void setup() {
#ifdef HAS_OXYGEN
    oxy.begin();
#endif
}
```

**3. Vérifier avant de push — les 3 envs**

```bash
pio test -e native        # le code portable compile toujours
pio run -e esp32-s3-ci    # le build "sans capteurs" compile toujours
pio run -e esp32-s3       # ton build avec le capteur compile
```

Si un des trois casse, c'est que la lib ou le `#include` a fuité hors du `#ifdef`.

**Cas particulier : lib de logique pure (pas de matériel)**

Si le code n'utilise pas Arduino, ajoute aussi le fichier au filtre natif pour
qu'il soit testé par la CI :

```ini
[env:native]
build_src_filter = +<message.cpp> +<logic/mon_fichier.cpp>
```

### Secrets
Les logins sont dans le fichier `include/secrets.h`. Ne pas le commiter !

### workflow pour un push

```bash
pio test -e native      # 1. les tests logiques passent
pio run -e esp32-s3     # 2. le firmware compile bien
pio run -e esp32-s3-ci  # 3.le build "sans capteurs" compile toujours
<TEST MANUEL>           # 4. test sur carte de bout-en-bout
git push                # 5. la CI prend le relais (build + test sur carte)
```

### Environnement de dev sans capteurs

Utilise l'env esp32-s3-ci qui ne prends pas en compte les capteurs.

```
pio test -e native      # pour les tests logique (fonctions, ...), se fait en local
pio run -e esp32-s3-ci  # build le firmware (en local)
pio run -e esp32-s3-ci -t upload  # build et flash la carte
pio device monitor --baud 115200  # voir les logs
```

## App Android

Il n'y a pas de déploiement sur le Play Store. La distribution se fait en téléchargeant
l'APK depuis la release et en l'installant à la main sur le téléphone.

### Environnement de développement

- Android Studio, avec le SDK Android 34 et un JDK 17.
- `local.properties` est généré par Android Studio et contient le chemin du SDK.
- Pour tester : un téléphone physique dès que la fonctionnalité touche au Bluetooth,
  l'émulateur n'ayant pas de BLE. Sinon un émulateur API ≥ 26 suffit.

### Ajouter un appel API

1. Déclarer le DTO `@Serializable` dans `data/api/dto/`.
2. Ajouter la méthode dans `data/api/WebappApiService.kt`.
3. L'appeler depuis un repository (`data/measurements/`, `data/auth/`, …) et non depuis un
   composable.

Le header `Authorization: Bearer …` est déjà posé par `data/api/AuthInterceptor.kt`, inutile
de le remettre. L'URL de base ne se code pas en dur non plus : `ApiClientProvider` conserve
un client Retrofit par URL, et l'URL courante vient de la session.

### Ajouter une dépendance

Les versions sont centralisées dans `gradle/libs.versions.toml`. Déclarer la version et
l'alias là-bas, puis dans `app/build.gradle.kts` :

```kotlin
dependencies {
    implementation(libs.mon.alias)
}
```
Pas de coordonnées Maven écrites en dur dans `build.gradle.kts`. Attention si tu touches à
la version de Kotlin : celle de KSP doit la suivre (`1.9.24` ↔ `1.9.24-1.0.20`).

### Écrire un test (TODO : et donc ? rien de pratique ici !!)

1. **Sortir la logique du composable et du ViewModel**, dans une fonction ou un `object`
   sans dépendance Android : elle ne prend que des types Kotlin et `java.time`, et retourne
   un résultat. C'est ce qu'ont fait `presentation/stats/StatsMath.kt` pour les courbes et
   `domain/BraceletMeasurementCodec.kt` pour le décodage des trames.
2. **Créer le fichier de test** dans `app/src/test/java/com/pdg/braceletconnecte/`, en
   miroir du package testé, nommé `<ClasseTestée>Test.kt`.
3. **Écrire les cas** en JUnit 4 : une méthode par comportement, nommée en backticks pour
   qu'elle se lise en anglais dans le rapport.

### Ce que la pipeline ne vérifie pas

Le smoke-test confirme uniquement que l'app se lance sans crasher sur un émulateur. Le
Bluetooth, les permissions runtime et le rendu Compose sont à vérifier avant de faire une PR :

- scan et connexion au bracelet, puis reconnexion après avoir coupé le Bluetooth ;
- réception d'une mesure et affichage dans le Dashboard ;
- coupure réseau : les mesures doivent être mises en file dans Room, puis remontées au
  retour du réseau ;
- login, logout, accès invité ;
- Dashboard et Stats sur un jeu de données non vide.

### Commandes dans le terminal (todo : qu'est ce qui est vraiment utiliser sur la cicd ? a regrouper dans le workflow avant un push !)

Commandes lancées par le CI. Les rejouer en local avant de pousser est la seule façon de voir un lint ou un test rouge sans attendre le retour de la PR :

### Workflow pour une PR

La CI ne lance que trois commandes Gradle, dans cet ordre, et échoue à la première qui
casse. Les rejouer en local est la seule façon de voir un lint ou un test rouge sans
attendre le retour de la PR :

```bash
./gradlew lintDebug           # 1. Android Lint : une seule erreur suffit à casser le job
./gradlew testDebugUnitTest   # 2. tests unitaires JVM ; rapport dans app/build/reports/tests/
./gradlew assembleDebug       # 3. build de l'APK, dans un job séparé qui suit les deux autres
```

Le reste, la CI ne le vérifie pas. Le smoke-test du CD confirme seulement que l'app démarre,
donc le test manuel sur un vrai téléphone est à faire, d'autant que l'émulateur n'a pas de BLE :

```bash
adb devices                                  # le téléphone répond ? sinon débogage USB coupé
./gradlew installDebug                       # build + installe : la boucle la plus rapide pour itérer
adb logcat | grep com.pdg.braceletconnecte   # les logs de l'app, seul moyen de suivre BLE et réseau
```

### Publier une version
Selon les règles générales : [ici](https://github.com/PDGGRP9/.github/blob/main/profile/CONTRIBUTING.md) 

Une fois la PR mergée dans `main` :

```bash
git checkout main && git pull
git tag 0.3.3
git push origin 0.3.3
```
