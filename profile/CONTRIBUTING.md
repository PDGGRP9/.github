# Collaboration de l'équipe et contribution

Merci de l'intérêt porté à ce bracelet connecté open source ! Ce document explique
comment l'équipe travaille et comment proposer une modification, quelle que soit sa
taille : correction de bug, nouvelle fonctionnalité, ou amélioration de la
documentation.

Le projet est découpé en plusieurs dépôts (firmware, hardware, backend, frontend web,
app Android...). Les règles ci-dessous s'appliquent à l'ensemble d'entre eux, sauf
mention contraire dans un dépôt spécifique.

---

## Avant de contribuer

- **Repérez le bon dépôt.** Chaque partie du projet a son propre repo (voir la liste
  dans le [README](README.md) de l'organisation). Une contribution touchant l'app
  Android doit être ouverte sur `webapp-app-android`, pas sur le repo général.
- **Vérifiez les issues existantes** avant d'en ouvrir une nouvelle, pour éviter les
  doublons.
- **Pour un changement important** (nouvelle fonctionnalité, refonte, choix
  d'architecture), ouvrez une issue de discussion *avant* de coder, afin de valider
  l'approche avec l'équipe.
- **Pour une correction mineure** (typo, petit bug), la création d'une Pull Request 
est nécessaire.

## Stratégie de branches

La branche `main` doit rester propre. Chaque nouvelle fonctionnalité passe par la
création d'une nouvelle branche avant Pull Request (PR). Toute modification passe donc
par une PR revue par un tierce et une validation minimale avant fusion.

Les branches `main` ne peuvent être protégées nativement que si le dépôt est privé —
sur un dépôt public, cette protection nécessite une offre payante. Nous avons donc pris
la décision de **toujours passer par une PR**, revue par au moins une autre personne,
même pour un petit changement.

## CI/CD

La CI vérifie automatiquement chaque push sur `main` ainsi que chaque Pull Request :

- vérification du build ;
- exécution des tests automatisés quand ils existent ;
- validation manuelle du matériel pour ce qui ne peut pas être testé automatiquement
  (test matériel).

Lors d'un tag `v*` (convention de nommage des tags : `Majeur.mineur.patch`, exemple
`3.5.6`), une release automatique est faite :

- **Docker** : lors du CD, une image Docker est poussée sur le registre.
- **Firmware** : lors du CI, une release est faite avec le binaire du firmware (le CD,
  c'est-à-dire le déploiement effectif sur le matériel, se fait à la main. Voir le
  chapitre *Processus de développement / firmware* ci-dessous).

## Processus de développement

### Software (DB, app Android, webapp)

Mode agile avec un tableau **Kanban** à la racine de l'organisation. Chaque nouvelle
tâche part de la création d'une issue dans le repo concerné, associée à une Merge
Request, et suivie sur le Kanban à travers les étapes :
`Backlog` → `Ready` → `In Progress` → `In Review` → `Done`.

### Firmware

- Processus de développement **waterfall** (planifié) pour les choix hardware.
- Processus **agile** pour le développement du firmware, avec le même Kanban que pour
  le software.

## Style de code

- Respectez le style déjà en place dans le fichier que vous modifiez (indentation,
  conventions de nommage, organisation des imports).
- Commentez les parties non triviales, en particulier le traitement du signal côté
  firmware (filtrage BPM/SpO2), qui n'est pas toujours intuitif à la relecture.
- Évitez de mélanger dans une même PR un reformatage massif et un changement
  fonctionnel : ça rend la revue illisible. Faites deux PR séparées.

## Signaler un bug ou proposer une idée

Ouvrez une issue sur le dépôt concerné avec :

- un titre clair et descriptif ;
- pour un bug : les étapes pour reproduire, le comportement attendu vs observé, et
  l'environnement (OS, version du firmware, navigateur, etc.) ;
- pour une idée : le problème que ça résout, pas seulement la solution envisagée.

## Contribuer sans coder

Toute aide est bienvenue, y compris :

- améliorer la documentation ou corriger des fautes ;
- tester le bracelet ou l'app et remonter des retours d'usage ;
- suggérer des designs pour la webapp ou l'app Android.

## Questions

Pour toute question, ouvrez une issue, ou contactez l'équipe :
R. Bouzourène, L. Haye, D. Kury, T. Nguyen.