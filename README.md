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
| 3.3 | Déploiement AWS EC2 | Fait | Malik |
| 3.3 | Création instance EC2 t3.micro Ubuntu 22.04 | Fait | Malik |
| 3.3 | Installation PHP 8.2 + extensions (GD, XML, mbstring, curl, zip) | Fait | Malik |
| 3.3 | Installation et configuration Apache 2.4 | Fait | Malik |
| 3.3 | Installation Composer | Fait | Malik |
| 3.3 | Configuration DocumentRoot Apache vers /var/www/html/public | Fait | Malik |
| 3.3 | Clonage repository sur serveur EC2 | Fait | Malik |
| 3.3 | Configuration clé SSH CircleCI (fingerprint SHA256:fVrzNmOU1wUYS9sRBQFH...) | Fait | Malik |
| 3.3 | Configuration variables CircleCI (PROD_SSH_USER, PROD_SSH_HOST, PROD_DEPLOY_DIRECTORY) | Fait | Malik |
| 3.3 | Modification deploy-ssh-production job avec identification explicite clé SSH | Fait | Malik |
| 3.3 | Test et validation déploiement automatique sur http://51.20.52.225 | Fait | Malik |
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

### 4. Accéder à AWS EC2 (si nécessaire)

**Important** : L'instance EC2 est déjà configurée et fonctionnelle.

Pour accéder à l'instance :
1. Récupérer la clé SSH `php-devops-production.pem`
2. Placer la clé dans un endroit sûr (ex: `~/.ssh/`)
3. Modifier les permissions :
```bash
chmod 400 php-devops-production.pem
```
1. Se connecter :
```bash
ssh -i php-devops-production.pem ubuntu@51.20.52.225
```

**Informations sur l'instance** :
- Type : t3.micro
- OS : Ubuntu 22.04 LTS
- IP : 51.20.52.225
- URL : http://51.20.52.225
- Dossier du projet : `/var/www/html`

### 5. Workflow de travail

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
| `GHCR_USERNAME` | Username GitHub | Fait |
| `GHCR_PAT` | Personal Access Token GitHub | Fait |
| `INFISICAL_TOKEN` | Service Token Infisical | Fait |
| `PROD_SSH_USER` | User SSH pour EC2 | Fait |
| `PROD_SSH_HOST` | IP ou DNS de l'instance EC2 | Fait |
| `PROD_DEPLOY_DIRECTORY` | Répertoire de déploiement sur EC2 | Fait |

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
| `deploy-ssh-production` | Déploiement SSH vers EC2 | Fait |

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

## 3.3 Déploiement AWS EC2 (Fait)

### Instance EC2 de production

**Informations** :
- Type : t3.micro
- OS : Ubuntu 22.04 LTS
- IP publique : 51.20.52.225
- URL : http://51.20.52.225
- Ports ouverts : 22 (SSH), 80 (HTTP), 443 (HTTPS)

**Logiciels installés** :
- PHP 8.2 avec extensions (GD, XML, mbstring, curl, zip)
- Composer
- Apache 2.4
- Git

### Configuration réalisée

#### 1. Création de l'instance EC2

1. Instance créée sur AWS EC2
2. Type : t3.micro (éligible au Free Tier)
3. AMI : Ubuntu Server 22.04 LTS
4. Clé SSH créée et téléchargée : `php-devops-production.pem`
5. Security Group configuré avec les ports 22, 80, 443 ouverts

#### 2. Installation des dépendances sur EC2

Connexion SSH et installation :

```bash
# Connexion à l'instance
ssh -i php-devops-production.pem ubuntu@51.20.52.225

# Mise à jour du système
sudo apt update && sudo apt upgrade -y

# Installation de PHP 8.2 et extensions
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
sudo apt install -y php8.2 php8.2-cli php8.2-fpm php8.2-mysql php8.2-xml \
                    php8.2-mbstring php8.2-curl php8.2-zip php8.2-gd php8.2-intl

# Installation de Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# Installation d'Apache
sudo apt install -y apache2 libapache2-mod-php8.2
sudo a2enmod rewrite
sudo systemctl enable apache2
sudo systemctl start apache2

# Installation de Git
sudo apt install -y git
```

#### 3. Clonage du projet

