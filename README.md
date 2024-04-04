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
- Assurez-vous d'avoir spécifié la région ca-central-1 pour que votre infrastructure AWS soit bien fonctionnelle.

# Pré-Course (9h30 - 10h)

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
- **KART_SERVICE_URL**: URL de la lambda que vous avez choisi à l'étape 1.
- **RACE_DIRECTOR_URL**: URL du directeur de course trouvé dans le bucket (à l'étape 2).

### Astuces

- Assurez-vous de prendre la bonne version de la _task definition_
- Le directeur de course écoute sur le port 80.

Le coureur est enregistré et la course n'est pas commencée? Super, vous pouvez déjà penser aux challenges!

Utilisez votre GUID que vous avez obtenu du directeur de course dans la lambda pour la rendre opérationnelle (il y a une variable pour cela).

# Course (10h - 12h30)

La course est débutée! Le directeur va maintenant vous envoyer des tours de piste à chaque 30 secondes.

## Challenges

À chaque tour de piste, le directeur de course appellera votre lambda à l'URL que vous avez enregistrée. Vous pourrez ainsi aller plus vite ou nuire aux autres coureurs si vous réussissez à répondre correctement aux challenges.

### Exemple d'appel avec challenge

```https://<URL_DE_VOTRE_LAMBDA>?challengeType=banana&challengeParameter=abcdef```

### Exemple de réponse attendue avec challenge

```json
{
  "team_guid": "Votre GUID d'équipe",
  "answer": "123abc"
}
```

### Liste des challenges
* * *

#### Banane :banana:
>Vous recevrez un paramètre en input une chaine de caractères.
>S'il s'agit d'un palindrome, retournez true.
>Sinon, retournez false.

#### Exemple de input avec ce challenge

* __`{challengeName: "banana",
    challengeParameter: "123321"
    }`__
#### Avantage 
``` Ralentit la vitesse du coureur devant vous (faible) ```
* * *

#### Carapace Rouge :turtle:
>Vous recevez un array de string représentant des nombres hexadecimaux.
>Vous devez trouver lesquels de ces nombres sont des anagrammes les uns des autres.
>Les nombres formant des anagrammes devront ensuite être additionnés puis retournés sous forme décimale pour former votre réponse.


#### Exemple de input avec ce challenge

* __`{challengeName: "redTurtle",
    challengeParameter: "string[]"
    }`__
#### Avantage 
``` Ralentit la vitesse du coureur devant vous (modéré) ```
* * *
#### Carapace Verte :turtle:
>Vous recevrez un paramètre en input qui sera un entier naturel entre 0 et 1000000.
>  * Si l'entier est divisible par 3, vous devrez renvoyer «Fizz».
>  * Si l'entier est divisible par 5, vous devrez renvoyer «Buzz».
>  * Si l'entier est divisible par 3 et par 5, vous devrez renvoyer «FizzBuzz».
>     Autrement, vous devrez renvoyer l'entier directement.


#### Exemple de input avec ce challenge
* __`{challengeName: "greenTurtle",
    challengeParameter: "999999999"
    }`__
#### Avantage 
``` Ralentit la vitesse du coureur devant (faible)``` 
* * *
#### Carapace Verte (triple) :turtle: :turtle: :turtle:

 >Vous recevrez un nombre premier en input. Il faut que vous renvoyiez le prochain nombre premier.


 #### Exemple de input avec ce challenge

* __`{challengeName: "tripleGreenTurtle",
    challengeParameter: "243522324"
    }`__
#### Avantage
```Ralentit la vitesse du coureur devant vous (modéré) ```
* * *
#### Champignon :mushroom:
>Un nombre entier vous est fourni, vous devez retourner l'élément correspondant à cet index dans la suite de fibonacci.


#### Exemple de input avec ce challenge
* __`{challengeName: "mushroom",
    challengeParameter: "124352"
    }`__

#### Avantage
``` Améliorer la vitesse de votre équipe (faible) ```
* * *
#### Champigion d'or :mushroom: 
>Deux nombres X et Y sont fournis. Vous devez répondre combien de bits sont différents entres les deux nombres.
>  * exemple #1:
>    * si x=1 et y=2 alors la représentation binaire est x=1 vs y=10, la réponse serait 2 bit de différent.
>  * exemple #2: 
>    * si x=6 et y=7 alors x=110 et y=111 la réponse est 1
>  * exemple #3:
>      * x=8008 donc 1111101001000, y=3337 donc 110100001001      la réponse est 4
>


#### Exemple de input avec ce challenge
* __`{challengeName: "goldenMushroom",
    challengeParameter: "{x:1, y:2}"
    }`__

#### Avantage
```Améliorer la vitesse de votre équipe (élevé) ```
* * *

#### Drift :racing_car:
>Vous devez traverser toutes les cases d'une grille en commençant par le point XY de départ et terminant par le point fourni. Générez une string qui contient vos déplacements. Seul les caractères V^<> sont acceptés, ils représentent des déplacements. Le nombre de déplacements devrait être exactement la taille de la grille (X*Y -1) Les dimensions de la grille sont toujours des nombres pairs de 2 à 100. Les points de départ et de fin se trouvent toujours dans un coin. On ne demande jamais de traverser la grille en diagonal: le point de fin sera dans un coin adjacent. Voici comment les positions de la grille fournie sont imaginées pour une grille 4x4:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;``` 1,1  2,1  3,1  4,1 ```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```1,2  2,2  3,2  4,2 ```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;``` 1,3  2,3  3,3  4,3 ```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;``` 1,4  2,4  3,4  4,4 ```

> Pour cette grille, si le point de départ est 4,1 et le point d'arrivée 1,1 voici une réponse valide
> vvv<^^^<vvv<^^^

#### Exemple de input avec ce challenge
* __`{challengeName: "drift",
    challengeParameter: "{width: 4, height:4, startPositionX: 4, startPositionY: 1, finishPositionX: 1, finishPositionY: 1}"
    }`__

#### Avantage
``` Améliorer la vitesse de votre equipe (élevé) ```
* * *

#### Étoile :star:
>Vous recevrez une chaine de caractères représentant:
>       * un courriel (ex: allo@gmail.com), 
>       * un code postal (ex: H1A 1Z9)
>       * ou un numéro de téléphone (ex: (450) 662-6101).
>     Vous devez renvoyer une regex qui permet de valider cette chaine de caractères.    

#### Exemple de input avec ce challenge

* __`{challengeName: "star",
    challengeParameter: "(514) 254-6011"
    }`__
#### Avantage 
``` Améliorer la vitesse de votre équipe (élevé) ```
* * *


## Leaderboard Grafana

https://croesus.grafana.net/d/e732b456-a8ca-4f7c-810d-010586b97c96/hackathon-croesus-2024-mario-kart
