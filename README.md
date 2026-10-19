# docker-discovery
1-1 For which reason is it better to run the container with a flag -e to give the environment variables rather than put them directly in the Dockerfile?

Il vaut mieux passer les paramètres avec `-e` au lancement pour pouvoir les changer sans reconstruire l’image. Cela évite aussi d’enregistrer les mots de passe dans le Dockerfile.

1-2 Why do we need a volume to be attached to our postgres container?

Le volume conserve les données de PostgreSQL même si le conteneur est supprimé ou recréé.

1-3 Document your database container essentials: commands and Dockerfile.

Commandes utilisées :

```bash
docker network create tp1-network
docker volume create postgres-data
docker build -t tp1-db ./Database
docker run -d --name db --network tp1-network --env-file .env -v postgres-data:/var/lib/postgresql/data tp1-db
docker exec db pg_isready
```

Le Dockerfile part de l’image officielle PostgreSQL, configure la base `db` et l’utilisateur `usr`, puis copie les scripts SQL d’initialisation dans `/docker-entrypoint-initdb.d/`. PostgreSQL les exécute au premier démarrage avec un volume vide.

1-4 Why do we need a multistage build? And explain each step of this Dockerfile.

Un build multi-stage permet de séparer la compilation de l’application de son exécution en utilisant plusieurs étapes dans un même Dockerfile.

Il présente plusieurs avantages :
- Réduire la taille de l’image finale : l’image finale contient uniquement le JRE et le fichier JAR, sans Maven ni JDK.
- Améliorer la sécurité : moins d’outils installés signifie moins de vulnérabilités potentielles.
- Optimiser le déploiement : une image plus légère est plus rapide à transférer et à déployer.
- Séparer la compilation et l’exécution : chaque étape utilise uniquement les outils nécessaires.

1-5 Why do we need a reverse proxy?

Un reverse proxy reçoit les requêtes des clients et les transmet aux services appropriés, comme le backend. Il fournit un point d’entrée unique, peut répartir le trafic entre plusieurs instances, gérer le HTTPS et ajouter des contrôles de sécurité. Il permet aussi de garder les services internes isolés du réseau public.

1-6 Why is docker-compose so important?

Docker Compose permet de décrire plusieurs conteneurs dans un seul fichier et de les démarrer ensemble. Il configure leurs réseaux, leurs ports et leurs volumes, ce qui rend le lancement du projet plus simple et reproductible.

1-7 Document docker-compose most important commands.

```bash
docker compose up -d       # Construire si nécessaire et démarrer les services en arrière-plan
docker compose ps           # Voir l’état des services
docker compose logs -f      # Suivre les journaux des services
docker compose build        # Construire ou reconstruire les images
docker compose down         # Arrêter et supprimer les conteneurs et le réseau du projet
```

1-8 Document your docker-compose file.

Le fichier `docker-compose.yml` définit quatre services : `database`, `backend`, `httpd` et `adminer`. Compose construit les images de PostgreSQL, du backend et d’Apache depuis leurs dossiers respectifs. Deux réseaux séparent les communications : `database-network` relie le backend, PostgreSQL et Adminer ; `proxy-network` relie Apache au backend. Le backend attend que PostgreSQL soit en bonne santé, puis Apache attend que le backend soit prêt. Adminer attend également PostgreSQL et utilise `database` comme serveur par défaut. Les contrôles de santé vérifient PostgreSQL avec `pg_isready` et l’API avec `/actuator/health`.

Les paramètres de connexion PostgreSQL sont lus depuis `.env`, pas écrits dans le fichier Compose. `.env` est ignoré par Git ; `.env.example` donne les noms des variables à configurer. Pour une nouvelle installation, copie `.env.example` vers `.env` et remplace les valeurs d’exemple. Le volume nommé `postgres-data` conserve les données de la base. Apache publie le port `8080` pour l’API, et Adminer publie le port `8081` pour son interface web. PostgreSQL et le backend ne publient pas directement de ports sur la machine.

Pour démarrer l’ensemble : `docker compose up -d --build`. Pour l’arrêter : `docker compose down`. Le volume persistant est conservé lors de cet arrêt.

## Difficultés rencontrées

- Démarrer Docker Desktop et résoudre les conflits de ports entre les conteneurs.
- Relier les conteneurs sur les bons réseaux et configurer les identifiants de PostgreSQL.
- Configurer Apache comme reverse proxy et retrouver son fichier `httpd.conf` depuis Git Bash.
- Organiser les services avec Docker Compose et vérifier leur état avec les logs et les contrôles de santé.

