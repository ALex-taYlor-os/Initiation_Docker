
Job 01

````
sudo apt-get install apt-transport-https ca-certificates curl gnupg2

````

Ajout 
````
sudo curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
````

````
cat /etc/apt/sources.list.d/docker.list
````

````
sudo apt install lsb-release
````

````
sudo echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
````

Installation de docker des composants 
````
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
````

Activer, voir sir le service tourne ..

````
sudo systemctl enable docker
````

````
sudo systemctl status docker
````

### La différence 

|                  | Image                    | Conteneur              |
|------------------|--------------------------|------------------------|
| **Nature**       | Modèle immuable          | Instance (processus)   |
| **Écriture**     | Read-only                | Layer R/W              |
| **Persistance**  | Stockée sur disque       | Éphémère sans volume   |
| **Multiplicité** | 1 image → N conteneurs   | 1 conteneur = 1 exécution |

### les instrcutions à connaitre
## 2. Les instructions
 
| Instruction | Rôle | Moment |
|-------------|------|--------|
| `FROM` | Image de base. Obligatoire, en premier (seul `ARG` peut la précéder) | Build |
| `ARG` | Variable disponible **uniquement pendant le build** | Build |
| `ENV` | Variable d'environnement **persistée** dans l'image | Build + Run |
| `RUN` | Exécute une commande. Chaque `RUN` crée une couche | Build |
| `COPY` | Copie des fichiers locaux dans l'image (**à privilégier**) | Build |
| `ADD` | Comme `COPY` + URL distantes + décompression d'archives | Build |
| `WORKDIR` | Répertoire de travail pour la suite, créé s'il n'existe pas | Build + Run |
| `USER` | Utilisateur pour les instructions suivantes et à l'exécution | Build + Run |
| `LABEL` | Métadonnées clé/valeur | — |
| `EXPOSE` | **Documente** un port. Ne le publie pas (il faut `-p` au `docker run`) | — |
| `VOLUME` | Déclare un point de montage pour des données persistantes | Run |
| `CMD` | Commande par défaut, **remplaçable** au `docker run` | Run |
| `ENTRYPOINT` | Exécutable **fixe** du conteneur (forçable avec `--entrypoint`) | Run |
| `HEALTHCHECK` | Commande périodique de vérification de santé | Run |
| `STOPSIGNAL` | Signal envoyé à l'arrêt du conteneur | Run |
| `SHELL` | Shell utilisé par `RUN` (défaut : `/bin/sh -c`) | Build |
| `ONBUILD` | Instruction déclenchée quand l'image sert de base à une autre | Build (image fille) |
 
### Les comparaisons à connaître
 
| | A | B |
|---|---|---|
| **ARG vs ENV** | `ARG` : build seulement, non persisté | `ENV` : conservé dans l'image, dispo à l'exécution |
| **COPY vs ADD** | `COPY` : simple et transparent | `ADD` : seulement pour une archive ou une URL |
| **CMD vs ENTRYPOINT** | `CMD` : commande/arguments par défaut, écrasables | `ENTRYPOINT` : exécutable fixe |
 


