# PHP DevOps TP - Pipeline CI/CD

Application PHP de démonstration pour le cours DevSecOps.

---

## PARTIE 1 : Lancer l'Application en Local

### Étape 1.1 : Construire l'image Docker

```bash
docker build -t php-devops-tp -f Docker/Dockerfile .
```
> Cette commande construit une image Docker à partir du Dockerfile.

### Étape 1.2 : Lancer le conteneur

```bash
docker run -d -p 8080:80 -e APP_SECRET="MonSecretTest" --name php-devops-app php-devops-tp
```
> Lance le conteneur en arrière-plan (-d), mappe le port 8080 vers 80, et passe le secret en variable d'environnement.

### Étape 1.3 : Installer les dépendances

```bash
docker exec php-devops-app composer install --no-interaction
```
> Installe les dépendances PHP (Carbon, phpdotenv) dans le conteneur.

### Étape 1.4 : Tester l'application

Ouvrir dans le navigateur : **http://localhost:8080**

> Tu dois voir une image PNG avec "DEVOPS - [date]" et "Une superbe image (secret: MonSecretTest)".

### Commandes utiles

```bash
# Si le conteneur existe déjà, le supprimer et relancer
docker rm -f php-devops-app

# Voir les logs du conteneur
docker logs php-devops-app

# Entrer dans le conteneur
docker exec -it php-devops-app bash

# Arrêter le conteneur
docker stop php-devops-app
```

---

## PARTIE 2 : Configuration CI/CD

### Étape 2.1 : Créer le repo GitHub

1. Aller sur **github.com**
2. Cliquer sur **"+"** en haut à droite → **"New repository"**
3. Nom : `devsecops-php-ci-cd`
4. Visibilité : **Public** ou **Private**
5. Cliquer **"Create repository"**

### Étape 2.2 : Pusher le code sur GitHub

```bash
# Initialiser git (si pas déjà fait)
git init

# Ajouter le remote
git remote add origin https://github.com/TON_USERNAME/devsecops-php-ci-cd.git

# Ajouter tous les fichiers
git add .

# Créer le premier commit
git commit -m "Initial commit"

# Pusher sur GitHub
git push -u origin main
```

### Étape 2.3 : Connecter CircleCI au repo

1. Aller sur **https://circleci.com**
2. Cliquer **"Log In"** (en haut à droite)
3. Choisir **"Log in with GitHub"**
4. Autoriser CircleCI à accéder à ton compte GitHub
5. Tu arrives sur le dashboard CircleCI
6. Dans la liste des projets, trouver **devsecops-php-ci-cd**
7. Cliquer **"Set Up Project"**
8. Sélectionner **"Fastest: Use the .circleci/config.yml in my repo"**
9. Cliquer **"Set Up Project"**

> CircleCI va automatiquement détecter le fichier `.circleci/config.yml` et lancer le premier build.

### Étape 2.4 : Créer un Personal Access Token GitHub (pour GHCR)

1. Aller sur **GitHub**
2. Cliquer sur ta **photo de profil** (en haut à droite)
3. Cliquer **"Settings"**
4. Descendre tout en bas et cliquer **"Developer settings"** (menu à gauche)
5. Cliquer **"Personal access tokens"**
6. Cliquer **"Tokens (classic)"**
7. Cliquer **"Generate new token"** → **"Generate new token (classic)"**
8. Remplir :
   - **Note** : `circleci-ghcr`
   - **Expiration** : 90 days (ou plus)
9. Cocher les permissions :
   - [x] `write:packages`
   - [x] `read:packages`
10. Cliquer **"Generate token"**
11. **COPIER LE TOKEN IMMÉDIATEMENT** (il ne sera plus visible après !)

> Ce token permet à CircleCI de pusher les images Docker vers GitHub Container Registry (GHCR).

### Étape 2.5 : Ajouter les variables GHCR dans CircleCI

