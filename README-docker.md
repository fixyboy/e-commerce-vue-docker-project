# Docker - E-Commerce Microservices

## Prerequis

- Docker
- Docker Compose

## Lancer en developpement

```bash
docker compose up --build
Lancer en production

docker compose -f docker-compose.prod.yml up --build
Verifier les services

curl http://localhost:3001/api/health
curl http://localhost:3000/api/health
curl http://localhost:3002/api/health
Stopper les conteneurs

docker compose down
Services et ports
auth-service : port 3001
product-service : port 3000
order-service : port 3002
frontend : port 8080
Chaque service a sa propre base de donnes MongoDB.
