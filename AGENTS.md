# AGENTS.md — GestSIS Dev Docker

Environnement de développement de GestSIS. Ce dépôt **n'est pas une application** : c'est un orchestrateur Docker Compose qui réunit plusieurs services, chacun étant un **sous-module git indépendant** (son propre dépôt sur `github.com/GestSIS/<service>`).

> Règle d'or : **toute commande applicative (tests, artisan, yarn, manage.py) s'exécute DANS le conteneur Docker correspondant**, jamais sur l'hôte. Les dépendances (vendor/, node_modules, .venv) ne sont installées qu'à l'intérieur des images.

## Services

| Service | Conteneur       | Service compose | Port | Stack                          | Guide du dépôt                                     |
| ------- | --------------- | --------------- | ---- | ------------------------------ | -------------------------------------------------- |
| APP     | `gestsis-app`   | `app`           | 8080 | Vue + Vite + Pinia + Bootstrap | [GestSIS_APP/AGENTS.md](GestSIS_APP/AGENTS.md)     |
| API     | `gestsis-api`   | `api`           | 8000 | Laravel + PHP                  | [GestSIS_API/AGENTS.md](GestSIS_API/AGENTS.md)     |
| Auth    | `gestsis-auth`  | `auth`          | 8001 | Laravel + JWT (RSA)            | [GestSIS_Auth/AGENTS.md](GestSIS_Auth/AGENTS.md)   |
| Alarm   | `gestsis-alarm` | `alarm`         | 8002 | Django + DRF, gestion via `uv` | [GestSIS_Alarm/AGENTS.md](GestSIS_Alarm/AGENTS.md) |
| Doc     | `gestsis-doc`   | `doc`           | 8081 | Retype                         | [GestSIS_Doc/README.md](GestSIS_Doc/README.md)     |
| DB      | `gestsis-db`    | `db`            | 3306 | MySQL (partagée)               | —                                                  |

Ce fichier ne fige volontairement **aucune version** (elles dérivent) : la source de vérité reste le `composer.json` / `pyproject.toml` / `package.json` de chaque sous-module. **Avant de coder dans un service, lis son `AGENTS.md` (ou son `README.md`)** — ils contiennent les conventions et versions spécifiques (ex. l'API suit les Laravel Boost Guidelines).

## Lancer les tests

**Toujours via Docker Compose.** Le service utilise le nom compose (`api`, `auth`, …), pas le nom du conteneur.

| Service | Commande                                                                |
| ------- | ----------------------------------------------------------------------- |
| API     | `docker compose exec api php artisan test`                              |
| Auth    | `docker compose exec auth php artisan test`                             |
| Alarm   | `docker compose exec alarm uv run manage.py test` (couverture minimale) |
| APP     | pas de suite de tests — lint : `docker compose exec app yarn lint`      |

Astuces pour les services Laravel (API/Auth) — PHPUnit :

```bash
# Sortie compacte
docker compose exec api php artisan test --compact

# Un seul fichier / un seul test
docker compose exec api php artisan test --compact tests/Feature/GradeTest.php
docker compose exec api php artisan test --compact --filter=testDestroyGradeReturnsErrorWhenLinkedToCours
```

Détails utiles côté API : les tests tournent en `APP_ENV=testing` (voir `phpunit.xml`), sur une base MySQL dédiée, et la plupart des feature tests passent l'en-tête `Sis-Key` (multi-tenant, voir ci-dessous). Lance le **minimum** de tests ciblés pendant le dev, puis la suite complète avant de finaliser.

## Travailler dans un service

Le code est monté en bind mount (`./GestSIS_X:/app`) avec hot reload — éditer un fichier sur l'hôte suffit, pas besoin de rebuild pour du code applicatif.

```bash
# Entrer dans un conteneur
docker compose exec api bash      # ou: auth bash, app sh, alarm sh

# Exécuter une commande ponctuelle
docker compose exec api php artisan route:list
docker compose exec app yarn lint
```

Rebuild **uniquement** si tu modifies un `Dockerfile` ou les dépendances système :

```bash
make rebuild-api   # rebuild-auth | rebuild-app | rebuild-alarm
make rebuild       # tout
```

Cycle de vie courant (voir `Makefile` pour la liste complète) : `make up`, `make down`, `make logs-api`, `make clean` (supprime volumes + vendor/node_modules).

## Premier démarrage

```bash
make init   # clone les sous-modules, génère les clés RSA, copie les .env.docker → .env
make up     # docker compose up
```

`init.sh` distribue la même paire de clés RSA à Auth (privée + publique), API et Alarm (publique) : Auth signe les JWT, API/Alarm les vérifient. Compte de test : `admin@gestsis.ch` / `apptest`.

## Base de données

- **Une seule** instance MySQL (`gestsis-db`, mot de passe `pwd` — dev uniquement), partagée par tous les services.
- Les bases sont créées au démarrage par `docker-compose/mysql/gestsis.sql` (`gestsis`, `gestsis_auth`, `gestsis_alarm`, `hs`, `test`).
- **L'API est multi-tenant** : elle sélectionne la base selon l'en-tête `Sis-Key` de la requête (middleware `DbSelector`). En dev, les bases tenant sont listées dans `DB_LISTE` (`hs,test`) du `.env.docker`. Les feature tests reproduisent cela via l'en-tête `Sis-Key`.
- Repartir d'une base propre : `docker compose down -v && docker compose up`.

## Contribuer (sous-modules)

Chaque service est un dépôt séparé. **Une modification de code se commit et se PR dans le dépôt du service** (`GestSIS/<service>`), pas ici — ce dépôt ne suit que des pointeurs de commit vers les sous-modules. Workflow détaillé (fork, rebase sur `upstream/main`, PR ciblée) dans [CONTRIBUTING.md](CONTRIBUTING.md).

Conventions : PHP (API, Auth) → PSR-12 ; Vue (APP) → config ESLint existante (`yarn lint`) ; Python (Alarm) → PEP 8 ; commits → sujet court à l'impératif.
