# GPCroesus-Pilotes

# Contexte
Toutes les voitures sont sur la ligne de départ et la course est lancée! 

Par contre, cette course est un peu spéciale. Pour compléter un tour, le directeur de course envoie un message chaque 5 secondes à toutes les voitures et seulement les voitures retournant un message valide seront créditées d’un tour. Vous devrez donc mettre en place divers services AWS avant que votre voiture fasse des tours. Et attention aux imprévus!

La voiture avec le plus de tours complétés après 3 heures sera déclarée gagnante.

# Épreuves

## Informations utiles à savoir avant de commencer
- La console AWS à utiliser: https://560247168066.signin.aws.amazon.com/console
- Vos identifiants, qui vous ont été communiqués avant ou au début du challenge:
  - Votre login: gpcroesus-teamN (*N* est votre identifiant d'équipe, i.e. *teamId*)
  - Votre mot de passe: *** 

## Épreuve 1: Trouver le endpoint du directeur de course
La première étape est de trouver l’URL du directeur de course. Celle-ci sera utilisée pour enregistrer votre voiture et recevoir les appels pour compléter des tours de piste.
Le URL du directeur de course est dans un fichier à l'intérieur du bucket S3 gpcroesus-2023-team-N (*N* est votre identifiant d'équipe, i.e. *teamId*). À vous de le trouver!

Vous devez donc écrire une Lambda en Python (et probablement utiliser le SDK AWS PYthon Boto3, https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) qui va trouver l'URL 
caché dans un des fichiers du bucket S3. Pour vous aider un peu, le nom du fichier contenant l’URL est disponible dans Parameter Store sous le path suivant: /grandprix/teams/*teamId*/challenge1/filename.

Attention! Le fichier et le paramètre peuvent seulement être lus par la ressource Lambda créé d'avance pour vous, soit team-N-challenge1-lambda (*N* est votre identifiant d'équipe, i.e. *teamId*).

## Épreuve 2: Enregistrer votre équipe au directeur de course
Maintenant que vous avez trouvé l’URL du directeur de course, il est temps d’enregistrer votre équipe pour pouvoir participer à la course!

Créez un service AWS ECS qui sera utilisé pour enregistrer votre équipe. Vous aurez besoin d’un token pour enregistrer ce service. Le token est dans un service AWS bien connu pour la gestion des secrets. 

Quelques informations utiles:
- Le service doit être déployé dans le subnet team-N-subnet-a ou team-N-subnet-b (*N* est votre identifiant d'équipe, i.e. *teamId*)
- L’image ECR à utiliser est la suivante: gpcroesus-challenge2-repo:latest
- Le service ECS doit utiliser les variables d'environnement suivantes pour enregistrer la voiture correctement:
  - **RACE_DIRECTOR_URL**: URL du directeur de course trouvé plus tôt
  - **TEAM_ID**: Votre identifiant d'équipe
  - **DRIVER_NAME**: Nom de votre pilote/équipe
  - **REGISTRATION_TOKEN**: Token pour l’enregistrement de la voiture.

## Épreuve 3: Créer votre service de voiture
Afin de recevoir les appels du directeur de course afin de compléter des tours, vous devrez mettre en place un service de voiture qui recevra les appels:
- Le service doit être implémenté dans une Lambda en Python. Utiliser le squelette team-N-challenge3-lambda (*N* est votre identifiant d'équipe, i.e. *teamId*))
- Le directeur enverra des tours à l'URL de votre Lambda (nommé **CAR_SERVICE_URL** dans le reste du document) à chaque 15 secondes environ:
  - Les appels seront envoyés avec un GET comme ceci: **CAR_SERVICE_URL**/startLap/?LapId=<lapId>
  - Le payload contiendra un *lapId* que vous devrez utiliser pour récupérer les informations sur le lap en question
    - Vous devrez récupérer ces informations dans la table DynamoDB gpcroesus-laps et répondre correctement en fonction des détails indiqués pour le tour (voir plus bas)
- Il y a deux actions à faire pour répondre à un message:
  - Répondre HTTP 200 à l'appel à /startLap
  - Placer un message de réponse dans la queue SQS gpcroesus-lap-queue

### Détails sur les laps
Pour chaque lap, les champs suivants sont disponibles dans la table DynamoDB:
  - **lapId**: L'identifiant du lap. Utilisez celui reçu dans le requête HTTP pour trouver le bon enregistrement de lap.
  - **lapStatus**: Le statut du lap
    - **DONE**: Le lap est complété avec succès, vous devrez retourner le temps pris
    - **PITSTOP**: Vous devrez faire un pit stop et répondre à une question!
  - **lapTime**: Si lapStatus == DONE, le temps prit pour le lap tel que récupéré de la table DynamoDB
  - **lapPitStopQ**: Si lapStatus == PITSTOP, la question à répondre pour sortir du pit stop

Note: Vous devez demander spécifiquement les attributs lapStatus, lapTime, lapPitStopQ lorsque vous faites votre get_item à DynamoDB, en utilisant lapId comme clé.

### Format de la réponse JSON à placer dans la queue SQS
    {
      "teamId": "Votre identifiant d'équipe",
      "lapId": "abcde12345",
      "lapTime": "1:20.559",                       # Si lapStatus == DONE
      "lapPitStopA": "La réponse à la question"    # Si lapStatus == PITSTOP
    }

## Épreuve 4: Enregistrer votre voiture au directeur de course
Maintenant que vous avez l’URL de votre Lambda, il est temps d’enregistrer votre voiture pour pouvoir commencer à accumuler des tours de piste!

Redéployez le service d'enregistrement utilisé à l'étape 2 en y ajoutant la variable d'environnement suivante:
- **CAR_SERVICE_URL**: URL de votre Lambda pour le challenge 3.

## Épreuve 5: Recevoir les appels du directeur de course
Si vous avez réussi toutes les épreuves précédentes, votre service de voiture devrait maintenant recevoir de appels réguliers du directeur de course. 

Prenez la bonne action pour chaque lap et cumulez plus de tours de pistes que les autres équipes afin de remporter le grand prix!
