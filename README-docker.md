#!/bin/bash

# Projet Docker - E-Commerce Microservices

## Architecture

Navigateur --> Nginx (port 8080) --> backends

| Service | Port | Base de donnees |
|---------|------|-----------------|
| auth-service | 3001 | mongo-auth:27017/authdb |
| product-service | 3000 | mongo-product:27017/productdb |
| order-service | 3002 | mongo-order:27017/orderdb |
| frontend (Nginx) | 8080 | - |

Chaque microservice possede sa propre base MongoDB. Nginx sert de reverse proxy et de serveur pour les fichiers statiques du frontend Vue.js.

## Prerequis

- Docker et Docker Compose installes
- Ports 8080, 3000, 3001, 3002 disponibles

## Demarrage

### Developpement

    cd /opt/ecommerce
    docker compose up --build -d

### Production

Creer un fichier .env a la racine :

    JWT_SECRET=efrei_super_pass

Lancer :

    docker compose -f docker-compose.prod.yml up --build -d

### Arreter

    docker compose down

## Verification

    curl http://localhost:3001/api/health
    curl http://localhost:3000/api/health
    curl http://localhost:3002/api/health

## Initialisation des donnees

    bash scripts/init-products.sh

## Tests

    bash scripts/run-tests.sh

## Scan de securite

    trivy image ecommerce-auth-service --severity HIGH,CRITICAL

## Structure des Dockerfiles

Les 3 microservices backend utilisent un Dockerfile multi-stage :
- base : image node:18-alpine, copie de package.json
- development : npm install (toutes les deps), nodemon pour le hot-reload
- production : npm ci --only=production, utilisateur non-root (appuser)

Le frontend utilise aussi un multi-stage :
- Stage 1 : compilation de Vue.js (npm run build)
- Stage 2 : image nginx:alpine avec les fichiers compiles et la config du reverse proxy

## Dev vs Prod

Dev : docker-compose.yml / target development / volumes hot-reload / JWT en dur / root
Prod : docker-compose.prod.yml / target production / pas de volumes / JWT depuis .env / appuser non-root / restart always / NODE_ENV=production

## Bonnes pratiques

- Multi-stage builds pour separer dev et prod
- Images Alpine (~50MB au lieu de ~900MB)
- Conteneurs de prod en utilisateur non-root
- .dockerignore pour exclure .env, node_modules, .git
- Secrets jamais dans les images ni dans git
- npm ci --only=production en prod
- Volumes nommes pour la persistance MongoDB
- Scan Trivy pour les vulnerabilites

## Choix techniques

- Nginx comme reverse proxy unique
- 1 MongoDB par service pour isoler les donnees
- Docker Compose pour orchestrer les 7 conteneurs
- GitFlow avec branches main et develop

## Problemes rencontres

1. Erreur JWT malformed : Nginx supprimait le header Authorization. Corrige avec proxy_set_header dans nginx.conf.

2. Produit non trouve dans order-service : il manquait VITE_PRODUCT_SERVICE_URL dans docker-compose.yml.

3. Tests qui echouent (libcrypto) : MongoDB 4.4 necessite OpenSSL 1.1 absent de Debian 12. Installe libssl1.1 depuis Debian 11.

4. Base vide au demarrage : execute scripts/init-products.sh pour peupler les produits.
ENDOFFILE

echo "README-docker.md mis a jour"

