# glpi_lab
# GLPI HESTIM — Déploiement Docker Compose

> Déploiement et personnalisation d'une instance **GLPI 11** avec **MariaDB**, **Docker Compose** et des volumes persistants.

## 📌 Présentation

Ce projet consiste à déployer une instance de **GLPI** dans un environnement conteneurisé avec Docker Compose.

L'objectif est de mettre en place une infrastructure simple et reproductible permettant de :

* Déployer GLPI avec Docker
* Utiliser MariaDB comme base de données
* Gérer la persistance des données avec des volumes Docker
* Configurer la communication entre les conteneurs
* Administrer un parc informatique
* Créer et gérer des entités et équipements
* Personnaliser l'interface GLPI
* Comprendre le principe d'Infrastructure as Code avec Docker Compose

---

## 🏗️ Architecture

```text
                    ┌──────────────────────┐
                    │      Navigateur      │
                    │   localhost:8080     │
                    └──────────┬───────────┘
                               │
                               │ HTTP
                               ▼
                    ┌──────────────────────┐
                    │      GLPI 11.0       │
                    │        :80           │
                    │                      │
                    │ glpi-hestim-glpi-1   │
                    └──────────┬───────────┘
                               │
                               │ MariaDB :3306
                               ▼
                    ┌──────────────────────┐
                    │     MariaDB 11.4     │
                    │                      │
                    │ glpi-hestim-mariadb-1│
                    └──────────────────────┘
```

Les deux services communiquent sur un réseau Docker privé :

```text
glpi-net
```

---

## 🛠️ Technologies utilisées

| Technologie    | Utilisation                    |
| -------------- | ------------------------------ |
| Docker         | Conteneurisation               |
| Docker Compose | Orchestration des services     |
| GLPI 11.0      | Gestion de parc et helpdesk    |
| MariaDB 11.4   | Base de données                |
| YAML           | Définition de l'infrastructure |
| Ubuntu         | Système d'exploitation         |

---

## 📂 Structure du projet

```text
glpi-hestim/
├── compose.yaml
├── branding/
│   └── logo.png
└── README.md
```

### `compose.yaml`

Décrit l'ensemble de l'infrastructure :

* Service GLPI
* Service MariaDB
* Volumes
* Réseau Docker
* Variables d'environnement
* Ports
* Dépendances entre services

### `branding/`

Contient les ressources utilisées pour personnaliser GLPI, notamment le logo.

---

## 🚀 Installation

### Prérequis

* Docker
* Docker Compose
* Ubuntu ou autre distribution Linux compatible

Vérifier l'installation :

```bash
docker --version
docker compose version
```

---

## ▶️ Démarrage

Se placer dans le répertoire du projet :

```bash
cd glpi-hestim
```

Démarrer les services :

```bash
docker compose up -d
```

Vérifier leur état :

```bash
docker compose ps
```

GLPI est accessible depuis :

```text
http://localhost:8080
```

---

## 🗄️ Base de données

La base utilisée par GLPI est :

```text
glpi
```

Configuration :

```text
Host     : mariadb
Port     : 3306
Database : glpi
```

### Accéder à MariaDB

```bash
docker compose exec mariadb mariadb -u glpi -p glpi
```

Puis :

```sql
USE glpi;
```

Pour compter les tables :

```sql
SELECT COUNT(*)
FROM information_schema.tables
WHERE table_schema = 'glpi';
```

Lors de la réalisation du projet, la base contenait :

```text
442 tables
```

Ces tables sont créées automatiquement par GLPI lors de l'installation et de l'initialisation de la base de données.

---

## 💾 Persistance des données

Deux volumes Docker sont utilisés :

```yaml
volumes:
  glpi-db:
  glpi-data:
```

### Volume MariaDB

```yaml
- glpi-db:/var/lib/mysql
```

Il permet de conserver les données de la base MariaDB.

### Volume GLPI

```yaml
- glpi-data:/var/glpi
```

Il permet de conserver les données nécessaires à GLPI.

Lister les volumes :

```bash
docker volume ls
```

---

## 🌐 Réseau Docker

Les services sont connectés au réseau :

```yaml
glpi-net:
```

