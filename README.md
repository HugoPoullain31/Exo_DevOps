Poullain Hugo
DevOps - Exercices Docker


Partie 1 — Les bases (Exercices 1 à 5) 
Exercice 1 — Premier contact avec Docker 

1.3. Vérifiez que le conteneur tourne. Quelle commande permet de lister uniquement les conteneurs en cours d'exécution ?
La commande est docker ps, elle permet de lister .

1.4. Ouvrez http://localhost:8080 dans votre navigateur (ou avec curl). Que voyez-vous ?
On voit la page d'accueil par défaut du serveur web Nginx (affichant "Welcome to nginx!").

1.6. Arrêtez le conteneur mon-nginx sans le supprimer. Puis listez tous les conteneurs (y compris arrêtés). Quelle est la différence avec la commande de la question 1.3 ?
Les commandes sont docker stop mon-nginx puis docker ps -a. La différence est que docker ps liste uniquement les conteneurs actifs, tandis que docker ps -a liste tous les conteneurs, y compris ceux qui sont arrêtés (Exited).

1.8. Quelle commande aurait permis de lancer le conteneur de façon à ce qu'il soit automatiquement supprimé à l'arrêt ?
Il aurait fallu utiliser l'option --rm lors du lancement. Voici la commande : docker run --rm -d -p 8080:80 --name mon-nginx nginx:alpine.

	Exercice 2 — Construire sa première image avec un            Dockerfile 

2.5. Listez les images locales. Quelle est la taille de mon-site:v1 ? Comparez avec nginx:alpine .
Commande : docker image ls. La taille de l'image mon-site:v1 est quasiment identique (très légèrement supérieure) à celle de l'image de base nginx:alpine. Cette petite différence correspond exactement au poids de notre fichier index.html que nous avons rajouté.

2.6. Inspectez les layers de l'image avec docker history mon-site:v1 . Combien de layers ont été ajoutés par rapport à l'image de base ?
Commande : docker history mon-site:v1. Il y a 2 layers qui ont été ajoutés par rapport à l'image de base. Ils correspondent aux instructions COPY et EXPOSE présentes dans le Dockerfile.

2.7. Modifiez index.html (changez le ). Reconstruisez l'image avec le tag mon-site:v2 . Quelle étape a été rechargée depuis le cache ? Quelle étape a été réexécutée ?
Commande de reconstruction : docker build -t mon-site:v2 .. Lors de la construction, l'étape FROM nginx:alpine a été rechargée depuis le cache. En revanche, l'étape COPY a été réexécutée. En effet, Docker a détecté que le fichier source index.html avait été modifié, ce qui a invalidé le cache pour cette couche précise









Exercice 3 — Volumes et persistance des données 


3.1. Lancez un conteneur alpine en mode interactif ( -it ) avec --rm . À l'intérieur, créez le fichier /data/test.txt avec le contenu "bonjour" . Quittez ( exit ). Relancez un nouveau conteneur alpine . Le fichier existe-t-il ? Expliquez pourquoi.
Non, le fichier n'existe plus dans le nouveau conteneur. Le système de fichiers d'un conteneur est éphémère : toute donnée écrite à l'intérieur disparaît définitivement à la suppression du conteneur.

3.2. Bind mount : Créez un dossier exercice-3/html/ sur votre machine. Placez-y un fichier index.html. Lancez un conteneur nginx:alpine en montant ce dossier dans /usr/share/nginx/html avec -v . Modifiez index.html sur votre machine (sans redémarrer le conteneur) et rafraîchissez le navigateur. Que constatez-vous ?
On constate que la page web se met à jour immédiatement après le rafraîchissement. Cela s'explique car le conteneur lit directement et en temps réel le dossier de la machine hôte grâce au bind mount

3.5. Lancez un nouveau conteneur alpine (différent du précédent) avec le même volume monté. Le fichier /data/persistant.txt existe-t-il ? Qu'est-ce que cela démontre ?
Oui, le fichier /data/persistant.txt existe toujours. Cela démontre que les volumes nommés sont persistants et survivent indépendamment à la destruction des conteneurs.

3.6 Listez les volumes Docker existants. Où Docker stocke-t-il physiquement ce volume sur votre machine ?
La commande est docker volume ls. Docker stocke physiquement ces volumes dans le répertoire /var/lib/docker/volumes/. 
3.7. Supprimez le volume mes-donnees . Quelle précaution faut-il prendre avant de le supprimer ?
Docker volume rm mes-donnees. La précaution indispensable avant de supprimer un volume est de s'assurer qu'aucun conteneur (même arrêté) ne l'utilise encore et que les données stockées ne sont plus utiles, car la suppression est irréversible.

Exercice 4 — Réseaux Docker 

4.1. Listez les réseaux Docker existants sur votre machine. Quels sont les trois réseaux créés par défaut ?
Commande : docker network ls. Les trois réseaux créés par défaut par Docker sont bridge, host et none.

4.5. Quittez client . Lancez un nouveau conteneur alpine nommé client-externe sans le connecter à mon-reseau (réseau par défaut). Essayez de joindre serveur-web par son nom. Que se passe-t-il ? Pourquoi ?
La requête échoue avec une erreur (type "bad address"). Cela se produit pour deux raisons : d'une part, ils ne sont pas sur le même réseau (ils sont donc isolés l'un de l'autre), et d'autre part, le réseau bridge par défaut de Docker ne propose pas la résolution DNS automatique par nom de conteneur.

4.6. Quelle commande permet de connecter client-externe à mon-reseau après son démarrage ?
Docker network connect mon-reseau client-externe. Cette commande permet d'attacher un conteneur en cours d'exécution à un réseau existant.



Exercice 5 — Containeriser un serveur Flask 

5.6. Relancez le conteneur sans passer APP_ENV . Quelle valeur s'affiche ? D'où vient-elle ?
La valeur qui s'affiche est "développement". Elle provient directement du code source app.py, plus précisément de la ligne os.environ.get("APP_ENV", "développement"), qui définit "développement" comme valeur par défaut si la variable d'environnement APP_ENV n'est pas trouvée.

5.7. Quelle est la taille de l'image flask-app:v1 ? Que pourrait-on faire pour la réduire davantage (donnez deux pistes) ? 

La commande docker image ls nous indique que la taille de l'image flask-app:v1 est de 197 MB.
Pour réduire davantage cette taille, on pourrait explorer deux pistes :
Utiliser un fichier .dockerignore : Cela permet d'éviter d'importer accidentellement des fichiers locaux inutiles dans l'image lors de l'instruction COPY . . (comme les environnements virtuels Python .venv, les dossiers de cache __pycache__ ou le dossier .git).
Utiliser le "multi-stage build" (build multi-étapes) : Cette technique consiste à utiliser une première image (builder) pour télécharger et compiler les dépendances, puis à ne copier que le résultat final (le strict nécessaire) dans une seconde image d'exécution beaucoup plus légère (comme alpine, qui ne pèse que 13 MB comme on le voit dans notre liste d'images).