1. Aller sur **CircleCI** (https://app.circleci.com)
2. Cliquer sur ton projet **devsecops-php-ci-cd**
3. Cliquer **"Project Settings"** (icône engrenage en haut à droite)
4. Dans le menu à gauche, cliquer **"Environment Variables"**
5. Cliquer **"Add Environment Variable"**

**Ajouter ces 2 variables :**

| Name | Value |
|------|-------|
| `GHCR_USERNAME` | ton username GitHub (ex: `potichacha`) |
| `GHCR_PAT` | le token copié à l'étape 2.4 |

> Ces variables permettent au job `build-docker-image` de se connecter à GHCR.

---

## PARTIE 2.3 : Gestion des Secrets avec Infisical

### Étape 2.6 : Créer un compte Infisical

1. Aller sur **https://infisical.com**
2. Cliquer **"Get Started"** ou **"Sign Up"**
3. Créer un compte (avec GitHub ou email)
4. Remplir le formulaire :
   - **Your Name** : ton nom
   - **Organization Name** : `Ecole` (ou ce que tu veux)
5. Cliquer **"Sign Up"**

> Infisical est un gestionnaire de secrets qui permet de stocker les mots de passe, API keys, etc. de manière sécurisée.

### Étape 2.7 : Créer un projet Infisical

1. Une fois connecté, cliquer **"+ Add New Project"**
2. Remplir :
   - **Project Name** : `php-devops-tp`
   - **Project Type** : `Secrets Management` (déjà sélectionné)
3. Cliquer **"Create Project"**

### Étape 2.8 : Ajouter un secret

1. Dans ton projet, tu es sur la page **"Project Overview"**
2. Cliquer **"Add Secrets"** (au centre) ou **"+ Add Secret"** (en haut à droite)
3. Remplir :
   - **Key** : `APP_SECRET`
   - **Value** : `MonSecretInfisical123`
   - **Environments** : sélectionner **Production**
4. Cliquer **"Create Secret"**

> Ce secret sera utilisé par l'application PHP pour afficher le message sur l'image.

### Étape 2.9 : Créer un Service Token

1. Dans Infisical, cliquer sur **"Access Control"** (menu en haut)
2. Dans le menu à gauche, cliquer **"Service Tokens"**
3. Cliquer **"Create"**
4. Remplir :
   - **Service Token Name** : `circleci`
   - **Environment** : sélectionner **Production**
   - **Secrets Path** : `/` (laisser par défaut)
   - **Expiration** : `Never` ou `6 Months`
   - **Permissions** : `Read` (déjà coché)
5. Cliquer **"Create"**
6. **COPIER LE TOKEN IMMÉDIATEMENT** (il ne sera plus visible après !)

> Ce token permet à CircleCI (ou au conteneur Docker) de récupérer les secrets depuis Infisical.

### Étape 2.10 : Ajouter le token Infisical dans CircleCI

1. Retourner sur **CircleCI**
2. Aller dans **Project Settings** → **Environment Variables**
3. Cliquer **"Add Environment Variable"**
4. Ajouter :
   - **Name** : `INFISICAL_TOKEN`
   - **Value** : le token copié à l'étape 2.9
5. Cliquer **"Add Environment Variable"**

---

## Étape 2.11 : Pusher le code pour déclencher le pipeline

```bash
git add .
git commit -m "Add README and configure CI/CD pipeline"
git push origin main
```

> Cela déclenche automatiquement le pipeline CircleCI.

### Étape 2.12 : Vérifier que le pipeline fonctionne

1. Aller sur **CircleCI** → **Pipelines**
2. Tu dois voir le pipeline en cours d'exécution
3. Les jobs suivants doivent passer en vert :
   - `debug-info` ✅
   - `build-setup` ✅
   - `lint-phpcs` ✅
   - `security-check-dependencies` ✅
   - `test-phpunit` ✅
   - `build-docker-image` ✅

> Le job `hold` attend une approbation manuelle avant de déployer en production.

---

## Résumé des Variables CircleCI

| Variable | Description | Où la trouver |
|----------|-------------|---------------|
| `GHCR_USERNAME` | Username GitHub | Ton profil GitHub |
| `GHCR_PAT` | Personal Access Token GitHub | GitHub → Settings → Developer settings → Tokens |
| `INFISICAL_TOKEN` | Service Token Infisical | Infisical → Access Control → Service Tokens |

---

## Structure du Projet

```
php-devops-tp/
├── .circleci/
│   └── config.yml          # Configuration CircleCI (pipeline CI/CD)
├── Docker/
│   └── Dockerfile          # Image Docker de l'application
├── public/
│   ├── index.php           # Point d'entrée de l'application
│   ├── info.php            # Page phpinfo() pour debug
│   └── font/consolas.ttf   # Police pour générer les images
├── src/
│   └── ImageCreator.php    # Classe principale qui génère les images
├── tests/
│   ├── Unit/               # Tests unitaires
│   └── Feature/            # Tests fonctionnels
├── composer.json           # Dépendances PHP
├── composer.lock           # Versions exactes des dépendances
├── phpunit.xml             # Configuration PHPUnit
├── phpcs.xml               # Configuration PHP CodeSniffer
├── .env.example            # Template des variables d'environnement
└── README.md               # Ce fichier
```

---

## Description de l'Application

L'application génère une **image PNG dynamique** qui affiche :
- Le texte "DEVOPS" avec la date et l'heure
- Un message personnalisé
- Le secret `APP_SECRET` (si configuré)

**Technologies utilisées :**
- PHP 8.2
- Librairie GD (génération d'images)
- Carbon (manipulation de dates)
- phpdotenv (chargement des variables d'environnement)
