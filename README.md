# PHP DevOps TP - Pipeline CI/CD

Application PHP de démonstration pour le cours DevSecOps.

**Projet GitHub** : https://github.com/potichacha/devsecops-php-ci-cd

---

## Pour refaire le projet 

### Ce qui est déjà configuré

- [x] Repo GitHub créé
- [x] CircleCI connecté au repo
- [x] Variables d'environnement CircleCI (GHCR_USERNAME, GHCR_PAT, INFISICAL_TOKEN)
- [x] Infisical configuré avec le secret APP_SECRET
- [x] Pipeline CI/CD fonctionnel (build, test, lint, security)
- [x] Job deploy-ssh-production créé

### Ce que vous devez faire

#### 1. Rejoindre le repo GitHub

1. Cloner le repo :
```bash
git clone https://github.com/potichacha/devsecops-php-ci-cd.git
cd devsecops-php-ci-cd
```

#### 2. Accéder à CircleCI

1. Aller sur **https://circleci.com**
2. Se connecter avec **GitHub**
3. Le projet **devsecops-php-ci-cd** devrait apparaître automatiquement
4. Vous pouvez voir les pipelines et leur statut

#### 3. Accéder à Infisical (optionnel)

1. Aller sur **https://infisical.com** et accepter l'invitation
2. Vous pourrez voir/modifier les secrets

#### 4. Tester l'application en local

```bash
# Construire l'image Docker
docker build -t php-devops-tp -f Docker/Dockerfile .

# Lancer le conteneur
docker run -d -p 8080:80 -e APP_SECRET="MonSecretTest" --name php-devops-app php-devops-tp

# Installer les dépendances
docker exec php-devops-app composer install --no-interaction

# Tester : ouvrir http://localhost:8080
```

#### 5. Workflow de travail

```bash
# Créer une branche pour vos modifications
git checkout -b feature/mon-travail

# Faire vos modifications...

# Commiter et pusher
git add .
git commit -m "Description de vos modifications"
git push origin feature/mon-travail

# Créer une Pull Request sur GitHub
```

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

## PARTIE 2 : Configuration CI/CD (fait )

### Étape 2.3 : Connecter CircleCI au repo

1. Aller sur **https://circleci.com**
2. Cliquer **"Log In"** → **"Log in with GitHub"**
3. Autoriser CircleCI
4. Trouver le projet **devsecops-php-ci-cd**
5. Cliquer **"Set Up Project"**
6. Sélectionner **"Fastest: Use the .circleci/config.yml in my repo"**
7. Cliquer **"Set Up Project"**

### Étape 2.4 : Créer un Personal Access Token GitHub (pour GHCR)

1. GitHub → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
2. **Generate new token (classic)**
3. Nom : `circleci-ghcr`
4. Cocher : `write:packages`, `read:packages`
5. **Generate token** → **COPIER LE TOKEN**

### Étape 2.5 : Ajouter les variables dans CircleCI

1. CircleCI → Projet → **Project Settings** → **Environment Variables**
2. Ajouter :

| Name | Value |
|------|-------|
| `GHCR_USERNAME` | ton username GitHub |
| `GHCR_PAT` | le token GitHub |

### Étape 2.6 : Configurer Infisical

1. Créer un compte sur **https://infisical.com**
2. Créer un projet `php-devops-tp`
3. Ajouter un secret : `APP_SECRET` = `MonSecretInfisical123`
4. Créer un Service Token (Access Control → Service Tokens)
5. Ajouter dans CircleCI : `INFISICAL_TOKEN` = le token

---

## PARTIE 3 : Extension du Pipeline (À FAIRE)

### 3.1 Jobs d'évaluation de code à ajouter

#### Job phpmetrics
> Génère des métriques de qualité du code PHP

```yaml
# À ajouter dans .circleci/config.yml (section jobs)
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

#### Job phploc
> Compte les lignes de code, classes, méthodes, etc.

```yaml
metrics-phploc:
  executor: php-executor
  steps:
    - *attach_workspace
    - run:
        name: Install phploc
        command: composer require --dev phploc/phploc
    - run:
        name: Run phploc
        command: ./vendor/bin/phploc --log-xml=phploc-report.xml ./src
    - store_artifacts:
        path: phploc-report.xml
        destination: phploc-report
```

### 3.2 Jobs de qualité de code à ajouter

#### Job phpmd (PHP Mess Detector)
> Détecte les problèmes de conception du code

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
          ./vendor/bin/phpmd ./src html cleancode,codesize,controversial,design,naming,unusedcode --reportfile phpmd-report.html || true
    - store_artifacts:
        path: phpmd-report.html
        destination: phpmd-report
```

#### Job php-doc-check
> Vérifie la documentation du code

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
        command: ./vendor/bin/php-doc-check ./src || true
```

### 3.3 Ajouter les jobs au workflow

```yaml
# Dans la section workflows → main_workflow → jobs
- metrics-phpmetrics:
    requires:
      - build-setup
- metrics-phploc:
    requires:
      - build-setup
