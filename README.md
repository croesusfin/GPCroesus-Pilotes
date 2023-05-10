# GPCroesus-Pilotes

# Contexte
Toutes les voitures sont sur la ligne de départ et la course est lancée! 

Par contre, cette course est un peu spéciale. Pour compléter un tour, le directeur de course envoie un message chaque 5 secondes à toutes les voitures et seulement les voitures retournant un message valide seront créditées d’un tour. Vous devrez donc mettre en place divers services AWS avant que votre voiture fasse des tours. Et attention aux imprévus!

La voiture avec le plus de tours complétés après 3 heures sera déclarée gagnante.

# Épreuves

## Informations utiles à savoir avant de commencer
- Compte AWS à utiliser: gpcroesus-2023-sandbox (560247168066)
  - Vous devriez être en mesure de vous y connecter via la console AWS avec votre compte AWS Croesus
- Vous aurex besoin de votre identifiant d'équipe (*team-id*) qui vous a été communiqué lorsque vous vous êtes inscrits à l'événement.

## Épreuve 1: Trouver le endpoint du directeur de course
La première étape est de trouver l’URL du directeur de course. Celle-ci sera utilisée pour enregistrer votre voiture et recevoir les messages utilisés pour compléter des tours de piste.
Le URL du endpoint est dans un fichier à l'intérieur du bucket S3 XXXYYYZZZ. À vous de le trouver!

Vous devez donc écrire une lambda (runtime de votre choix) qui va trouver le endpoint parmi tous les fichiers du bucket. Pour vous aider un peu, le nom du fichier contenant l’URL est disponible dans Parameter Store sous le path suivant /xxx/yyy/zzz.

Attention! Le fichier peut seulement être lu par le service AWS Lambda utilisant le rôle d’exécution suivant XXXXX.

## Épreuve 2: Enregistrer votre équipe au directeur de course
Maintenant que vous avez trouvé l’URL du directeur de course, il est temps d’enregistrer votre équipe pour pouvoir participer à la course!

Créez un service AWS ECS qui sera utilisé pour enregistrer votre équipe. Vous aurez besoin d’un token pour enregistrer ce service. Le token est dans un service AWS bien connu pour la gestion des secrets. 

Quelques informations utiles:
- Le service doit être déployé dans le subnet xxxxxx
- L’image ECR à utiliser est la suivante xxx
- Le service ECS doit utiliser les variables d'environnement suivantes pour enregistrer la voiture correctement:
  - **RACE_DIRECTOR_URL**: URL du directeur de course trouvé plus tôt
  - **TEAM_ID**: Votre identifiant d'équipe
  - **DRIVER_NAME**: Nom de votre pilote/équipe
  - **REGISTRATION_TOKEN**: Token pour l’enregistrement de la voiture.

## Épreuve 3: Créer votre service de voiture
...

## Épreuve 4: Enregistrer votre voiture au directeur de course
Maintenant que vous avez l’URL de votre service de voiture, il est temps d’enregistrer votre voiture pour pouvoir commencer à accumuler des tours de piste!

Redéployez le service d'enregistremene utilisé à l'étape 2 en y ajoutant la variable d'environnement suivante:
- **CAR_SERVICE_URL**: Url du load balancer de votre service de voiture

## Épreuve 5: Recevoir les messages du directeur de course
Si vous avez réussi toutes les épreuves précédentes, votre service de voiture devrait maintenant recevoir de messages réguliers du directeur de course. 

Cumulez plus de tours de pistes que les autres équipes afin de remporter le grand prix!
