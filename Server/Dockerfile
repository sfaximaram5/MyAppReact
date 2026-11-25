# Image de base légère
FROM node:lts-alpine

# Répertoire de travail dans le conteneur
WORKDIR /app

# Copier d'abord package*.json pour profiter du cache Docker
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier le reste des fichiers du serveur
COPY . .

# URL de connexion à MongoDB (valeur par défaut, surchargée par Docker Compose)
ENV MONGO_URI=mongodb://mongodb:27017/mern

# Port de l'API Express
EXPOSE 9000

# Démarrer l'application
CMD ["npm", "start"]
