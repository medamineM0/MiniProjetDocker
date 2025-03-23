<h1 style="border-bottom: none; text-align:center; padding: 30px 10px"><strong>Rapport du Mini-Projet Docker</strong></h1>

**Réalisé par** : MOUTAOUAKIL Mohamed Amine
<br><br>

# Introduction

Ce projet consiste à conteneuriser une application Python avec un frontend en PHP et un backend en Flask, en utilisant Docker. L'application, nommée "student_list", permet d'afficher la liste des étudiants avec leur âge.

# Objectifs

- **Améliorer le processus de déploiement** : Rendre le déploiement de l'application plus fiable et reproductible grâce à Docker.
- **Gestion des versions de l'infrastructure** : Utiliser Docker pour gérer les différentes versions de l'application et de ses dépendances.
- **Adopter les meilleures pratiques** : Implémenter une infrastructure Docker conforme aux standards modernes de développement et de déploiement.
- **Infrastructure as Code (IaC)** 

# Architecture
L'application "student_list" adopte l'architecture client-serveur et se compose de deux modules :
- **API REST** (avec authentification de base 
nécessaire) : envoie la liste souhaitée des étudiants à partir d'un fichier ```JSON```.
- **Interface web** : écrite en ```PHP``` & ```HTML``` et permet aux utilisateurs finaux d'obtenir la liste des étudiants. 

# Construction et Test d'API : Docker File
- Le ```Dockerfile``` est fichier permettant de créer une image ```Docker``` selon les besoins en éxecutant des commandes. 
```Dockerfile
FROM python:3.8-buster

LABEL maintainer="Med Amine Moutaouakil <amine20991@gmail.com>"

COPY student_age.py /

RUN apt-get update -y && apt-get install -y python3-dev libsasl2-dev libldap2-dev libssl-dev

COPY requirements.txt /

RUN pip3 install -r requirements.txt

RUN mkdir /data

VOLUME /data

COPY student_age.json /data/student_age.json

EXPOSE 5000

CMD [ "python3", "./student_age.py" ]
```
## Explications des commandes clées 


1. **Spécifier l'image parente** :

   ```Dockerfile
   FROM python:3.8-buster
   ```
2. **Mentionner le mainteneur** :

   ```Dockerfile
   LABEL maintainer="Med Amine Moutaouakil <amine20991@gmail.com>"
   ```
3. **Installation des packages** :

   ```Dockerfile
   RUN apt-get update -y && apt-get install -y python3-dev libsasl2-dev libldap2-dev libssl-dev
   ```
4. **Installation des modules pythons** :

   ```Dockerfile
   RUN pip3 install -r requirements.tx
   ```
5. **Montage de /data** :

   ```Dockerfile
   VOLUME /data
   ```
    Docker gérera ce répertoire comme un espace de stockage externe, indépendant du système de fichiers du conteneur.

6. **Exposition du port 5000** :

   ```Dockerfile
   EXPOSE 5000
   ```
    Définir le port par lequel on peut accéder à notre application conteneurisée.

7. **Exposition du port 5000** :

   ```Dockerfile
   CMD [ "python3", "./student_age.py" ]
   ```
    CMD correspondra aux commandes exécutées par le conteneur.

### Build Docker Image à partir Dockerfile
![Texte alternatif](screens\1.png)
![Texte alternatif](screens\2.png)

### Lancement du conteneur
![Texte alternatif](screens\3.png)

### On vérifie les logs et s'assure que le conteneur écoute et est prêt à répondre. 
![Texte alternatif](screens\4.png)

### On teste la réponse de l'API
![Texte alternatif](screens\5.png)


# Insfastructure as Code (IaC)
Maintenant on procède au déploiement de l'ensemble de l'application à l'aide de ```Docker Compose``` . On va configurer le fichier ```docker-compose.yml``` pour déploier deux services (Backend & Frontend).

- ```docker-compose.yml``` :

  ```yaml
    services:
        supmit_api:
            image: student_api
            container_name: student-container
            ports:
            - "5000:5000"
            volumes:
            - C:\Users\Admin\Desktop\student_list\simple_api\student_age.json:/data/student_age.json
            networks:
            - student_network

        website:
            image: php:apache
            container_name: supmit-website
            ports:
            - "8080:80"
            volumes:
            - ./website:/var/www/html
            environment:
            - USERNAME=root
            - PASSWORD=root
            depends_on:
            - supmit_api
            networks:
            - student_network

    networks:
        student_network:
            driver: bridge
  ```
<br>

### Explication du fichier `docker-compose.yml`

Le fichier `docker-compose.yml` est utilisé pour définir et orchestrer les services Docker nécessaires au fonctionnement de l'application. Il contient deux services principaux : **`supmit_api`** (backend) et **`website`** (frontend), ainsi qu'un réseau Docker pour permettre la communication entre ces services.