```bash
# Préparation du répertoire web
sudo rm -rf /var/www/html/*
cd /var/www/html

# Clonage du repo
sudo git clone https://github.com/potichacha/devsecops-php-ci-cd.git .

# Installation des dépendances
sudo composer install --optimize-autoloader --no-interaction --prefer-dist --no-dev

# Configuration des permissions
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

#### 4. Configuration Apache

Fichier `/etc/apache2/sites-available/000-default.conf` :

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/public

    <Directory /var/www/html/public>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Redémarrage d'Apache :
```bash
sudo systemctl restart apache2
```

#### 5. Configuration CircleCI pour le déploiement

**Clé SSH ajoutée dans CircleCI** :
1. Project Settings → SSH Keys
2. Hostname : `51.20.52.225`
3. Private Key : contenu de `php-devops-production.pem`
4. Fingerprint : `SHA256:fVrzNmOU1wUYS9sRBQFH+SFZVU41VH0Tv7FoHO5qlxg`

**Variables d'environnement configurées** :

| Variable | Valeur |
|----------|--------|
| `PROD_SSH_USER` | `ubuntu` |
| `PROD_SSH_HOST` | `51.20.52.225` |
| `PROD_DEPLOY_DIRECTORY` | `/var/www/html` |

**Job de déploiement dans `.circleci/config.yml`** :

```yaml
deploy-ssh-production:
  executor: simple-executor
  steps:
    - checkout
    - add_ssh_keys:
        fingerprints:
          - "SHA256:fVrzNmOU1wUYS9sRBQFH+SFZVU41VH0Tv7FoHO5qlxg"
    - run:
        name: Deploy to Production AWS
        command: |
          KEY_FILE=$(ls ~/.ssh/id_rsa_* 2>/dev/null | head -n1)
          echo "Using key: $KEY_FILE"
          chmod 600 "$KEY_FILE"
          ssh -i "$KEY_FILE" -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o IdentitiesOnly=yes $PROD_SSH_USER@$PROD_SSH_HOST \<< 'EOF'
          PHP_FPM_VERSION=$(php -v | head -n 1 | cut -d ' ' -f 2 | cut -d '.' -f 1-2)
          cd $PROD_DEPLOY_DIRECTORY
          git pull origin main
          composer install --optimize-autoloader --no-interaction --prefer-dist --no-dev
          sudo service php${PHP_FPM_VERSION}-fpm restart || true
          EOF
```

### Workflow de déploiement

1. Un commit est poussé sur la branche `main`
2. CircleCI lance automatiquement le pipeline
3. Tous les jobs (build, test, lint, metrics) s'exécutent
4. Le job `hold` attend une approbation manuelle
5. Un développeur approuve le déploiement sur CircleCI
6. Le job `deploy-ssh-production` se connecte en SSH à EC2
7. Le code est mis à jour via `git pull`
8. Les dépendances sont installées
9. Apache/PHP-FPM est redémarré
10. L'application est déployée et accessible sur http://51.20.52.225

### Pour les collaborateurs : Tester le déploiement

1. Faire une modification dans le code (ex: `src/ImageCreator.php`)
2. Commiter et pusher sur `main`
3. Aller sur CircleCI et attendre que le pipeline soit au vert
4. Cliquer sur le job `hold` et approuver
5. Attendre que `deploy-ssh-production` se termine
6. Vérifier sur http://51.20.52.225 que la modification est visible

### Accès à l'instance EC2

**Important** : La clé SSH (`php-devops-production.pem`) n'est PAS dans le repo pour des raisons de sécurité.

Pour accéder à l'instance EC2 :
1. Demander la clé SSH à Sacha
2. Placer la clé dans un endroit sûr (ex: `~/.ssh/`)
3. Modifier les permissions : `chmod 400 php-devops-production.pem`
4. Se connecter : `ssh -i php-devops-production.pem ubuntu@51.20.52.225`

### Dépannage

**Problème : Le déploiement échoue avec "Permission denied"**
- Vérifier que la variable `PROD_SSH_USER` est bien `ubuntu`
- Vérifier que la clé SSH est bien ajoutée dans CircleCI
- Vérifier que le fingerprint est correct dans `config.yml`

**Problème : L'application ne s'affiche pas après le déploiement**
- Se connecter en SSH à l'instance
- Vérifier les logs Apache : `sudo tail -f /var/log/apache2/error.log`
- Vérifier que le DocumentRoot pointe bien vers `/var/www/html/public`

**Problème : "composer: command not found"**
- Réinstaller Composer sur l'instance EC2 (voir section Installation)

---

## 3.4 Ancienne section (pour référence uniquement)

Cette section contenait les instructions pour configurer AWS EC2 manuellement. Elle a été remplacée par la section 3.3 qui documente la configuration déjà réalisée.

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
