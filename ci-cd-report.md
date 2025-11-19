# Rapport CI/CD – Projet BobApp

## Introduction

Ce document présente :

- La description complète du workflow CI/CD mis en place pour BobApp
- Les KPIs proposés pour mesurer la qualité
- L'analyse des métriques Sonar, tests back et front
- L'analyse des retours utilisateurs
- Les recommandations pour les prochaines étapes

## 1. Workflow CI/CD

Le workflow GitHub Actions est déclenché à chaque push sur `main`, et se divise en deux jobs :

- **tests** : validation qualité (tests + coverage + SonarCloud)
- **docker-publish** : publication des images Docker si les tests sont validés

### 1.1 Job tests

#### Étape 1 – Checkout

Permet de récupérer l'ensemble du code, avec l'historique (`fetch-depth: 0`) pour permettre l'analyse de "new code" par SonarCloud.

#### Backend (Java 11 – Spring Boot)

##### Étape 2 – Installation du JDK 11 (Temurin)

Prépare l’environnement pour build Maven + tests JUnit.

##### Étape 3 – Exécution des tests backend

Génère automatiquement :

- Tests JUnit
- Coverage JaCoCo

##### Étape 4 – Upload du rapport JaCoCo

Permet de visualiser les rapports HTML dans GitHub Actions.

#### Frontend (Angular)

##### Étape 5 – Installation Node.js 18 + Angular CLI

Prépare l'environnement nécessaire à Angular.

##### Étape 6 – Installation des dépendances

```bash
npm install
```

##### Étape 7 – Exécution des tests frontend avec coverage

```bash
ng test --watch=false --browsers=ChromeHeadless --code-coverage
```

##### Étape 8 – Upload du coverage Angular

Permet de consulter les rapports générés par Istanbul.

#### SonarCloud

##### Étape 9 – Analyse qualité

Effectuée via :

```yaml
uses: SonarSource/sonarqube-scan-action@v6
```

**Fonctionnalités analysées** :

- **Bugs**
- **Vulnérabilités:** Failles de sécurité qui pourraient être exploitées (ex : injections, accès non autorisés).
- **Code smells**: Mauvaises pratiques de code (structure, lisibilité, complexité) qui ne cassent pas l’application mais rendent le code difficile à maintenir.
- **Duplicate code**
- **Couverture**
- **Dette technique**

Un seul fichier `sonar-project.properties` pilote l'analyse du backend et du frontend.

### 1.2 Job docker-publish

**Conditions de déclenchement** :

- Seulement si le job `tests` réussit

**Étapes** :

1. Checkout
2. Connexion à Docker Hub
3. Build & push `bobapp-back`
4. Build & push `bobapp-front`

Les images publiées sont automatiquement taguées `latest`.

## 2. KPIs proposés


### KPI #1 – Couverture minimale

| KPI | Valeur proposée |
|-----|-----------------|
| Coverage global minimal | 70 % |
| Objectif long terme | 80 % |
| Coverage New Code (SonarCloud) | ≥ 85 % |

### KPI #2 – Qualité du "New Code"

| KPI | Seuil |
|-----|-------|
| Blocker sur New Code | 0 |
| Critical sur New Code | 0 |
| Duplications | < 3 % |
| Maintainability Rating | A |
| Reliability Rating | A |


## 3. Analyse des métriques réelles

Les données présentées ci-dessous proviennent de la dernière analyse SonarCloud + rapports de couverture générés par la CI.

### 3.1 Résultats SonarCloud – Vue générale

| Indicateur | Valeur |
|------------|--------|
| Quality Gate | Passed |
| Coverage global | 35.2 % |
| Security Issues | 0 |
| Reliability Issues | 1 |
| Maintainability Issues | 10 |
| Duplications | 0 % |
| Security Hotspots | 2 |
| Severity High Issues | 2 |

> La qualité du code est globalement bonne, mais la couverture de tests reste insuffisante (objectif : 70 %).

### 3.2 Backend – JaCoCo Coverage

| Package | Coverage | Commentaire |
|---------|----------|-------------|
| model | 0 % | Aucun test, zone critique |
| service | 25 % | Logique métier insuffisamment testée |
| controller | 54 % | À renforcer |
| **Global backend** | **32 %** | **Insuffisant pour garantir la stabilité** |

> Le backend est le point faible du projet.

### 3.3 Frontend – Coverage Angular (Istanbul)

| Type | Coverage |
|------|----------|
| Statements | 76.92 % |
| Branches | 100 % |
| Lines | 83.33 % |
| Functions | 57.14 % |

> Le frontend est globalement bien couvert. Seules les fonctions méritent un peu plus de tests.

### 3.4 Analyse des issues Sonar (High Severity)

Deux problèmes majeurs à corriger :

#### 1. Réutilisation d'un objet Random

- **Type** : Reliability
- **Action à mettre en place** : injecter un Random partagé ou utiliser `ThreadLocalRandom`

#### 2. Méthode de test vide

- **Type** : Maintainability
- **Action à mettre en place** : supprimer ou implémenter le test

## 4. Analyse des retours utilisateurs

Les avis recensés affichent une frustration croissante :

| Retour utilisateur | Symptôme | Niveau |
|-------------------|----------|--------|
| "Impossible de poster une suggestion de blague, ça plante" | Erreur front/back non gérée | Critique |
| "Bug sur le post vidéo toujours présent après 2 semaines" | Manque de tests, régressions | Critique |
| "Je ne reçois plus rien depuis une semaine" | Backend instable, absence de monitoring | Critique |
| "J'ai supprimé ce site de mes favoris" | Perte de confiance | Critique |

### Mise en relation avec les métriques techniques

- Backend très peu testé → bugs récurrents
- Fiabilité : 1 issue Sonar ouverte
- Maintainability : 10 issues → dette qui freine les évolutions
- Coverage bas → erreurs non détectées

> Les retours utilisateurs confirment l'analyse technique :  
> **Le backend est la source principale des problèmes.**

## 5. Recommandations prioritaires

### 1. Augmenter la couverture backend

**Cibles** :

- `model` : passer de 0 % → 70 %
- `service` : 25 % → 70 %
- `controller` : 54 % → 80 %

### 2. Corriger les issues High Severity

- Réutilisation du Random
- Méthode de test vide

### 3. Stabiliser les fonctionnalités critiques signalées par les utilisateurs

- Formulaire de suggestion
- Fonctionnalité vidéo
- Mécanisme de réception des blagues