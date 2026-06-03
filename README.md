# GestSIS Dev Docker

Environnement de développement Docker pour le projet GestSIS - Architecture microservices avec Laravel, Vue.js, Django et MySQL.

## 🚀 Démarrage rapide

### Prérequis
- Docker et Docker Compose
- Git avec support SSH
- Make (optionnel mais recommandé)

### Installation

1. **Cloner le projet**
```bash
git clone git@github.com:GestSIS/GestSIS_dev_docker.git
cd GestSIS_dev_docker
```

2. **Initialiser l'environnement**
```bash
make init
# ou directement: ./init.sh
```

3. **Lancer les services**
```bash
make up
# ou: docker compose up
```

Le script d'initialisation va:
- 📦 Cloner tous les sous-modules (Auth, API, APP, Alarm, Doc)
- 🔑 Générer les clés RSA pour l'authentification
- ⚙️ Configurer automatiquement tous les services

---

## 🌐 Services disponibles

| Service   | URL                   | Description                        |
| --------- | --------------------- | ---------------------------------- |
| **APP**   | http://localhost:8080 | Interface Vue.js 3                 |
| **API**   | http://localhost:8000 | API principale Laravel             |
| **Auth**  | http://localhost:8001 | Service d'authentification Laravel |
| **Alarm** | http://localhost:8002 | Service d'alarmes Django           |
| **Doc**   | http://localhost:8081 | Documentation Retype               |

### Compte de test
- **Email:** test@gmail.com
- **Mot de passe:** apptest

---

## 📝 Commandes utiles (Makefile)

```bash
make help         # Afficher toutes les commandes
make up           # Démarrer tous les services
make down         # Arrêter tous les services
make restart      # Redémarrer les services
make logs         # Afficher tous les logs
make logs-api     # Logs d'un service spécifique
make pull         # Mettre à jour les sous-modules
make clean        # Nettoyer (volumes + node_modules/vendor)
make rebuild      # Rebuild les containers
```

---

## 🛠️ Configuration

### Structure du projet
```
GestSIS_dev_docker/
├── GestSIS_Auth/       # Service d'authentification (Laravel)
├── GestSIS_API/        # API principale (Laravel)
├── GestSIS_APP/        # Interface utilisateur (Vue.js)
├── GestSIS_Alarm/      # Service d'alarmes (Django)
├── GestSIS_Doc/        # Documentation (Retype)
├── docker-compose.yml  # Orchestration Docker
├── Makefile           # Commandes simplifiées
└── init.sh            # Script d'initialisation
```

## 🔧 Développement

### Hot reload
Tous les services supportent le hot reload. Modifiez le code et les changements seront automatiquement pris en compte.

### Accéder aux logs
```bash
# Tous les services
docker compose logs -f

# Un service spécifique
docker compose logs -f api
make logs-api
```

### Accéder à un container
```bash
docker compose exec api bash
docker compose exec alarm sh
```

### Rebuild après modifications
```bash
# Tous les containers
make rebuild

# Un service spécifique
docker compose up --build api
```

---

## ⚠️ WSL2 / Docker Desktop

**Important:** Sur Windows avec WSL2, assurez-vous que le projet est dans le système de fichiers Linux (`\\wsl$\DISTRO_NAME`) et **non** sur une partition Windows montée. Cela évite des problèmes de performance significatifs.

---

## 🐛 Dépannage

### Les sous-modules ne se clonent pas
```bash
git submodule update --init --recursive
```

### Erreur de permissions sur les clés RSA
```bash
rm -rf rsa_keys/
./init.sh
```

### Port déjà utilisé
Vérifiez qu'aucun service ne tourne sur les ports 8000-8002, 8080-8081.

### Base de données vide
```bash
docker compose down -v
docker compose up
```

---

## 📚 Documentation complète

La documentation complète est disponible sur http://localhost:8081 après avoir lancé les services.

---

## 🏗️ Architecture technique

- **Frontend:** Vue.js 3.5, Vite, Pinia, Bootstrap 5
- **Backend API:** Laravel 12, PHP 8.4
- **Auth:** Laravel 12, JWT avec RSA 4096
- **Alarm:** Django 5.2, Python 3.11+, DRF
- **Database:** MySQL 8.1 partagée
- **Orchestration:** Docker Compose

---

## Idées d'améliorations

- Génération des fichiers .env de config des projets Laravel à la volée.
- La config actuelle nécessite plusieurs databases et le fichier de config `docker-compose\mysql\gestsis.sql` est hardcodé.
