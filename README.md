# PHP DevOps TP - Pipeline CI/CD

Application PHP de démonstration pour le cours DevSecOps.

**Projet GitHub** : https://github.com/potichacha/devsecops-php-ci-cd

---

# Table des matières

1. [Ce qui a été fait](#ce-qui-a-été-fait)
2. [Pour les collaborateurs](#pour-les-collaborateurs)
3. [Partie 1 : Application en local](#partie-1--lancer-lapplication-en-local)
4. [Partie 2 : Configuration CI/CD](#partie-2--configuration-cicd-fait-)
5. [Partie 3 : Extension du Pipeline](#partie-3--extension-du-pipeline)
6. [Partie 4 : Documentation et Rapport](#partie-4--documentation-et-rapport)
7. [Partie 5 : Sécurité](#partie-5--sécurité-à-faire)

---

# Ce qui a été fait

## Résumé des tâches accomplies

| Partie | Tâche | Status | Par qui |
|--------|-------|--------|---------|
| 1 | Application Docker fonctionnelle en local | Fait | Sacha |
| 2 | Repo GitHub créé | Fait | Sacha |
| 2 | CircleCI connecté au repo | Fait | Sacha |
| 2 | Variables GHCR configurées | Fait | Sacha |
| 2 | Infisical configuré | Fait | Sacha |
| 2 | Job deploy-ssh-production créé | Fait | Sacha |
| 3 | Job metrics-phpmetrics | Fait | Sacha |
| 3 | Job metrics-phploc | Fait | Sacha |
| 3 | Job lint-phpmd | Fait | Sacha |
| 3 | Job lint-php-doc-check | Fait | Sacha |
| 3 | Déploiement AWS EC2 | À faire | - |
| 4 | Rapport final | À faire | - |
| 5 | Sécurité GitHub | À faire | - |
| 5 | Sécurité Docker (scan Trivy) | À faire | - |
| 5 | Sécurité AWS EC2 | À faire | - |

---

# Pour les collaborateurs

## Comment rejoindre le projet

### 1. Cloner le repo

```bash
git clone https://github.com/potichacha/devsecops-php-ci-cd.git
cd devsecops-php-ci-cd
```

### 2. Accéder à CircleCI

1. Aller sur **https://circleci.com**
2. Se connecter avec **GitHub**
3. Le projet **devsecops-php-ci-cd** apparaît automatiquement
4. Vous pouvez voir les pipelines et leur statut

### 3. Accéder à Infisical (optionnel)

1. Demander une invitation à Sacha
2. Aller sur **https://infisical.com** et accepter l'invitation
3. Vous pourrez voir/modifier les secrets

### 4. Workflow de travail

```bash
# 1. Se mettre à jour
git pull origin main

# 2. Créer une branche pour vos modifications
git checkout -b feature/ma-modification

# 3. Faire vos modifications...

# 4. Commiter et pusher
git add .
git commit -m "Description de vos modifications"
git push origin feature/ma-modification

# 5. Créer une Pull Request sur GitHub (ou merger directement dans main)
```

---

# Partie 1 : Lancer l'Application en Local

## Étape 1.1 : Construire l'image Docker

```bash
docker build -t php-devops-tp -f Docker/Dockerfile .
```
> Cette commande construit une image Docker à partir du Dockerfile.

## Étape 1.2 : Lancer le conteneur

```bash
docker run -d -p 8080:80 -e APP_SECRET="MonSecretTest" --name php-devops-app php-devops-tp
```
> Lance le conteneur en arrière-plan (-d), mappe le port 8080 vers 80, et passe le secret en variable d'environnement.

## Étape 1.3 : Installer les dépendances

```bash
docker exec php-devops-app composer install --no-interaction
```
> Installe les dépendances PHP (Carbon, phpdotenv) dans le conteneur.

## Étape 1.4 : Tester l'application

Ouvrir dans le navigateur : **http://localhost:8080**

> Tu dois voir une image PNG avec "DEVOPS - [date]" et "Une superbe image (secret: MonSecretTest)".

## Commandes utiles

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

# Partie 2 : Configuration CI/CD (Fait)

## Ce qui a été configuré

### Variables CircleCI

| Variable | Description | Status |
|----------|-------------|--------|
| `GHCR_USERNAME` | Username GitHub |
| `GHCR_PAT` | Personal Access Token GitHub |
| `INFISICAL_TOKEN` | Service Token Infisical |

### Jobs existants dans le pipeline

| Job | Description | Status |
|-----|-------------|--------|
| `debug-info` | Affiche les infos de debug |
| `build-setup` | Installe les dépendances Composer |
| `lint-phpcs` | Vérifie le code avec PHP_CodeSniffer |
| `security-check-dependencies` | Scan des vulnérabilités |
| `test-phpunit` | Exécute les tests unitaires |
| `metrics-phpmetrics` | Génère rapport métriques HTML |
| `metrics-phploc` | Compte les lignes de code |
| `lint-phpmd` | PHP Mess Detector - qualité du code |
| `lint-php-doc-check` | Vérifie la documentation du code |
| `build-docker-image` | Build et push image vers GHCR |
| `deploy-ssh-production` | Déploiement SSH (pas encore configuré) | **A FAIRE** |

### Infisical

- **Projet** : php-devops-tp
- **Secret configuré** : `APP_SECRET` = `MonSecretInfisical123`
- **Service Token** : créé et ajouté dans CircleCI

---

# Partie 3 : Extension du Pipeline

## 3.1 Jobs d'évaluation de code (Fait)

### Job metrics-phpmetrics (Fait)

**Ce que ça fait** : Génère un rapport HTML avec des métriques de qualité du code (complexité, couplage, maintenabilité).

**Comment voir le rapport** :
1. Aller sur CircleCI → Pipeline → Job `metrics-phpmetrics`
2. Onglet **Artifacts**
3. Cliquer sur `phpmetrics-report/index.html`

**Code du job** (déjà dans `.circleci/config.yml`) :
```yaml
metrics-phpmetrics:
  executor: php-executor
  steps:
    - *attach_workspace
    - run:
        name: Install PHPMetrics
        command: composer require --dev phpmetrics/phpmetrics
    - run:
        name: Run PHPMetrics
        command: ./vendor/bin/phpmetrics --report-html=phpmetrics-report ./src
    - store_artifacts:
        path: phpmetrics-report
        destination: phpmetrics-report
```

### Job metrics-phploc (Fait)

**Ce que ça fait** : Compte les lignes de code, classes, méthodes, etc.

**Comment voir le rapport** :
1. Aller sur CircleCI → Pipeline → Job `metrics-phploc`
2. Onglet **Artifacts**
3. Télécharger `phploc-report.xml`

**Code du job** (déjà dans `.circleci/config.yml`) :
```yaml
metrics-phploc:
  executor: php-executor
  steps:
    - *attach_workspace
    - run:
        name: Install phploc
        command: |
          curl -L -o phploc.phar https://phar.phpunit.de/phploc.phar
          chmod +x phploc.phar
    - run:
        name: Run phploc
        command: php phploc.phar --log-xml=phploc-report.xml ./src || true
    - store_artifacts:
        path: phploc-report.xml
        destination: phploc-report
```

---

## 3.2 Jobs de qualité de code (Fait)

### Job lint-phpmd (Fait)

**Ce que ça fait** : PHP Mess Detector - détecte les problèmes de conception du code (code mort, complexité, etc.)

**Comment voir le rapport** :
1. Aller sur CircleCI -> Pipeline -> Job `lint-phpmd`
2. Onglet **Artifacts**
3. Cliquer sur `phpmd-report.html`

**Code du job** :

```yaml
  lint-phpmd:
    executor: php-executor
    steps:
      - *attach_workspace
      - run:
          name: Install PHPMD
          command: composer require --dev phpmd/phpmd
      - run:
          name: Run PHPMD
          command: |
            ./vendor/bin/phpmd ./src html cleancode,codesize,controversial,design,naming,unusedcode --report-file=phpmd-report.html || true
      - store_artifacts:
          path: phpmd-report.html
          destination: phpmd-report
```

### Job lint-php-doc-check (Fait)

**Ce que ça fait** : Vérifie que le code est bien documenté (PHPDoc).

**Comment voir le rapport** :
1. Aller sur CircleCI -> Pipeline -> Job `lint-php-doc-check`
2. Onglet **Artifacts**
3. Télécharger `php-doc-check-report.txt`

**Code du job** :

```yaml
  lint-php-doc-check:
    executor: php-executor
    steps:
      - *attach_workspace
      - run:
          name: Install php-doc-check
          command: composer require --dev niels-de-blaauw/php-doc-check
      - run:
          name: Run php-doc-check
          command: |
            ./vendor/bin/php-doc-check ./src > php-doc-check-report.txt || true
      - store_artifacts:
          path: php-doc-check-report.txt
          destination: php-doc-check-report
```

---

## 3.3 Déploiement AWS EC2 (À FAIRE )

### Prérequis

1. Une instance EC2 avec :
   - Ubuntu ou Amazon Linux
   - PHP 8.2 installé
   - Apache ou Nginx installé
   - Composer installé
   - Git installé

2. Une clé SSH pour se connecter à l'instance

### Étapes

#### 1. Ajouter la clé SSH dans CircleCI

1. Aller sur CircleCI → Project Settings → **SSH Keys**
2. Cliquer **"Add SSH Key"**
3. Hostname : l'IP ou DNS de votre EC2
4. Private Key : coller votre clé privée SSH
5. Cliquer **"Add SSH Key"**
6. **Copier le fingerprint** généré

#### 2. Ajouter les variables d'environnement dans CircleCI

Aller dans Project Settings → Environment Variables et ajouter :

| Variable | Valeur | Exemple |
|----------|--------|---------|
| `PROD_SSH_USER` | User SSH | `ubuntu` ou `ec2-user` |
| `PROD_SSH_HOST` | IP ou DNS EC2 | `ec2-xx-xx-xx-xx.compute.amazonaws.com` |
| `PROD_SSH_FINGERPRINT` | Le fingerprint copié | `ab:cd:ef:...` |
| `PROD_DEPLOY_DIRECTORY` | Chemin sur le serveur | `/var/www/html` |

#### 3. Préparer l'instance EC2

```bash
# Se connecter à l'instance
ssh -i votre-cle.pem ubuntu@votre-ip-ec2

# Installer les dépendances
sudo apt update
sudo apt install -y php8.2 php8.2-gd php8.2-xml composer git apache2

# Cloner le repo
cd /var/www/html
sudo git clone https://github.com/potichacha/devsecops-php-ci-cd.git .
sudo chown -R www-data:www-data /var/www/html

# Installer les dépendances PHP
sudo composer install --no-dev
```

#### 4. Tester le déploiement

1. Aller sur CircleCI
2. Lancer le pipeline sur `main`
3. Approuver le job `hold`
4. Le job `deploy-ssh-production` devrait s'exécuter

---

# Partie 4 : Documentation et Rapport

## Contenu suggéré pour le rapport

### 1. Introduction
- Présentation du projet (application PHP qui génère des images)
- Objectifs du TP
- Membres du groupe

### 2. Partie 1 : Configuration locale
- Screenshots de l'application en local (http://localhost:8080)
- Explication du Dockerfile
- Commandes utilisées

### 3. Partie 2 : Pipeline CI/CD
- Schéma du pipeline (voir ci-dessous)
- Explication de chaque job
- Screenshots de CircleCI (pipelines verts)
- Configuration Infisical

### 4. Partie 3 : Extensions
- Jobs ajoutés et leur utilité
- Screenshots des rapports générés (phpmetrics, phploc, etc.)

### 5. Partie 5 : Sécurité
- Mesures de sécurité GitHub
- Scan de l'image Docker
- Sécurité EC2

### 6. Conclusion
- Défis rencontrés
- Solutions apportées

---

## Schéma du Pipeline

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           MAIN WORKFLOW                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────┐                                                       │
│  │  debug-info  │                                                       │
│  └──────────────┘                                                       │
│                                                                         │
│  ┌──────────────┐                                                       │
│  │ build-setup  │────┬────────────────────────────────────────┐         │
│  └──────────────┘    │                                        │         │
│                      │                                        │         │
│         ┌────────────┼────────────┬───────────────┐           │         │
│         ▼            ▼            ▼               ▼           ▼         │
│  ┌────────────┐ ┌─────────┐ ┌──────────┐ ┌─────────────┐ ┌─────────┐   │
│  │ lint-phpcs │ │ test-   │ │ security │ │  metrics-   │ │ metrics │   │
│  │            │ │ phpunit │ │ -check   │ │ phpmetrics  │ │ -phploc │   │
│  └────────────┘ └─────────┘ └──────────┘ └─────────────┘ └─────────┘   │
│                      │                                                  │
│         ┌────────────┴────────────┐  (À ajouter)                       │
│         ▼                         ▼                                     │
│  ┌─────────────┐          ┌──────────────────┐                         │
│  │ lint-phpmd  │          │ lint-php-doc-    │                         │
│  │  (À faire)  │          │ check (À faire)  │                         │
│  └─────────────┘          └──────────────────┘                         │
│                                                                         │
│  ┌──────────────┐         ┌─────────────────────────┐                  │
│  │    hold      │────────►│ deploy-ssh-production   │                  │
│  │  (approval)  │         │ (branches: main/master) │                  │
│  └──────────────┘         └─────────────────────────┘                  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                        CONTAINER WORKFLOW                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌────────────────────┐                                                 │
│  │ build-docker-image │───► Push vers GHCR                              │
│  └────────────────────┘     (ghcr.io/potichacha/devsecops-php-ci-cd)   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

# Partie 5 : Sécurité (À FAIRE)

## 5.1 Sécurité GitHub

### Checklist

- [ ] Activer 2FA sur tous les comptes GitHub du groupe
- [ ] Configurer Branch Protection sur `main` :
  1. Settings → Branches → Add rule
  2. Branch name pattern : `main`
  3. Cocher "Require pull request reviews before merging"
- [ ] Vérifier les permissions des collaborateurs
- [ ] Scanner les secrets avec `trufflehog` :
```bash
docker run --rm -it -v "$PWD:/pwd" trufflesecurity/trufflehog:latest github --repo https://github.com/potichacha/devsecops-php-ci-cd
```

## 5.2 Sécurité Docker

### Scanner l'image avec Trivy

```bash
# Installer Trivy (ou utiliser Docker)
docker run --rm aquasec/trivy image ghcr.io/potichacha/devsecops-php-ci-cd:main
```

### Ajouter un job de scan dans le pipeline (optionnel)

```yaml
security-scan-docker:
  executor: builder-executor
  steps:
    - checkout
    - setup_remote_docker
    - run:
        name: Install Trivy
        command: |
          curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
    - run:
        name: Scan Docker image
        command: |
          trivy image ghcr.io/potichacha/devsecops-php-ci-cd:main --severity HIGH,CRITICAL
```

## 5.3 Sécurité AWS EC2

### Checklist

- [ ] Vérifier les Security Groups (ports ouverts)
  - Port 22 (SSH) : uniquement depuis votre IP
  - Port 80 (HTTP) : ouvert
  - Port 443 (HTTPS) : ouvert
- [ ] Désactiver connexion SSH root :
```bash
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```
- [ ] Installer fail2ban :
```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
```
- [ ] Utiliser des clés SSH (pas de mots de passe)
- [ ] Mettre à jour régulièrement :
```bash
sudo apt update && sudo apt upgrade -y
```

---

# Structure du Projet

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

# Résumé des Variables CircleCI

| Variable | Description | Status |
|----------|-------------|--------|
| `GHCR_USERNAME` | Username GitHub | Configuré |
| `GHCR_PAT` | Personal Access Token GitHub | Configuré |
| `INFISICAL_TOKEN` | Service Token Infisical | Configuré |
| `PROD_SSH_USER` | User SSH EC2 production | À configurer |
| `PROD_SSH_HOST` | Host EC2 production | À configurer |
| `PROD_SSH_FINGERPRINT` | Fingerprint clé SSH | À configurer |
| `PROD_DEPLOY_DIRECTORY` | Répertoire de déploiement | À configurer |

---

# Description de l'Application

L'application génère une **image PNG dynamique** qui affiche :
- Le texte "DEVOPS" avec la date et l'heure
- Un message personnalisé
- Le secret `APP_SECRET` (si configuré)

**Technologies utilisées :**
- PHP 8.2
- Librairie GD (génération d'images)
- Carbon (manipulation de dates)
- phpdotenv (chargement des variables d'environnement)

---

# Membres du Groupe

| Nom | Rôle | Tâches |
|-----|------|--------|
| Chantoiseau Sacha | Lead DevOps | Partie 1, 2, 3.1 |
| [Nom 2] | - | - |
| [Nom 3] | - | - |
| [Nom 4] | - | - |
