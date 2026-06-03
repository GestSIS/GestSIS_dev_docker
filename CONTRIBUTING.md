# Contribuer à GestSIS

Merci de l'intérêt que tu portes au projet GestSIS !

## Environnement de développement

Ce dépôt contient l'environnement de développement Docker complet.
Suis le [README](README.md) pour démarrer — `make init && make up` suffit.

## Processus de contribution

Chaque service est un dépôt indépendant (sous-module git). L'environnement de dev se clone via ce dépôt, mais les contributions se font service par service.

### 1. Mettre en place l'environnement local

Si ce n'est pas déjà fait, clone ce dépôt et initialise l'environnement :

```bash
git clone git@github.com:GestSIS/GestSIS_Dev_docker.git
cd GestSIS_Dev_docker
make init && make up
```

### 2. Préparer son fork

Fork le dépôt du service visé sur GitHub (ex. `GestSIS/GestSIS_API`), puis redirige le sous-module local vers ton fork :

```bash
cd GestSIS_API  # ou le service concerné
git remote rename origin upstream
git remote add origin git@github.com:TON_USERNAME/GestSIS_API.git
```

### 3. Se synchroniser avec upstream

Avant de créer ta branche, assure-toi d'être à jour avec `main` :

```bash
git fetch upstream
git rebase upstream/main
```

### 4. Développer

```bash
git checkout -b feat/ma-fonctionnalite
# ... modifications et tests ...
```

### 5. Pousser et ouvrir une pull request

```bash
git push origin feat/ma-fonctionnalite
```

Ouvre ensuite une PR sur GitHub depuis ta branche vers `main` du dépôt upstream (`GestSIS/<service>`).

Garde les pull requests focalisées sur un seul changement. Pour plusieurs correctifs indépendants, ouvre des PRs séparées.

### Se re-synchroniser en cours de développement

Si `main` a avancé pendant que tu travaillais :

```bash
git fetch upstream
git rebase upstream/main
```

Si le rebase génère des conflits, résous-les fichier par fichier puis continue :

```bash
# Pour chaque conflit résolu :
git add fichier-en-conflit
git rebase --continue
```

Le rebase est recommandé au lieu du merge (`git merge upstream/main`) car il rejoue tes commits par-dessus `main` sans créer de commit de fusion parasite. L'historique reste linéaire et plus lisible pour les reviewers.

Une fois le rebase terminé, force-push ta branche :

```bash
git push origin feat/ma-fonctionnalite --force-with-lease
```

> `--force-with-lease` est plus sûr que `--force` : il échoue si quelqu'un d'autre a poussé sur ta branche entre-temps.

## Contribuer à GestSIS_Dev_docker

Pour des modifications sur l'environnement de dev lui-même (docker-compose, Makefile, init.sh), le processus est identique mais sans manipulation de sous-module — fork directement ce dépôt et suis les mêmes étapes.

## Modifier un Dockerfile

Si tu modifies un `Dockerfile`, il faut rebuilder l'image concernée :

```bash
# Rebuilder un seul service
make rebuild-api    # ou rebuild-auth, rebuild-app, rebuild-alarm

# Rebuilder tous les services
make rebuild
```

Si tu veux repartir d'un environnement totalement propre (volumes, node_modules, vendor) :

```bash
make clean && make up
```

## Lancer les tests

| Service       | Commande                                    |
| ------------- | ------------------------------------------- |
| GestSIS_API   | `docker compose exec api php artisan test`  |
| GestSIS_Auth  | `docker compose exec auth php artisan test` |
| GestSIS_APP   | pas de tests automatisés pour l'instant     |
| GestSIS_Alarm | pas de tests automatisés pour l'instant     |

## Conventions

- **PHP (API, Auth) :** PSR-12
- **Vue.js (APP) :** respecter la config ESLint existante
- **Python (Alarm) :** PEP 8
- **Messages de commit :** sujet court à l'impératif, ex. `Corrige la requête grade sapeur`

## Signaler un bug

Ouvre une issue sur le dépôt concerné en précisant les étapes pour reproduire le problème, le comportement attendu vs observé.
