# TP 1 Conteneurisation

Ce TP a pour objectif de vous faire découvrir le fonctionnement des conteneurs Linux à travers Docker.

## Rendus du TP:

- Rendre un fichier contenant les réponses aux questions (format markdown)
- Pour chacune des questions, il faudra veiller à écrire les commandes utilisées permettant de répondre à la question. Ne pas hésiter à ajouter des détails supplémentaires afin de justifier votre réflexion.

## Contexte

Votre objectif sera d'automatiser l'environnement d'exécution d'une application en utilisant la conteneurisation.

## A) Installer Docker  

- Installer Docker en suivant la documentation officielle et s'assurer qu'il est fonctionnel :

**Pour installer Docker, on devait installer WSL2 qui inclut Ubuntu on a du vérifier la version d'Ubuntu et vérifier sa disponibilité, puis installer Docker Desktop**

```bash
docker run hello-world
```

**En executant cette commande on remarque la création d'un conteneur docker avec le nom "hello-world"**

## B) Déployer la base de données

1) Déployer une base données MongoDB (mode standalone) avec Docker. La base de données devra respecter les conditions suivantes:

- Une connection avec un user et un mot de passe sécurisé. 
- La base de données devra être accessible depuis l'host afin d'y accéder depuis un client MongoDB.

**Nous allons déployer une base de données MongoDB avec Docker d'abord, on va utiliser la commande pour créer une base de données MongoDB avec Docker en créant un conteneur en mode détaché, c'est à dire que le conteneur en mode détaché s'arrêtera si le processus racine qui fait marcher le conteneur s'arrête, on précise aussi latest pour avoir la dernière version:**

```bash
docker run --name docker-tp1 -d mongo:latest
```

**Dans le fichier docker-compose.yaml, on a l'USERNAME et le PASSWORD pour se connecter à la base de données MongoDB**

```bash
# Use root/example as user/password credentials
version: '3.1'

services:
  mongo:
    image: mongo
    restart: always
    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGODB_USERNAME}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGODB_PASSWORD}
```

*** Les identifiants de connexions sont en fait donnés dans le fichier .env
```bash
MONGODB_USERNAME="root"
MONGODB_PASSWORD="toto"
```
Nous allons maintenant éxecuter la commande permettant de créer et démarrer le conteneur avec la commande suivante en précisant avec le -f pour préciser le fichier concerné: 

```bash
docker-compose -f docker-compose.yaml up
```

**On peut aussi faire cette commande sans préciser le fichier:**

```bash
docker-compose up
```

**On peut arrêter et supprimer le conteneur avec la commande suivante:**

```bash
docker-compose down
```

La base de données sera utilisé dans la prochaine section.

2) Télécharger MongoDB Compass (client graphique pour visualiser la base de données) et se connecter à MongoDB.
 
**Sur MongoDB Compass, on entre l'adresse url de la connexion de la base de donnée avec l'identifiant et le mot de passe et le port 27017 comme le montre la doc: mongodb://root:toto@localhost:27017**

## C) Déploiement de l'API

### Setup de l'API

#### Installation de Node et Yarn:

Afin de démarrer l'API il est nécessaire d'installer `node` dans votre environnement. Le plus simple est de passer par `nvm`.

`Node` est un runtime javascript permettant d'interpréter le language. `Nvm` (Node Version Manager) est un outil permettant d'installer différentes versions de NodeJS.

Ces 2 commandes vont respectivement installer `nvm` ainsi que la dernier version de `node` :

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
nvm use node
```

Vérifier que `node` est fonctionnel avec cette commande :

```bash
node -v
```

Une application NodeJS est composée de packages permettant d'apporter des fonctionnalités supplémentaires. Un package, est un ensemble de fonctionnalités développées et maintenues par la communauté NodeJS. Un package permet donc d'étendre les fonctionnalités d'une application, sans avoir à les coder directement soi-même. 

Yarn est un package manager permettant de gérer les packages nodejs. Il est nécessaire pour télécharger les dépendances de l'API:
```bash
npm install --global yarn
```

#### Démarrage de l'API

Une fois node et Yarn installés, il est nécessaire de télécharger les dépendances de l'API (il faut se situer dans le dossier de `api`):
```bash
cd api
yarn install
```

Cette commande va télécharger l'ensemble des packages nécessaires au bon fonctionnement de l'application dans le dossier `node_modules`.

Le code de l'application est écrit en Typescript. Il faut pour cela transpiler le code TS en JS grâce à cette commande:

```bash
yarn build
```
Cela va générer un dossier `dist` contenant le code Javascript.

L'api peut ensuite être démarré grâce à la commande suivante:
```bash
node dist/main.js
```

#### Connexion de l'API à MongoDB

Il est nécessaire de configurer la connection à la base de données afin que l'API puisse démarrer.


***La connection à MongoDB est faite dans le fichier `src/app.module.ts` `L8`. L'instruction `process.env.MONGODB_URI` permet de récupérer la valeur de la variable d'environnement `MONGODB_URI`. 
*** On va créer une variable d'environnement en renseignant l'url pour se connecter à la base de donnée mongodb avec la commande :

```bash
 export MONGODB_URI="mongodb://root:toto@localhost:27017"
