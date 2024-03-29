# Hackathon Croesus 2024 - Mario Kart - Circuit Croesus

# Contexte

La piste est prête, _rainbow road_ n'a rien à envier au Circuit Croesus.

Le tout se déroulera sur 2 étapes, une pré-course (enregistrement de votre coureur) et la course (remplie
d'opportunités)!

La voiture avec le plus de tours complétés à la fin de l'épreuve sera déclarée gagnante.

Choisissez votre coureur, enregistrez vous, la course débutera dans 30 minutes!

## Informations utiles à savoir avant de commencer

- La console AWS à utiliser: `https://637423207885.signin.aws.amazon.com/console`
- Vos identifiants, qui vous ont été communiqués avant ou au début du challenge:
    - Votre login: `hackathoncroesus-team<X>` (*x* est votre identifiant d'équipe, i.e. *teamId*)
    - Votre mot de passe: ***

# Pré-Course (9h - 9h30)

## Étape 1: Trouver la lambda utilisée par votre coureur (son kart)

Nous avons déployé pour vous une Lambda Python pour avancer dans la course:

- Lambda en Python (`hackathoncroesus-team<X>-step1-lambda-python-kart`)

Vous aurez besoin de coder quelques petites fonctions afin d'obtenir des bonis!

Notez le _function URL_ de la lambda, il sera requis à l'étape 3.

## Étape 2: Trouver le endpoint du directeur de course

La deuxième étape est de trouver l’URL du directeur de course. Il sera utilisé pour vous inscrire dans la course.

L'URL du directeur de course est dans un fichier à l'intérieur du bucket
S3 `hackathoncroesus-team<X>-step2-bucket` (`<X>` est votre identifiant d'équipe, i.e. *teamId*). À vous de le trouver!

Vous devez donc modifier la fonction lambda qui a été préalablement déployée pour vous afin d'aller chercher et
récupérer l'URL caché dans le bucket. La lambda a comme nom `hackathoncroesus-team<X>-step2-lambda`.

Pour vous aider un peu, le nom du fichier contenant l’URL est disponible dans Parameter Store sous le path
suivant: `/hackathoncroesus-team<X>-step2/filename`.

Attention! Le fichier et le paramètre SSM peuvent seulement être lus par la fonction Lambda!

## Étape 3: Enregistrer votre coureur au directeur de course

Maintenant que vous avez trouvé l’URL du directeur de course et choisi votre cheval de bataille, il est temps
d’enregistrer votre coureur et son kart pour pouvoir participer à la course!

Roulez une **tâche ECS*** qui fera l'enregistrement de votre coureur auprès du directeur de course.

Quelques informations utiles:

- Un cluster ECS à préalablement été déployé pour vous. Celui devant être utilisé porte le
  nom `hackathoncroesus-team<X>-cluster`.
-
    - Le _task definition_ doit être `hackathoncroesus-team<X>-task-definition`
- Le type de lancement doit être `FARGATE`.
- Les _subnets_ doivent être `hackathoncroesus-team<X>-subnet-a` et `hackathoncroesus-team<X>-subnet-b`
- Le groupe de sécurité doit être `hackathoncroesus-team<X>-ecs-security-group`
- _Public IP_ doit être mis à `off`
- L'image ECR est déjà déployée et prète à être utilisée. Aucune modification n'est requise.

La tâche ECS doit avoir les variables d'environnement suivantes pour enregistrer la voiture correctement.

- **TEAM_ID**: Votre identifiant d'équipe.
- **DRIVER_NAME**: Nom de votre coureur. Respectez la thématique! (Attention, premier arrivé premier servi!)
- **RACE_DIRECTOR_URL**: URL du directeur de course trouvé à l'étape 1.
- **KART_SERVICE_URL**: URL de la lambda que vous avez choisi à l'étape 2.

### Astuces

- Assurez-vous de prendre la bonne version de la _task definition_
- Le directeur de course écoute sur le port 80.

Le coureur est enregistré et la course n'est pas commencée? Super, vous pouvez déjà penser aux challenges!

# Course (9h30 - 12h)

La course est débutée! Le directeur va maintenant vous envoyer des tours de piste à chaque 30 secondes.

## Tours de piste

Un tour de piste fonctionne comme suit:

- Le directeur va invoquer votre lambda et lui passer un *LapId* en paramètre.
- Vous devez récupérer ce lapId et l'afficher dans la réponse en format JSON.
- Si l'information retournée est valide, vous aurez fait un tour, aussi simple que cela!

### Exemple d'appel

```https://<URL_DE_VOTRE_LAMBDA>?lapId=abcde12345```

### Exemple de réponse attendue

```json
{
  "teamId": "Votre identifiant d'équipe",
  "lapId": "abcde12345"
}
```

Vous devez donc:

- Modifier la lambda afin de recevoir et capturer le lapId envoyé par le directeur de course.
- Générer un payload JSON pour la réponse.

## Challenges

Il est très possible, qu'en plus du *LapId*, le directeur de course vous offre la chance de jouer un bonus! Ceci est
fait de façon aléatoire mais équitable!

Chaque challenge représente une fonction à compléter dans la Lambda qui vous a été fournie. La puissance du bonus est
directement en lien avec la difficulté de la fonction à compléter.

### Exemple d'appel avec challenge

```https://<URL_DE_VOTRE_LAMBDA>?lapId=abcde12345&challengeType=banana&challengeInput=abcdef```

### Exemple de réponse attendue avec challenge

```json
{
  "teamId": "Votre identifiant d'équipe",
  "lapId": "abcde12345",
  "challengeType": "banana",
  "challengeOutput": "fedcba"
}
```

### Liste des challenges

#### Banane

Bloque le prochain tour du coureur derrière vous

* `challengeType=banana`
* `challengeInput=...`

#### Banane (triple)

Bloque le prochain tour des 3 coureurs derrière vous

* `challengeType=banana-triple`
* `challengeInput=...`

#### Carapace Rouge

Bloque le prochain tour du coureur devant vous

* `challengeType=red-shell`
* `challengeInput=...`

#### Carapace Rouge (triple)

Bloque le prochain tour des 3 coureurs devant vous

* `challengeType=red-shell-triple`
* `challengeInput=...`

#### Carapace Verte

À une chance sur 2 de bloquer le prochain tour du coureur devant vous

* `challengeType=green-shell`
* `challengeInput=...`

#### Carapace Verte (triple)

À une chance sur 3 de bloquer le prochain tour des 3 coureurs devant vous

* `challengeType=green-shell-triple`
* `challengeInput=...`

#### Carapace Bleue

Bloque les 5 prochains tours du coureur en tête!

* `challengeType=blue-shell`
* `challengeInput=...`

#### Champignon

Vous fait faire 2 tours au lieu d'un seul

* `challengeType=mushroom`
* `challengeInput=...`

#### Éclair

Vous fait faire 3 tours au lieu d'un seul ET bloque le prochain tour de TOUS les autres coureurs!

* `challengeType=lightning`
* `challengeInput=...`

#### Étoile

Vous fait faire 5 tours au lieu d'un seul!

* `challengeType=starpower`
* `challengeInput=...`

## Leaderboard Grafana

https://croesus.grafana.net/<UPDATE_ME>