Le conteneur GLPI communique avec MariaDB en utilisant le nom du service :

```text
mariadb
```

Il n'est donc pas nécessaire d'utiliser directement l'adresse IP du conteneur MariaDB.

---

## 🔌 Mapping des ports

GLPI écoute sur le port `80` à l'intérieur de son conteneur.

Le fichier `compose.yaml` utilise :

```yaml
ports:
  - "8080:80"
```

Ce mapping signifie :

```text
Machine hôte                Conteneur
localhost:8080  ─────────►  GLPI:80
```

GLPI est donc accessible avec :

```text
http://localhost:8080
```

---

## 🖥️ Gestion du parc

Une première machine a été ajoutée dans GLPI afin de tester la gestion de parc.

Exemple :

```text
Nom              : PC-SUPERAMA-01
Numéro de série  : SN-2026-001
```

Cette opération permet de comprendre le fonctionnement de l'inventaire matériel dans GLPI.

---

## 🏢 Gestion des entités

L'entité racine de GLPI a été renommée :

```text
HESTIM
```

Une entité représente une organisation, un site ou un service.

Dans ce projet, l'entité `HESTIM` représente l'établissement.

---

## 🎨 Personnalisation de GLPI

### Palette utilisateur

La palette de couleurs peut être modifiée depuis :

```text
Avatar
→ Mes préférences
→ Personnalisation
→ Palette de couleurs
```

Cette modification est propre à l'utilisateur connecté.

### Branding

Un dossier local est monté dans le conteneur GLPI :

```yaml
- ./branding:/var/www/glpi/public/pics/branding:ro
```

Le suffixe `:ro` signifie **read-only**.

Le dossier local :

```text
./branding
```

est accessible dans le conteneur à :

```text
/var/www/glpi/public/pics/branding
```

Le logo peut être testé avec :

```text
http://localhost:8080/pics/branding/logo.png
```

---

## 🔄 Mise à jour de l'infrastructure

Le projet utilise une approche **Infrastructure as Code**.

Après modification de `compose.yaml`, il suffit d'exécuter :

```bash
docker compose up -d
```

Docker Compose compare la configuration actuelle avec la nouvelle définition et recrée uniquement les services nécessitant une modification.

Par exemple, après l'ajout du montage `branding`, le conteneur GLPI est recréé tandis que MariaDB reste en fonctionnement.

---

## 🔍 Commandes Docker utiles

### Voir les conteneurs

```bash
docker compose ps
```

### Voir les logs

```bash
docker compose logs
```

### Logs de GLPI

```bash
docker compose logs glpi
```

### Logs de MariaDB

```bash
docker compose logs mariadb
```

### Arrêter les services

```bash
docker compose stop
```

### Redémarrer les services

```bash
docker compose restart
```

### Arrêter et supprimer les conteneurs

```bash
docker compose down
```

> ⚠️ `docker compose down -v` supprime également les volumes et peut entraîner la perte des données persistantes.

---

## 🧠 Notions DevOps mises en pratique

Ce projet permet de pratiquer plusieurs concepts fondamentaux :

* Conteneurisation
* Docker Compose
* Infrastructure as Code
* Volumes persistants
* Bind mounts
* Réseaux Docker
* Variables d'environnement
* Port mapping
* Healthchecks
* Gestion du cycle de vie des conteneurs
* Recréation sélective des services
* Séparation application / base de données

---

## 📚 Objectifs pédagogiques

À travers ce projet, les objectifs sont de comprendre :

1. Comment déployer une application avec Docker.
2. Comment plusieurs conteneurs communiquent entre eux.
3. Comment assurer la persistance des données.
4. Comment modifier une infrastructure à partir d'un fichier Compose.
5. Comment Docker Compose détecte les changements de configuration.
6. Comment GLPI permet de gérer un parc informatique.
7. Comment personnaliser une application conteneurisée.

---

## 👤 Auteur

**Superrama**

Projet réalisé dans le cadre d'un apprentissage pratique de :

**Docker · DevOps · Administration système · Gestion de parc informatique**

---

## 📄 Licence

Projet réalisé à des fins pédagogiques.