---

### **1. Service : `supmit_api` (Backend)**

Le service `supmit_api` représente le backend de l'application, qui est une API Flask pour gérer les données des étudiants.

- **`image: student_api`** :  
  Ce service utilise une image Docker nommée `student_api`, qui a été construite à partir d'un Dockerfile pour exécuter l'API Flask.

- **`container_name: student-container`** :  
  Le conteneur créé à partir de cette image sera nommé `student-container`. Cela permet de facilement identifier et gérer le conteneur.

- **`ports: "5000:5000"`** :  
  Le port `5000` du conteneur (port utilisé par l'API Flask) est exposé sur le port `5000` de l'hôte. Cela permet d'accéder à l'API via `http://localhost:5000`.

- **`volumes`** :  
  Le fichier `student_age.json` (contenant les données des étudiants) est monté depuis le chemin `C:\Users\Admin\Desktop\student_list\simple_api\student_age.json` sur l'hôte vers `/data/student_age.json` dans le conteneur. Cela permet à l'API d'accéder aux données des étudiants de manière persistante.

- **`networks`** :  
  Le service est connecté au réseau `student_network`, ce qui permet une communication fluide avec les autres services (comme le frontend).

---

### **2. Service : `website` (Frontend)**

Le service `website` représente le frontend de l'application, qui est une interface utilisateur en PHP exécutée avec Apache.

- **`image: php:apache`** :  
  Ce service utilise l'image officielle `php:apache`, qui fournit un environnement Apache préconfiguré pour exécuter des applications PHP.

- **`container_name: supmit-website`** :  
  Le conteneur créé à partir de cette image sera nommé `supmit-website`.

- **`ports: "8080:80"`** :  
  Le port `80` du conteneur (port par défaut d'Apache) est exposé sur le port `8080` de l'hôte. Cela permet d'accéder à l'interface utilisateur via `http://localhost:8080`.

- **`volumes`** :  
  Le répertoire `./website` (contenant les fichiers PHP de l'application) est monté dans `/var/www/html` du conteneur. Cela remplace le site web par défaut d'Apache par les fichiers de l'application.

- **`environment`** :  
  Deux variables d'environnement sont définies :  
  - `USERNAME=root` : Nom d'utilisateur pour l'authentification.  
  - `PASSWORD=root` : Mot de passe pour l'authentification.  
  Ces informations sont utilisées pour permettre à l'application web d'accéder à l'API.

- **`depends_on: supmit_api`** :  
  Ce service dépend du service `supmit_api`, ce qui garantit que l'API démarre avant le site web.

- **`networks`** :  
  Le service est connecté au réseau `student_network`, ce qui permet une communication fluide avec le backend.

---

### **3. Réseau : `student_network`**

Un réseau Docker nommé `student_network` est créé pour permettre la communication entre les services `supmit_api` et `website`.

- **`driver: bridge`** :  
  Le réseau utilise le pilote `bridge`, qui est le pilote par défaut pour les réseaux Docker. Cela permet aux conteneurs de communiquer entre eux tout en étant isolés du réseau externe.

<br>

On lance ```docker-compose.yml``` :

![Texte alternatif](screens\6.png)

Enfin, on accéde au site web ```http://localhost:8080.``` et on clique sur le bouton "List Student" :

![Texte alternatif](screens\7.png)

## Docker Registry

SUPMIT demande de déployer un registre privé et de stocker les images construites. Puis on va poussé les images Docker.
- ```docker-compose-registry.yml``` : 

  ```yaml 
    services:
        registry:
            image: registry:2
            container_name: registry
            ports:
            - "5001:5000" 
            volumes:
            - registry_data:/var/lib/registry  

        registry-ui:
            image: joxit/docker-registry-ui:1.5-static
            container_name: registry-ui
            ports:
            - "8082:80"  
            environment:
            - REGISTRY_TITLE=SUPMIT Registry  
            - REGISTRY_URL=http://registry:5000  
            depends_on:
            - registry  

    volumes:
        registry_data:  
  ```
    Puis on lance les commandes suivante :
  ```bash
  > docker-compose -f docker-compose-registry.yml up -d
  
  > docker tag student_api localhost:5001/student_api

  > docker push localhost:5001/student_api
  ```
  Enfin un registre privé sera créé, et notre image sera poussée.

  ![Texte alternatif](screens\8.png)

  On vérifie que l'image existe :

  ![Texte alternatif](screens\9.png)

# Conclusion
Ce TP nous a permis de maîtriser les bases de Docker, notamment la conteneurisation d'applications, l'utilisation de Docker Compose pour orchestrer plusieurs services, et la mise en place d'un registre Docker privé avec une interface web.