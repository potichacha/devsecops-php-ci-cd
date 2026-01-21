# PHP DevOps TP - Pipeline CI/CD

Application PHP de démonstration pour le cours DevSecOps.

## Description

Application web PHP qui génère une image PNG dynamique affichant :
- Le texte "DEVOPS" avec la date/heure actuelle
- Un message personnalisé
- Un secret (si configuré via `APP_SECRET`)

## Structure du Projet

```
php-devops-tp/
├── .circleci/
│   └── config.yml          # Configuration CircleCI
├── Docker/
│   └── Dockerfile          # Image Docker
├── public/
│   ├── index.php           # Point d'entrée
│   └── font/consolas.ttf   # Police TTF
├── src/
│   └── ImageCreator.php    # Classe principale
├── tests/                  # Tests PHPUnit
├── composer.json           # Dépendances PHP
└── .env.example            # Template variables d'environnement
```

## Partie 1 : Lancer l'Application en Local

### Prérequis
- Docker installé

### Commandes

```bash
# 1. Construire l'image Docker
docker build -t php-devops-tp -f Docker/Dockerfile .

# 2. Lancer le conteneur (avec secret)
docker run -d -p 8080:80 -e APP_SECRET="MonSecretTest" --name php-devops-app php-devops-tp

# 3. Installer les dépendances Composer
docker exec php-devops-app composer install --no-interaction

# 4. Tester l'application
# Ouvrir http://localhost:8080 dans le navigateur
```

### Commandes utiles

```bash
# Arrêter le conteneur
docker stop php-devops-app

# Supprimer le conteneur
docker rm php-devops-app

# Supprimer et relancer
docker rm -f php-devops-app && docker run -d -p 8080:80 -e APP_SECRET="MonSecretTest" --name php-devops-app php-devops-tp

# Voir les logs
docker logs php-devops-app

# Entrer dans le conteneur
docker exec -it php-devops-app bash
```

## Partie 2 : Configuration CI/CD

### Variables CircleCI requises

| Variable | Description |
|----------|-------------|
| `GHCR_USERNAME` | Username GitHub |
| `GHCR_PAT` | Personal Access Token GitHub (avec permissions `write:packages`) |
| `STAGING_SSH_USER` | User SSH pour EC2 staging |
| `STAGING_SSH_HOST` | Host EC2 staging |
| `STAGING_SSH_FINGERPRINT` | Fingerprint clé SSH staging |
| `STAGING_DEPLOY_DIRECTORY` | Répertoire de déploiement staging |
| `PROD_SSH_USER` | User SSH pour EC2 production |
| `PROD_SSH_HOST` | Host EC2 production |
| `PROD_SSH_FINGERPRINT` | Fingerprint clé SSH production |
| `PROD_DEPLOY_DIRECTORY` | Répertoire de déploiement production |

### Workflows

- **main_workflow** : Build, lint, tests, sécurité, déploiement
- **container_workflow** : Build et push image Docker vers GHCR

## Partie 3 : Gestion des Secrets avec Infisical

```bash
# Dans le conteneur, exporter les secrets
infisical export --env=prod > .env

# Ou en variables d'environnement
eval $(infisical export --env=prod --format=shell)
```

## Technologies

- PHP 8.2
- Apache
- Docker
- CircleCI
- GitHub Container Registry (GHCR)
- Infisical (gestion des secrets)
