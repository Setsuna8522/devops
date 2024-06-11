# TP1
## Base de données
Se mettre dans le répertoire `database/`.

### Création de l'environnement pour la base de données
1. Permettre les communications avec `adminer`
    - `docker network create app-network`
    - `docker run -p "8090:8080" --net=app-network --name=adminer -d adminer`

2. Créer le volume pour la persistence des données
    - `docker volume create volume-database`

3. Créer l'image docker
    - `docker build -t postgres-database .`

4. Créer le container sur le port 5432:5432
    - ~~`docker run -it -d -p 5432:5432 --name postgres-database postgres-database`~~  
    devient
    - `docker run --name postgres-database --network app-network -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -v volume-database:/var/lib/postgresql/data -p 5432:5432 -d postgres-database`

### Vérifications
1. Vérifier la connexion avec la base de données
    - `docker run -it --rm --network=host postgres:14.1-alpine psql -h localhost -U usr -d db`
    - mot de passe : `pwd`
    - lancer la requête `select * from students;`

2. Vérifier la persisitence des données
    - modifier les données (ex: `INSERT INTO students (department_id, first_name, last_name) VALUES (1, 'Eli', 'Copter2');`)
    - supprimer le container
        - `docker rm -f postgres-database`
    - relancer le container
    - vérifier que les données modifiées le sont toujours (ici, `select * from students;`)


## Backend
Se mettre dans le répertoire `backend/`.

### Création de l'environnement pour le backend
1. Créer l'image docker
    - `docker build -t spring-app .`

2. Créer le container sur le port 8080:8080
    - `docker run --name spring-app --network app-network -p 8080:8080 spring-app`

### Question 1.2
#### Bénéfice du multistage
Le multistage build sert à optimiser les dockerfiles tout en les maintenant facile à lire.
Dans le cas de Java, il permet de laisser le code source directement dans le container et lui laisser la gestion de la compilation, sans que ce soit à nous de compiler ce code et de l'insérer dans le container. Cela conduit à des images docker de taille plus petite et permet de séparer les environnements de compilation et d'exécution (JRE et JDK).

#### Explication Docker
Le dockerfile donné est séparé en deux parties : le build et le run.

##### Build stage
- On importe le JDK avec la ligne `FROM maven:3.8.6-amazoncorretto-17 AS myapp-build`.
- On initialise ensuite le répertoire source de l'application avec `ENV MYAPP_HOME /opt/myapp`, et on spécifie que `MYAPP_HOME` est le répertoire de travail du container grâce à `WORKDIR $MYAPP_HOME`.
- On copie les configurations du `pom` et les fichiers du répertoires avec les commandes `COPY`.
- Tous les paquets sont ensuite compilés avec `RUN mvn package -DskipTests`, qui génère un fichier `.jar`.

##### Run stage
- On importe le JRE pour lancer les applications Java avec `FROM amazoncorretto:17`.
- De la même manière que l'étape du build, on initialise le répertoire de travail avec la commande `ENV` et `WORKDIR`.
- Le fichier `.jar` est copié pour l'exécution grâce à `COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar`.
- On précise le point d'entrée de l'application : `ENTRYPOINT java -jar myapp.jar`.


## Serveur HTTP
Se mettre dans le répertoire `httpd/`.

### Création d'un serveur HTTP
1. Créer l'image docker
    - `docker build -t server .`

2. Créer le container sur le port 8080:8080
    - `docker run -d --name server -p 80:80 server`

- Pour avoir les statistiques du serveur
    -  `docker stats server`
- Pour inspecter le serveur
    -  `docker inspect server`
- Pour avoir les logs du serveur
    -  `docker logs server`

### Configuration du Reverse Proxy
1. Récupération de la configuration par défaut d'Apache depuis le container
    - `docker exec -it server cat /usr/local/apache2/conf/httpd.conf > httpd.conf`

2. Mise à jour de la configuration avec Reverse Proxy
    - ajout des lignes de configurations dans le fichier récupéré
    - mise à jour du fichier dockerfile pour le téléchargement vers le container

3. Reconstruction de l'image avec la bonne configuration du serveur
    - `docker build -t server-proxy .`
    - `docker run -d --name server-proxy -p 80:80 server-proxy`


## Liaison avec l'application
Se remettre à la racine du projet, un niveau au dessus des répertoires `backend/`, `database/` et `httpd/`.

- Pour construire l'image docker-compose et lancer le container
    - `docker-compose up --build`

Voir `./docker-compose.yml` pour la documentation du fichier.


## Publication sur Docker Hub
- Pour tag les différentes images créées par le docker compose
    - `docker tag adminer username/adminer:1.0`
    - `docker tag nttp1-backend username/nttp1-backend:1.0`
    - `docker tag nttp1-database username/nttp1-database:1.0`
    - `docker tag nttp1-httpd username/nttp1-httpd:1.0`

- Pour les publier sur le Docker Hub
    - `docker push username/adminer:1.0`
    - `docker push username/nttp1-backend:1.0`
    - `docker push username/nttp1-database:1.0`
    - `docker push username/nttp1-httpd:1.0`

- Remarque : `username` sera remplacé par le pseudo du compte, afin de faire un push sur le répertoire privé



# TP2
## Continuous Integration
### Question 2.1
Testcontainers est un framework qui se sert des containers Docker pour créer un environnement de test et faciliter les tests d'applications automatisés.

### Question 2.2
Voir `./.github/workflows/main.yml`.


## Continuous Delivery
test