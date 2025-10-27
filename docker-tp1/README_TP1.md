
1-1 Pour quelle raison est-il préférable de faire tourner le conteneur avec un drapeau -e plutôt que le Dockerfile pour les variables d'environnement ?
Il est préférable d'utiliser le drapeau -e (ou un fichier .env) pour des raisons de sécurité et de flexibilité.

Sécurité : Placer les mots de passe dans le Dockerfile les rend visibles dans l'historique de l'image (les "couches"). Le drapeau -e injecte la variable au moment de l'exécution, empêchant les secrets d'être stockés dans l'image.

Flexibilité : Cela permet de réutiliser la même image dans différents environnements (développement, production) en changeant simplement la valeur du mot de passe à chaque lancement, sans avoir à reconstruire l'image.


1-2 Pourquoi avons-nous besoin d'un volume pour être attaché à notre conteneur postgres ?
Nous avons besoin d'un volume parce que les conteneurs Docker sont éphémères par défaut. Si le conteneur de base de données est détruit (ce qui arrive lors des mises à jour ou des nettoyages), toutes les données écrites à l'intérieur sont perdues. Le volume permet de persister les données sur le disque hôte, dissociant ainsi les données du cycle de vie du conteneur.


1-3 Documentez votre conteneur de base de données essentiels : commandes et Dockerfile.

Dockerfile : FROM postgres:17.2-alpine COPY ./init/ /docker-entrypoint-initdb.d/
Ilutilise une image légère et assure l'exécution des scripts SQL d'initialisation au premier démarrage.

docker run : docker run --name postgres-db -d --network app-network -e POSTGRES_USER=usr -v pgdata:/var/lib/postgresql/data my-postgres-db
il lance le conteneur en utilisant les variables d'environnement et assure la persistance via le volume pgdata.


1-4 Pourquoi avons-nous besoin d'une construction en plusieurs étapes? Et expliquez chaque étape de ce dossier.
La construction en plusieurs étapes (Multistage Build) est une bonne pratique pour réduire la taille de l'image finale et améliorer la sécurité.

Étape 1 (myapp-build) : Utilise une image complète (-jdk) contenant le JDK et Maven pour construire le code source et produire l'artefact (le .jar). Cette étape est jetable.

Étape 2 (Run stage) : Repart d'une image minimale (-jre) contenant seulement le Java Runtime (JRE). Elle utilise COPY --from=myapp-build pour copier uniquement le fichier .jar compilé de l'étape 1. L'image finale ne contient pas les outils de build (Maven), ce qui réduit sa taille et sa surface d'attaque.

1-5 Pourquoi avons-nous besoin d'un proxy inversé?
Un proxy inversé agit comme un point d'entrée unique et essentiel pour l'application.

Sécurité (SSL/TLS) : Il gère le chiffrement (HTTPS) et la terminaison SSL, protégeant l'API backend interne des requêtes directes.

Routage Centralisé : Il permet de basculer intelligemment le trafic : servir une page statique (index.html) pour la racine et transférer les requêtes spécifiques (/departments/...) vers le conteneur backend sur un port interne (8080).

Performance : Il peut gérer la compression ou l'équilibrage de charge (Load Balancing) sur plusieurs backends.


1-6 Pourquoi docker-compose est-il si important?
Docker Compose est important car il est l'outil d'orchestration local des applications multi-conteneurs. Au lieu de commandes manuelles longues, il permet de déclarer toute l'architecture (services, dépendances, réseaux, volumes) dans un seul fichier. Ceci rend le déploiement reproductible pour tous les développeurs.


1-7 Documentez docker-compose les commandes les plus importantes.
docker compose up -d : Construit les images et démarre l'architecture complète en mode détaché.

docker compose down : Arrête et supprime les conteneurs et le réseau associés.

docker compose up --build : Force la reconstruction des images (utile après une modification du code).

docker compose ps : Liste les conteneurs en cours d'exécution dans le projet.


1-8 Documentez votre dossier docker-compose.
Le fichier docker-compose.yml définit la structure de l'application :

service : database, backend, httpd	
Liste et configure les conteneurs.

depends_on :	backend attend database	
Assure que les services démarrent dans le bon ordre de dépendance.

networks :	app-network	
Crée un réseau interne pour la communication entre services.

networks :app-network	
Crée un réseau interne pour la communication entre services.

ports : "80:80"	
Expose uniquement le port du proxy à l'hôte, laissant les autres services sécurisés en interne.

1-9 Documentez vos commandes de publication et vos images publiées dans dockerhub.
Connexion : docker login

Taguer : docker tag docker-tp-database blaina/tp-postgres:1.0 (Permet de renommer l'image locale pour le dépôt distant).

Pousser : docker push blaina/tp-postgres:1.0 (Envoie l'image au registre).

Images publiées : blaina/tp-postgres:1.0, blaina/tp-backend-api:1.0, blaina/tp-httpd-proxy:1.0.

1-10 Pourquoi mettons-nous nos images dans un dépôt en ligne?
Nous utilisons un dépôt en ligne (registre) pour la portabilité et l'industrialisation (CI/CD).

Partage  : Permet de partager l'application exacte avec d'autres membres de l'équipe (ou serveurs) avec la commande docker pull.

Fiabilité  : Le registre garantit que l'image utilisée en production est la version exacte qui a été testée.

Déploiement  : C'est le point de départ pour les systèmes d'orchestration (Kubernetes) qui tirent les images directement du registre.