- lint-phpmd:
    requires:
      - build-setup
- lint-php-doc-check:
    requires:
      - build-setup
```

### 3.4 Déploiement AWS EC2

#### Prérequis
1. Une instance EC2 avec PHP, Apache/Nginx, Composer installés
2. Une clé SSH pour se connecter à l'instance
3. Le repo cloné sur l'instance

#### Variables à ajouter dans CircleCI

| Variable | Description |
|----------|-------------|
| `PROD_SSH_USER` | `ubuntu` ou `ec2-user` |
| `PROD_SSH_HOST` | IP ou DNS de l'instance EC2 |
| `PROD_SSH_FINGERPRINT` | Fingerprint de la clé SSH |
| `PROD_DEPLOY_DIRECTORY` | `/var/www/html` |

#### Ajouter la clé SSH dans CircleCI
1. Project Settings → **SSH Keys**
2. **Add SSH Key**
3. Coller la clé privée
4. Copier le fingerprint généré

---

## PARTIE 4 : Documentation et Rapport (À FAIRE)

### Contenu du rapport

1. **Introduction**
   - Présentation du projet
   - Objectifs du TP

2. **Partie 1 : Configuration locale**
   - Screenshots de l'application en local
   - Explication du Dockerfile

3. **Partie 2 : Pipeline CI/CD**
   - Schéma du pipeline (voir ci-dessous)
   - Explication de chaque job
   - Screenshots de CircleCI

4. **Partie 3 : Extensions**
   - Jobs ajoutés et leur utilité
   - Screenshots des rapports générés

5. **Partie 5 : Sécurité**
   - Mesures de sécurité GitHub
   - Scan de l'image Docker
   - Sécurité EC2

6. **Conclusion**
   - Défis rencontrés
   - Solutions apportées

### Schéma du Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                        MAIN WORKFLOW                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐                                               │
│  │  debug-info  │                                               │
│  └──────────────┘                                               │
│                                                                 │
│  ┌──────────────┐    ┌─────────────┐    ┌────────────────────┐  │
│  │ build-setup  │───►│  lint-phpcs │    │ security-check-    │  │
│  └──────────────┘    └─────────────┘    │ dependencies       │  │
│         │                               └────────────────────┘  │
│         │            ┌─────────────┐                            │
│         ├───────────►│ test-phpunit│                            │
│         │            └─────────────┘                            │
│         │            ┌─────────────┐                            │
│         ├───────────►│ lint-phpmd  │  (À ajouter)               │
│         │            └─────────────┘                            │
│         │            ┌──────────────────┐                       │
│         ├───────────►│ metrics-phpmetrics│  (À ajouter)         │
│         │            └──────────────────┘                       │
│         │            ┌──────────────┐                           │
│         └───────────►│ metrics-phploc│  (À ajouter)             │
│                      └──────────────┘                           │
│                                                                 │
│  ┌──────────────┐    ┌─────────────────────────┐                │
│  │    hold      │───►│ deploy-ssh-production   │                │
│  │  (approval)  │    │ (branches: main/master) │                │
│  └──────────────┘    └─────────────────────────┘                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                     CONTAINER WORKFLOW                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌────────────────────┐                                         │
│  │ build-docker-image │───► Push to GHCR                        │
│  └────────────────────┘                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## PARTIE 5 : Sécurité (À FAIRE)

### 5.1 Sécurité GitHub

- [ ] Activer 2FA sur tous les comptes
- [ ] Configurer Branch Protection sur `main`
- [ ] Vérifier les permissions des collaborateurs
- [ ] Scanner les secrets avec `git-secrets` ou `trufflehog`

### 5.2 Sécurité Docker

- [ ] Scanner l'image avec **Trivy** :
```bash
trivy image ghcr.io/potichacha/devsecops-php-ci-cd:main
```
- [ ] Ajouter un job de scan dans le pipeline

### 5.3 Sécurité AWS EC2

- [ ] Vérifier les Security Groups (ports ouverts)
- [ ] Désactiver connexion SSH root
- [ ] Configurer fail2ban
- [ ] Utiliser des clés SSH (pas de mots de passe)

---

## Résumé des Variables CircleCI

| Variable | Description | Statut |
|----------|-------------|--------|
| `GHCR_USERNAME` | Username GitHub | ✅ Configuré |
| `GHCR_PAT` | Personal Access Token GitHub | ✅ Configuré |
| `INFISICAL_TOKEN` | Service Token Infisical | ✅ Configuré |
| `PROD_SSH_USER` | User SSH EC2 production | ❌ À configurer |
| `PROD_SSH_HOST` | Host EC2 production | ❌ À configurer |
| `PROD_SSH_FINGERPRINT` | Fingerprint clé SSH | ❌ À configurer |
| `PROD_DEPLOY_DIRECTORY` | Répertoire de déploiement | ❌ À configurer |

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

---

## Membres du Groupe

- Membre 1 : [Nom]
- Membre 2 : [Nom]
- Membre 3 : [Nom]
- Membre 4 : [Nom]
