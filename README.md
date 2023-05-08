# GPCroesus-Pilotes

# Contexte
Toutes les voitures sont sur la ligne de départ et la course est lancée! 

Par contre, cette course est un peu spéciale. Pour compléter un tour, le directeur de course envoie un message chaque 5 secondes à toutes les voitures et seulement les voitures retournant un message valide seront créditées d’un tour. Vous devrez donc mettre en place divers services AWS avant que votre voiture fasse des tours. Et attention aux imprévus!

La voiture avec le plus de tours complétés après 3 heures sera déclarée gagnante.

# Épreuves

## Informations utiles
- Compte AWS à utiliser: gpcroesus-2023-sandbox (560247168066)
  - Vous devriez être en mesure de vous y connecter via la console AWS avec votre compte AWS Croesus

## Épreuve 1: Trouver le endpoint du directeur de course
La première étape est de trouver l’URL du directeur de course. Celle-ci sera utilisée pour enregistrer votre voiture et recevoir les messages utilisés pour compléter des tours de piste.
Le URL du endpoint est dans un fichier à l'intérieur du bucket S3 XXXYYYZZZ. À vous de le trouver!

Vous devez donc écrire une lambda (runtime de votre choix) qui va trouver le endpoint parmi tous les fichiers du bucket. Pour vous aider un peu, le nom du fichier contenant l’URL est disponible dans Parameter Store sous le path suivant /xxx/yyy/zzz.

Attention! Le fichier peut seulement être lu par le service AWS Lambda utilisant le rôle d’exécution suivant XXXXX.

## Épreuve 2: Enregistrer votre voiture au directeur de course
Maintenant que vous avez trouvé l’URL du directeur de course, il est temps d’enregistrer la voiture afin de commencer à recevoir les messages!

Créez un service AWS ECS qui sera utilisé pour enregistrer votre voiture. Vous aurez besoin d’un token pour enregistrer ce service. Le token est dans un service AWS bien connu pour la gestion des secrets. 

Quelques informations utiles:
- Le service doit être déployé dans le subnet xxxxxx
- L’image ECR à utiliser est la suivante xxx
- Le service ECS doit utiliser les variables d'environnement suivantes pour enregistrer la voiture correctement:
  - RACE_DIRECTOR_URL: URL du directeur de course trouvé plus tôt
  - DRIVER_NAME: Nom de votre pilote/équipe
  - REGISTRATION_TOKEN: Token pour l’enregistrement de la voiture.


