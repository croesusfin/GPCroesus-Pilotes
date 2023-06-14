# GPCroesus-Pilotes

# Contexte
Toutes les voitures sont sur la ligne de départ et la course est lancée! 

Par contre, cette course est un peu spéciale. Pour compléter un tour, le directeur de course vous enverra un message à toutes les voitures et celles-ci doivent retourner un message valide pour être crédité d’un tour. Vous devrez donc mettre en place divers services AWS avant que votre voiture fasse des tours. Et attention aux imprévus!

La voiture avec le plus de tours complétés après 3 heures sera déclarée gagnante.

# Épreuves

## Informations utiles à savoir avant de commencer
- La console AWS à utiliser: https://560247168066.signin.aws.amazon.com/console
- Vos identifiants, qui vous ont été communiqués avant ou au début du challenge:
  - Votre login: gpcroesus-teamN (*N* est votre identifiant d'équipe, i.e. *teamId*)
  - Votre mot de passe: *** 

## Épreuve 1: Trouver le endpoint du directeur de course

La première étape est de trouver l’URL du directeur de course. Celle-ci sera utilisée pour enregistrer votre voiture et recevoir les appels pour compléter des tours de piste.
L'URL du directeur de course est dans un fichier à l'intérieur du bucket S3 `gpcroesus-2023-team-<x>` (*x* est votre identifiant d'équipe, i.e. *teamId*). À vous de le trouver!

Vous devez donc modifier la fonction lambda qui a été préalablement déployée  pour vous afin d'aller chercher et récupérer l'URL caché dans le bucket. La lambda a comme nom `team-<x>-challenge1-lambda`.

Pour vous aider un peu, le nom du fichier contenant l’URL est disponible dans Parameter Store sous le path suivant: `/grandprix/teams/*teamId*/challenge1/filename`.

Attention! Le fichier et le paramètre SSM peuvent seulement être lus par la fonction Lambda!

## Épreuve 2: Enregistrer votre équipe au directeur de course
Maintenant que vous avez trouvé l’URL du directeur de course, il est temps d’enregistrer votre pilote pour pouvoir participer à la course!

Roulez une **tâche** ECS qui fera l'enregistrement de votre équipe auprès du directeur de course. Vous aurez besoin d’un token pour enregistrer ce service. Le token est dans un service AWS bien connu pour la gestion des secrets.

Quelques informations utiles:
- Un cluster ECS à préalablement été déployé pour vous. Celui devant être utilisé porte le nom `team-<x>-cluster`.
- Le type de lancement doit être `FARGATE`.
- Le _task definition_ doit être `team-<x>-task-definition`
- Les _subnets_ doivent être `team-subnet-a` et `team-subnet-b`
- Le groupe de sécurité doit être `team-<x>-ecs-security-group`
- _Public IP_ doit être mis à `off`
- L'image ECR est déjà déployée et prète à être utilisée. Aucune modification n'est requise.

La tâche ECS doit avoir les variables d'environnement suivantes pour enregistrer la voiture correctement.
  - **RACE_DIRECTOR_URL**: URL du directeur de course trouvé plus tôt.
  - **TEAM_ID**: Votre identifiant d'équipe.
  - **DRIVER_NAME**: Nom de votre pilote. Soyez original!
  - **REGISTRATION_TOKEN**: Token pour l’enregistrement de la voiture.

### Astuces

- Vous ne pourrez pas avoir accès au secret directement. Comment faire pour passer le secret au container?
- Assurez-vous de prendre la bonne version de la _task definition_
- Le directeur de course écoute sur le port 80.

## Épreuve 3: Enregistrer la voiture

Vous êtes presque prêt pour débuter la course! Vous devez maintenant enregister la voiture auprès du directeur de course pour compléter des tours. Nous avons déployé pour vous une lambda ayant comme nom `team-<x>-challenge3-lambda`. Prenez le _function URL_ de cette lambda et envoyez-la au directeur de course en utilisant la tâche ECS du challenge précédent.

Cette fois-ci, vous devez ajouter la variable d'environnement **CAR_SERVICE_URL** à la tâche avec l'URL de la lambda comme valeur pour que l'enregistrement se fasse correctement.

La voiture est enregistrée? Super! La course est débutée! Le directeur va maintenant vous envoyer des tours de piste à chaque 15 secondes que vous devrez recevoir, traiter et renvoyer.

Un tour de piste fonctionne comme suit:

- Le directeur va invoquer votre lambda et lui passer un *lapId* en paramètre.
- Vous devez récupérer ce lapId et trouver le temps du tour de piste.
- Le tour doit finalement être renvoyé au directeur de course via la _file d'arrivée_. Une file SQS.
- Si l'information retournée est valide, vous serez crédité d'un tour.

Vous devez donc:

- Modifier la lambda afin de recevoir et capturer le lapId envoyé par le directeur de course.
- Faire une requête sur une table DynamoDB `gpcroesus-laps` afin d'aller chercher le temps du tour.
- Générer un payload JSON pour la réponse.
- Pousser la réponse dans la queue SQS `arn:aws:sqs:ca-central-1:560247168066:gpcroesus-lap-queue`.


### Détails sur les laps

Pour chaque lap, les champs suivants sont disponibles dans la table DynamoDB:
  - ``lapId``: L'identifiant du lap. Utilisez celui reçu dans le requête HTTP pour trouver le bon enregistrement de lap.
  - ``lapStatus``: Le statut du lap
    - ``DONE``: Le lap est complété avec succès, vous devrez retourner le temps pris
    - ``PITSTOP``: Vous devrez faire un pit stop et répondre à une question!
  - ``lapTime``: Si lapStatus == DONE, le temps prit pour le lap tel que récupéré de la table DynamoDB
  - ``lapPitStopQ``: Si lapStatus == PITSTOP, la question à répondre pour sortir du pit stop

Note: Vous devez demander spécifiquement les attributs lapStatus, lapTime, lapPitStopQ lorsque vous faites votre get_item à DynamoDB, en utilisant lapId comme clé. Les scans sont refusés.

### Format de la réponse JSON à placer dans la queue SQS

```json
{
  "teamId": "Votre identifiant d'équipe",
  "lapId": "abcde12345",
  "lapTime": "1:20.559",                       # Si lapStatus == DONE
  "lapPitStopA": "La réponse à la question"    # Si lapStatus == PITSTOP
}
```
