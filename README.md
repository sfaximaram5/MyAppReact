# TP4 : Docker et Docker Compose — Application MERN

ISG de Sousse — Introduction au DevOps — 3ème année LIG

Application : `Client` (React + Vite) et `Server` (API Express). MongoDB tourne dans son propre conteneur.

## Structure

```
MyAppReact/
├── Client/              # React (build Vite servi par "serve" sur le port 3000)
│   ├── Dockerfile
│   └── .dockerignore
├── Server/              # API Express (port 9000)
│   ├── Dockerfile
│   └── .dockerignore
└── docker-compose.yml
```

> Les dossiers du dépôt s'appellent `Client` et `Server` (majuscules). Les commandes
> ci-dessous utilisent ces noms exacts, car Linux est sensible à la casse.

## Étape 1 : Cloner le projet

```bash
git clone https://github.com/wiem-kb/MyAppReact.git
cd MyAppReact
```

## Étape 2 : Dockerfiles

### Serveur (`Server/Dockerfile`), image `mern-server`

1. Image de base `node:lts-alpine`
2. `WORKDIR /app`
3. Copie de `package*.json`, puis `npm install` (le cache Docker évite de réinstaller si les dépendances ne changent pas)
4. Copie du reste des fichiers
5. `EXPOSE 9000`
6. `CMD ["npm", "start"]`
7. Variable `MONGO_URI` (valeur par défaut dans le Dockerfile, définie pour de bon dans Docker Compose)

```bash
docker build -t mern-server ./Server
```

### Client (`Client/Dockerfile`), image `mern-client`

1. Image de base `node:lts-alpine`
2. `WORKDIR /app`
3. Copie de `package*.json`, puis `npm install`
4. Copie du reste des fichiers, puis `npm run build` (sortie dans `dist/`)
5. Installation globale du serveur HTTP léger `serve`
6. `EXPOSE 3000`
7. `CMD ["serve", "-s", "dist", "-l", "3000"]`

```bash
docker build -t mern-client ./Client
```

> Dans l'énoncé, la commande du client est écrite `docker build -t mern-server ./client`.
> C'est une coquille : l'image du client doit s'appeler `mern-client`, sinon elle écraserait
> l'image du serveur.

### Pourquoi `package-lock.json` est exclu côté client

Le lockfile du client a été généré sous Windows et ne contient pas le binaire natif de
`rolldown` (le bundler utilisé par `rolldown-vite`) pour Linux. Dans le conteneur Alpine,
`npm run build` échoue alors avec « Cannot find native binding ». Le fichier
`Client/.dockerignore` exclut donc `package-lock.json` : npm résout les bonnes dépendances
pour la plateforme du conteneur.

## Étape 3 : Réseau Docker

```bash
docker network create mern-network
```

## Étape 4 : MongoDB

```bash
docker run -d --name mongodb --network mern-network mongo
```

## Étape 5 : Serveur et client

```bash
docker run -d --name server --network mern-network -p 9000:9000 mern-server
docker run -d --name client --network mern-network -p 3000:3000 mern-client
```

Vérification :

- API : http://localhost:9000/api/health → `{"message":"API is running"}`
- Client : http://localhost:3000 → affiche « API is running »

Le client appelle l'API depuis le navigateur sur `http://localhost:9000`, c'est pourquoi le
port 9000 doit être publié (`-p 9000:9000`).

Nettoyage avant l'étape suivante :

```bash
docker rm -f client server mongodb
docker network rm mern-network
```

## Étape 6 : Docker Compose

Le fichier `docker-compose.yml` à la racine définit :

| Service   | Source            | Conteneur | Port        | Dépend de | Particularité                                      |
|-----------|-------------------|-----------|-------------|-----------|----------------------------------------------------|
| `mongodb` | image `mongo`     | mongodb   | —           | —         |                                                    |
| `server`  | build `./Server`  | server    | 9000:9000   | mongodb   | `MONGO_URI=mongodb://mongodb:27017/mern`           |
| `client`  | build `./Client`  | client    | 3000:3000   | server    |                                                    |

Les trois services sont connectés au réseau `mern-network` (driver `bridge`). Dans Compose,
les services se joignent par leur nom : le serveur atteint MongoDB via l'hôte `mongodb`.

Lancer l'application :

```bash
docker compose up --build
```

Arrêter et supprimer les conteneurs :

```bash
docker compose down
```

## Remarque

Le code actuel du serveur (`Server/src/index.mjs`) n'utilise pas encore MongoDB : il n'y a
pas de dépendance `mongoose` et `MONGO_URI` n'est pas lue. La variable est tout de même
définie, comme l'exige le TP, et sera exploitable dès que le serveur se connectera à la base.
