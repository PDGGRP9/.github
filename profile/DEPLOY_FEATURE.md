# Déployer une nouvelle fonctionnalité
 
Procédure reproductible pour intégrer et déployer une fonctionnalité via la pipeline CI/CD.

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
| `hardware-test` | Flashe et teste sur l'ESP32-S3 physique | après `build` |
| `release` | Publie `firmware.bin` en release | tag `v*` |
  
### Déployer une fonctionnalité
 
1. Développer sur une branche partant de `main`.
2. Vérifier en local avant de pousser :
```bash
   pio test -e native      # tests logiques
   pio run -e esp32-s3     # firmware
   pio run -e esp32-s3-ci  # build sans capteurs
```
   Puis un test manuel sur carte (voir [CONTRIBUTING.md](./CONTRIBUTING.md)).
3. Ouvrir une PR vers `main` — merge possible seulement si la CI est verte.
4. Merger : `test`, `build` et `hardware-test` rejouent, cette fois avec le test sur carte physique.
5. Publier une version (optionnel) :
```bash
   git tag v1.2.0 && git push origin v1.2.0
```
   Le job `release` produit une release GitHub avec `firmware.bin`.
6. Flasher les cartes manuellement depuis le binaire de la release.

