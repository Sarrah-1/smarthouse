# Smart Home : Conteneurisation Spring Boot / Angular (TP 24)

Application **Smart Home** conteneurisée avec **Docker** et **Docker Compose**.
Elle gère des catégories et des appareils connectés (allumer / éteindre).

## Architecture

| Service | Technologie | Port (hôte) | Rôle |
|---|---|---|---|
| `frontend` | Angular + Nginx | 80 | Interface web |
| `backend` | Spring Boot (Java 17) | 8085 | API REST |
| `mysql` | MySQL | 3307 | Base de données `smart-house` |
| `phpmyadmin` | phpMyAdmin | 8081 | Administration de la base |

```
Navigateur --> frontend (Nginx :80) --> backend (Spring Boot :8085) --> mysql (:3306)
                                                                          ^
                                                      phpMyAdmin (:8081) -+
```

## Structure du projet

```
smarthouse/
├── Smart_Home_back/        # Backend Spring Boot
│   ├── Dockerfile
│   ├── pom.xml
│   └── src/
├── smartHome-front/        # Frontend Angular
│   ├── Dockerfile
│   ├── package.json
│   └── src/
├── docker-compose.yml      # Orchestration des 4 services
└── README.md
```

## Prérequis

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Docker Compose inclus)
- [Git](https://git-scm.com/)

Vérification :

```bash
docker --version
docker compose version
git --version
```

## Dockerfiles

### Backend (build en 2 étapes)

```dockerfile
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Run
FROM eclipse-temurin:17-jre-alpine
VOLUME /tmp
COPY --from=build /app/target/*.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

### Frontend (build en 2 étapes)

```dockerfile
FROM node:14.15.0-alpine as builder
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist/smart-home /usr/share/nginx/html
```

## Accès aux services

| Service | URL |
|---|---|
| Frontend | http://localhost |
| Backend (API) | http://localhost:8085 |
| phpMyAdmin | http://localhost:8081 (utilisateur `root`, mot de passe `root`) |

### Frontend

<img width="951" height="305" alt="24_1" src="https://github.com/user-attachments/assets/ff6f641f-bd0e-489d-87a1-782921f44a9f" />

### Backend (API REST)

<img width="555" height="273" alt="Capture d&#39;écran 2026-11-25 092255" src="https://github.com/user-attachments/assets/eb89f5ca-1f4a-4316-87eb-bfd3febb99d1" />

### phpMyAdmin (base `smart-house`)

<img width="959" height="392" alt="Capture d&#39;écran 2026-11-25 092248" src="https://github.com/user-attachments/assets/ecbaf9bf-332d-4b3a-9fe2-b6e3a3bbde40" />


## Commandes utiles

```bash
docker compose logs -f backend    # Logs du backend en direct
docker compose logs backend       # Logs du backend
docker compose restart backend    # Redémarrer le backend
docker compose down               # Arrêter et supprimer les conteneurs
docker compose down -v            # Idem + suppression des données MySQL
docker compose up -d --build      # Reconstruire et relancer
```

## Dépannage

| Problème | Solution |
|---|---|
| Port 80 déjà utilisé | Remplacer `"80:80"` par `"8080:80"` dans `docker-compose.yml` |
| Port 3307 déjà utilisé | Changer le port hôte de MySQL |
| Le backend ne démarre pas | Consulter `docker compose logs backend` |
| Le build Angular échoue | Vérifier le `outputPath` dans `angular.json` et la version de Node |

## Auteur

TP réalisé par **<OUADAY SARA>**.
