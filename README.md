# GPCroesus-Pilotes

# Contexte
Toutes les voitures sont sur la ligne de départ et la course est lancée! 

Par contre, cette course est un peu spéciale. Pour compléter un tour, le directeur de course envoie un message chaque 5 secondes à toutes les voitures et seulement les voitures retournant un message valide seront créditées d’un tour. Vous devrez donc mettre en place divers services AWS avant que votre voiture fasse des tours. Et attention aux imprévus!

La voiture avec le plus de tours complétés après 3 heures sera déclarée gagnante.

# Épreuves

## Informations utiles
- Compte AWS à utiliser: gpcroesus-2023-sandbox (560247168066)
  - Vous devriez être en mesure de vous y connecter via la console AWS avec votre compte AWS Croesus
- VPC, ...

## Épreuve 1: Trouver le endpoint du service d'enregistrement
La première étape est de trouver l’URL du du service d'enregistrement. Celle-ci sera utilisée pour enregistrer votre voiture et recevoir les messages utilisés pour compléter des tours de piste.
Le URL du endpoint est dans un fichier à l'intérieur du bucket S3 XXXYYYZZZ. À vous de le trouver!

Vous devez donc écrire une lambda (runtime de votre choix) qui va trouver le endpoint parmi tous les fichiers du bucket. Pour vous aider un peu, le nom du fichier contenant l’URL est disponible dans Parameter Store sous le path suivant /xxx/yyy/zzz.

Attention! Le fichier peut seulement être lu par le service lambda utilisant le rôle d’exécution suivant XXXXX.