### les différentes commandes
| Catégorie | Commande | Description |
|-----------|----------|-------------|
| **Images** | `docker images` | Lister les images locales |
| | `docker pull nginx:alpine` | Télécharger une image depuis Docker Hub |
| | `docker pull ghcr.io/owner/image:tag` | Télécharger depuis un autre registry |
| | `docker push myregistry.com/myimage:v1` | Pousser une image vers un registry |
| | `docker inspect nginx:alpine` | Inspecter une image |
| | `docker commit container image` | Créer une image à partir de l'état d'un conteneur (rarement recommandé) |
| **Layers** | `docker history nginx:alpine` | Voir les layers d'une image |
| | `docker history --no-trunc nginx:alpine` | Layers avec commandes complètes et tailles |
| | `docker inspect nginx:alpine --format '{{json .RootFS.Layers}}'` | Afficher les empreintes des layers |
| **Conteneurs** | `docker run -d --name web nginx:alpine` | Créer et démarrer un conteneur (= create + start) |
| | `docker create --name myapp nginx:alpine` | Créer sans démarrer (état *created*) |
| | `docker start myapp` | Démarrer un conteneur créé ou arrêté |
| | `docker ps` | Lister les conteneurs en cours |
| | `docker ps -a` | Lister tous les conteneurs, y compris arrêtés |
| | `docker ps -a --filter "name=myapp"` | Filtrer par nom pour vérifier l'état |
| **Cycle de vie** | `docker pause myapp` | Suspendre le processus (running → paused) |
| | `docker unpause myapp` | Reprendre (paused → running) |
| | `docker stop myapp` | Arrêt propre : SIGTERM puis SIGKILL après 10 s |
| | `docker kill myapp` | Arrêt immédiat : SIGKILL |
| | `docker restart myapp` | Arrêt puis redémarrage (running → exited → running) |
| | `docker rm myapp` | Supprimer un conteneur (exited → removed) |
| | `docker rm -f myapp` | Forcer la suppression (ex. état *dead*) |
| | `docker container prune` | Faire le ménage des dockers arrêtés |
| **Diagnostic** | `docker inspect myapp --format '{{.State.ExitCode}}'` | Voir le code de sortie |
| | `docker logs myapp` | Voir les derniers logs |
| | `docker inspect myapp \| grep -i oom` | Vérifier un kill par manque de mémoire |
| | `dmesg \| grep -i "killed process"` | Vérifier l'OOM killer côté noyau |
| **Persistance** | `-v mydata:/var/lib/data` | Volume nommé (BDD, fichiers importants) |
| | `-v ./src:/app` | Bind mount (développement) |
| **Isolation** | `docker run --rm alpine ps aux` | Voir les PID depuis l'intérieur du conteneur |
| | `docker top <container_id>` | Voir les PID réels depuis l'hôte |
| | `docker run --init ...` | Utiliser tini comme PID 1 (gestion des signaux) |
| **Cgroups** | `docker run -d --memory 128m --cpus 0.5 nginx:alpine` | Limiter mémoire et CPU |
| | `--cpus 0.5` | Limiter à 50 % d'un CPU |
| | `--memory 256m` | Limiter la mémoire |
| | `--device-read-bps` | Limiter la lecture disque |
| | `--pids-limit` | Nombre max de processus |
| | `docker stats --no-stream` | Voir la consommation des conteneurs |
| **Daemon** | `sudo systemctl status docker` | Vérifier que le daemon tourne |
| | `sudo journalctl -u docker.service` | Voir les logs du daemon |
| | `ps aux \| grep dockerd` | Voir le processus dockerd |
| **Client** | `export DOCKER_HOST=tcp://remote-host:2376` | Piloter un daemon distant |
| | `cat ~/.docker/config.json` | Voir la configuration du client |
| **Sécurité** | `docker run -v /:/hostroot -it alpine chroot /hostroot` | Démo du risque : le groupe docker = root sur l'hôte |


### Job 03
````
FROM debian:bookworm-slim
LABEL Alex TAYLOR "alex.taylor@gmail.com"
ENTRYPOINT ["echo", "Hello World !"]
````

Pour lancer le fichier, 2 options : 

````
mv createDocker.txt Dockerfile
docker build -t mydocker:latest .
````

##### ou faire :  

````
docker build -f createDocker.txt -t mydocker:latest .
````

Le nom des images (ici mydocker) doit être toujours en minuscule.

### job 04

Pour un port SSH 5100 dans mon conteneur. 

Création de mon fichier Dockerfile 

> nano SSHDocker.txt

````
FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y openssh-server
RUN sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config

RUN mkdir /var/run/sshd

RUN echo 'root:root123' | chpasswd

RUN echo "Port 5100" >> /etc/ssh/sshd_config

EXPOSE 5100

ENTRYPOINT ["/usr/sbin/sshd", "-D"]
````

Ensuite je lance les commandes suivantes pour construire le conteneur, lancer en arrière plan avec -d 


````
docker build -f SSHDocker.txt -t sshdocker:latest .
docker run -d --name ssh1 -p 2222:5100 sshdocker
ssh -p 2222 root@localhost
````

je regarde avec docker ps tous les conteneur activés.

### On peut accéder au shell de notre conteneur sans passer par ssh et executé des commandes
````
docker exec -it  ssh1 ls | cd /home | ls
````