```

**Puis on va l'afficher avec la commande:**

 ```bash
 echo $MONGO_URI
 ```

**Il est donc nécessaire de configurer cette variable d'environnement avant de démarrer l'API** 

- Chercher sur internet la commande permettant de configurer une variable d'environnement sous Linux.

#### Tester l'API

Une fois démarrée, l'API est accessible à l'adresse suivante : http://localhost:3000.

**en allant sur l'url http://localhost:3000 on remarque que la connexion à la base de donnée mongoDB marche puisque c'est écrit "Hello World" dans le navigateur**


```bash
# Créer un user
curl --location --request POST 'http://localhost:3000/users' \
--header 'Content-Type: application/json' \
--data-raw '{"email":"alexis.bel@ynov.com", "firstName": "Alexis", "lastName": "Bel"}'

# Liste tous les users
curl 'http://localhost:3000/users' 
```
**on crée un utilisateur puis on affiche les informations de l'utilisateur qu'on vient de créer au format json**


La 2ème commande doit renvoyer le user précédemment créé.

### 1) Conteneurisation de l'API

Dans cette partie vous allez conteneuriser l'API précédemment utilisée. Cela vous permettra de packager l'application dans une image Docker, qui pourra alors être déployé de manière totalement indépendante. Cette image sera par ailleurs utilisé dans les prochains TP lors que vous effectuerez son déploiement en production.

**Pour toutes les prochaines questions, vous pouvez vous inspirer de la [section Get started](https://docs.docker.com/get-started/) sur le site Docker**

#### a) Création de l'image à partir d'un Dockerfile

La première étape consiste à créer l'image à partir d'un Dockerfile.


**On doit créer un fichier Dockerfile dans le dossier de l'API qui est constitué des étapes citées au-dessus** 

```bash
# syntax=docker/dockerfile:1
FROM node
COPY . .
RUN yarn install
RUN yarn build
CMD node dist/main.js
```

**On va créer l'image avec la commande suivante avec le nom personnel et en précisant le tag dev:**

```bash
docker build jamil/myapi:dev
```

**On remarque bien la création de la nouvelle image avec le tag dev associé**

S'il n'y a pas d'erreur lors du build de votre image, celle-ci devrait être lisible en lançant la commande `docker image ls`.

#### b) Créer le conteneur à partir de l'image.

La commande `docker run <NOM_DE_VOTRE_IMAGE>` permet de créer un conteneur à partir de l'image de votre API.

Exécuter cette commande. Vous allez recevoir un message d'erreur lorsque le conteneur va essayer de démarrer. Comment expliquez cette erreur ? (N'oubliez pas que l'API a besoin de MongoDB pour fonctionner)

Regarder la [documentation de docker run](https://docs.docker.com/engine/reference/commandline/run/) afin de trouver un moyen de corriger ce problème.


**Si on exécute la commande `docker run jamil/myapi:dev`, nous remarquons une erreur de connexion à la base de données de MongoDB, pour se connecter cette fois-ci à la base de données MongoDB avec l'API, nous devons remplacer localhost par mongo puisque cette fois-ci on a une image mongo et on doit préciser le port 3000 pour l'API.**

#### c) Docker compose 

Mettez à jour la configuration du fichier `docker-compose.yaml` afin déployer le conteneur précédemment créer avec `docker run` en utilisant la commande `docker compose up`

```bash
# Use root/example as user/password credentials
version: '3.1'

services:
  mongo:
    image: mongo
    restart: always
    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGODB_USERNAME}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGODB_PASSWORD}
  api:
    image: jamil/myapi:dev
    restart: always
    ports:
      - 3000:3000
    environment:
      MONGODB_URI: mongodb://${MONGODB_USERNAME}:${MONGODB_PASSWORD}@mongo:27017
```

 **Comme on a configuré le fichier docker-compose.yaml, on n'utilise plus docker run, on utilise maintenant docker compose up**

#### d) Optimiser l'image précédemment créée.

**Pour réduire la taille de l'image, nous allons se servir d'une base alpine qui est de taille inférieure à celle qu'on a actuellement, voici le Dockerfile modifié:** 

```bash
# syntax=docker/dockerfile:1
FROM node:alpine
COPY . .
RUN yarn install
RUN yarn build
CMD node dist/main.js
```

**Nous allons également créer un .dockerignore dans le même dossier que le docker-compose.yaml afin d'ignorer les dossiers node_modules, dist et le fichier Dockerfile car ils ne vont plus nous servir et prennent de la place**

```bash
node_modules
dist
Dockerfile
```

#### d) Partager l'image sur le Dockerhub

Créer vous un compte sur le DockerHub et partagez l'image que vous avez créé. 
Vous pouvez vous référer à [cette documentation de Docker](https://docs.docker.com/get-started/04_sharing_app/)


## D) Persistance des données

### 1) Supprimer le conteneur MongoDB et le récréer. Qu'est il arrivé aux données et pourquoi ?

Les données d'un conteneur sont versatiles, elles sont supprimées quand le conteneur est supprimé. Il est nécessaire de mettre en place un volume qui va persister les données les données une fois le conteneur supprimé. 

Le conteneur va monter ce volume à chaque fois qu'il démarrera, ce qui lui permettra de récupérer les données de l'ancien conteneur.
### 2) Mettre en place un mécanisme permettant de persister les données même quand le conteneur est supprimé (pensez aux volumes).

Voir le `docker-compose.yaml`

## E) (Bonus) Réplication de l'API et load balancing avec un proxy NGINX

- Créer plusieurs replica de l'API 
- Créer un conteneur NGINX en front gérant les requêtes vers l'API et permettant de load balancer les requêtes vers les différents replica de l'API, le tout à partir d'un même hostname.