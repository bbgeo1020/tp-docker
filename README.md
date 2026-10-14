# TP Discover Docker - Architecture 3-Tiers

Ce projet présente la mise en place d'une application 3-tiers conteneurisée à l'aide de **Docker** et **Docker Compose**, réalisée dans le cadre du TP de découverte de Docker.

---

##  Architecture du Projet

L'application repose sur 3 services interconnectés au sein d'un réseau Docker dédié (`app-network`) :
1. **Database (PostgreSQL 17.2-alpine)** : Stocke les données des départements et des étudiants, avec initialisation automatique via des scripts SQL.
2. **Backend (Spring Boot 3.4.5)** : API REST Java 21 connectée à la base de données.
3. **HTTP Server (Apache httpd 2.4)** : Sert de serveur web et de reverse proxy pour rediriger le trafic vers le backend.

---

##  Structure des Dossiers

```text
tp-docker/
├── backend/
│   ├── Dockerfile
│   ├── pom.xml
│   └── src/
├── database/
│   ├── Dockerfile
│   ├── 01-CreateScheme.sql
│   └── 02-InsertData.sql
├── http-server/
│   ├── Dockerfile
│   └── httpd.conf
└── docker-compose.yml

Réponses aux Questions du TP (Rapport)
1. Base de données (PostgreSQL)
1-1. Pourquoi est-il préférable d'utiliser le flag -e pour les variables d'environnement plutôt que de les écrire en clair dans le Dockerfile ?

Pour des raisons de sécurité. Inscrire des mots de passe en clair dans un Dockerfile les fige dans l'historique des couches de l'image et sur le système de contrôle de version (Git). L'utilisation de flags (-e) ou de fichiers de configuration sécurisés permet d'injecter dynamiquement les informations sensibles à l'exécution.

1-2. Pourquoi avons-nous besoin d'un volume attaché au conteneur Postgres ?

Les conteneurs Docker sont par nature éphémères. Si un conteneur de base de données est supprimé (docker rm), toutes les données qu'il contient sont perdues. L'utilisation d'un volume Docker permet de persister les données durablement sur le disque de la machine hôte.

1-3. Essentiels de la base de données :

Image : postgres:17.2-alpine

Les scripts d'initialisation (01-CreateScheme.sql et 02-InsertData.sql) sont placés dans /docker-entrypoint-initdb.d/ pour s'exécuter automatiquement au premier démarrage.

2. Backend API (Spring Boot)
1-4. Pourquoi avons-nous besoin d'un multistage build et explication des étapes :

Le multistage build permet d'alléger considérablement l'image finale en évitant d'y inclure les outils de développement et de compilation (JDK complet, Maven).

Étape 1 (Build - myapp-build) : Utilise une image lourde (eclipse-temurin:21-jdk-alpine) avec Maven pour télécharger les dépendances et compiler le code source en fichier .jar.

Étape 2 (Run / Stage final) : Utilise une image légère contenant uniquement le runtime nécessaire (eclipse-temurin:21-jre-alpine), puis copie uniquement le fichier .jar généré à l'étape précédente pour l'exécuter.

3. Serveur HTTP & Reverse Proxy
1-5. Pourquoi avons-nous besoin d'un reverse proxy ?

Le reverse proxy (ici configuré avec Apache) agit comme une passerelle unique entre les clients (navigateurs) et les services internes. Il permet de centraliser le trafic, de masquer l'infrastructure réseau interne, de gérer la sécurité et de faciliter l'évolutivité (load balancing).

4. Orchestration (Docker Compose)
1-6. Pourquoi Docker Compose est-il si important ?

Il permet d'administrer des architectures multi-conteneurs complexes de manière centralisée. Au lieu de lancer manuellement chaque conteneur avec de longs arguments (réseaux, ports, volumes, dépendances), un fichier unique (docker-compose.yml) et une seule commande suffisent.

1-7. Commandes essentielles de Docker Compose :

docker compose up --build -d : Construit et lance les services en arrière-plan.

docker compose down : Arrête et supprime les conteneurs et réseaux.

docker compose logs -f : Affiche les logs en temps réel.

1-8. Structure du fichier docker-compose.yml :

Le fichier orchestre les trois services (database, backend, httpd), les relie via un réseau virtuel commun (app-network), gère la persistance des données via les volumes et définit l'ordre de démarrage (depends_on).

5. Publication (Docker Hub)
1-9 & 1-10. Commandes de publication et utilité :

Commandes : docker login, docker tag my-image user/my-image:1.0, puis docker push user/my-image:1.0.

Utilité d'un dépôt en ligne : Centraliser et partager les images prêtes à l'emploi avec les membres d'une équipe ou faciliter le déploiement sur des environnements distants/production.