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
docker run -d --name db --network tp1-network -v postgres-data:/var/lib/postgresql/data tp1-db
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

