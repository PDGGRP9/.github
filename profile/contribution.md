# Collaboration de l'équipe et contribution

### Stratégie de branches

La branche `main` doit rester propre. Chaque nouvelle fonctionnalité passe la création d'une nouvelle branche avant Pull Request (PR). Toute modification passe donc par une PR et une validation minimale avant fusion. Les branches main ne peuvent uniquement être protégées si la visibilité du projet reste publique. Autrement, la version payante de Github est nécessaire. Nous avons donc pris la décision de toujours passer par une PR.

### CI/CD

La CI doit vérifier automatiquement un push sur `main` ou sur les pull requests.

- vérification du build;
- exécution des tests automatisés quand ils existent;
- validation manuelle du matériel pour ce qui ne peut pas être testé automatiquement (test matériel)
- lors d'un tag v* (convention de nomage des tags : `Majeur`.`mineur`.`patch`, exemple : 3.5.6), une release automatique est faite
  - docker: lors du CD, une image du docker est push sur la gitlab docker registry.
  - firmware : lors du CI une release est faite avec le binaire du firmware (CD fait à la main cf. chapitre processus de développement/firmware)

### Procésus de développement

#### software (DB, app android, webapp)
Mode agile avec tableau Kanban à la racine de l'organisation. Création d'issues dans les repos qu'on associe à des Merge Requests et qu'on ajoute dans le tableau Kanban avec les différentes étapes : `Backlog`/`Ready`/`In Progress`/`In Review`/`Done`.

#### firmware
Processus de développement waterfall (planifié) pour les choix hardware
Processus agile pour le développement du firmware et utilisation du Kanban présenté au chapitre ci-dessus.
