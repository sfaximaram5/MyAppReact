# Image de base légère
FROM node:lts-alpine

# Répertoire de travail dans le conteneur
WORKDIR /app

# Copier d'abord package*.json pour profiter du cache Docker
# (package-lock.json est exclu par .dockerignore, voir README)
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier le reste des fichiers du client
COPY . .

# Construire l'application React (sortie dans /app/dist)
RUN npm run build

# Serveur HTTP léger pour servir les fichiers de build
RUN npm install -g serve

# Port du client
EXPOSE 3000

# Servir le dossier de build (-s : mode SPA, -l : port d'écoute)
CMD ["serve", "-s", "dist", "-l", "3000"]
