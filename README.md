# GPCroesus-Pilotes

# Contexte
Toutes les voitures sont sur la ligne de départ et la course est lancée! 

Par contre, cette course est un peu spéciale. Pour compléter un tour, le directeur de course envoie un message chaque 5 secondes à toutes les voitures et seulement les voitures retournant un message valide seront créditées d’un tour. Vous devrez donc mettre en place divers services AWS avant que votre voiture fasse des tours. Et attention aux imprévus!

La voiture avec le plus de tours complétés après 3 heures sera déclarée gagnante.

# Épreuves

## Informations utiles à savoir avant de commencer
- Compte AWS à utiliser: gpcroesus-2023-sandbox (560247168066)
  - Vous devriez être en mesure de vous y connecter via la console AWS avec votre compte AWS Croesus
- Vous aurex besoin de votre identifiant d'équipe (*teamId*) qui vous a été communiqué lorsque vous vous êtes inscrits à l'événement.

## Épreuve 1: Trouver le endpoint du directeur de course
La première étape est de trouver l’URL du directeur de course. Celle-ci sera utilisée pour enregistrer votre voiture et recevoir les messages utilisés pour compléter des tours de piste.
Le URL du endpoint est dans un fichier à l'intérieur du bucket S3 XXXYYYZZZ. À vous de le trouver!

Vous devez donc écrire une lambda (runtime de votre choix) qui va trouver le endpoint parmi tous les fichiers du bucket. Pour vous aider un peu, le nom du fichier contenant l’URL est disponible dans Parameter Store sous le path suivant: /grandprix/teams/*teamId*/challenge1/filename.

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
Afin de recevoir les messages du directeur de course pour compléter des tours, vous devrez mettre en place un service de voiture qui recevra les messages:
- Le service doit être déployé dans le subnet xxxx.
- Un ALB doit absolument être utilisé. Le endpoint du load balancer doit être envoyé au directeur de course.
- Le directeur enverra des payloads JSON à chaque 5 secondes à votre service, sur le port 12345:
  - Les messages seront envoyés avec un POST sur le chemin /startLap
  - Le payload contiendra un *lapId* que vous devrez utiliser pour récupérer les informations sur le lap en question
    - Vous devrez récépérer ces informations dans la table DynamoDB xxxyyyzzz et répondre correctement en fonction des détails indiqués pour le tour (voir plus bas)
- Il y a deux actions à faire pour répondre à un message:
  - Répondre HTTP 200 à l'appel à /startLap
  - Placer un message dans une queue SQS xxxyyyzzz

### Format du paylod JSON de la requête HTTP
    {
      "lapId": "abcde12345"
    }

### Détails sur les laps
Pour chaque lap, les champs suivants seront disponibles dans le table DynamoDB:
  - lapId: L'identifiant du tour. Utilisez celui reçu dans le requête HTTP pour trouver le bon enregistrement de tour.
  - lapStatus: Le statut du tour
    - DONE: Le tour est complété avec succès
    - PITSTOP: Vous devez faire un pit stop et répondre à une question
  - lapTime: Le temps prit pour le tour
  - lapPitStopQ: Si lapStatus == PITSTOP, la question à répondre pour sortir du pitStop

### Format de la réponse JSON à placer dans la queue SQS
    {
      "teamId": "Votre identifiant d'équipe",
      "lapId": "abcde12345",
      "lapTime": "1:20.559",
      "lapPitStopA": "La réponse à la question si lapStatus == PITSTOP. Sinon, omettre cet element"
    }

## Épreuve 4: Enregistrer votre voiture au directeur de course
Maintenant que vous avez l’URL de votre service de voiture, il est temps d’enregistrer votre voiture pour pouvoir commencer à accumuler des tours de piste!

Redéployez le service d'enregistremene utilisé à l'étape 2 en y ajoutant la variable d'environnement suivante:
- **CAR_SERVICE_URL**: Url du load balancer de votre service de voiture

## Épreuve 5: Recevoir les messages du directeur de course
Si vous avez réussi toutes les épreuves précédentes, votre service de voiture devrait maintenant recevoir de messages réguliers du directeur de course. 

Répondez correctement aux messages et cumulez plus de tours de pistes que les autres équipes afin de remporter le grand prix!
