---
description: >-
  Une page permettant de faire part des problèmes rencontrés lors du début
  jusqu'à la fin du projet.
---

# Problème\(s\) rencontré\(s\)

## La fonction random\(\) de la librairie standard d'Arduino

Aussi étrange que cela puisse paraître, Arduino ne générait pas de nombre aléatoire d'une échelle de zéro à trois. N'ayant pas trouvé d'explications rationnel à ce problème, nous avons décidé de générer un nombre aléatoire grâce à cette fonction mais cette fois de zéro à dix en adaptant ce comportement l'algorithme de gestion des leds, ce qui a fonctionné.

## Le module Grove MP3 v2.0 pour Arduino

Dû à un manque critique de documentation officiel, l'implémentation de se module s'est avéré être un échec. En effet, les sources de documentation improvisées nous ont guidés vers la composition du schéma électrique mais la mise en place de la partie programmation manquait de source et semblait archaïque ou point de manipuler des données en hexadécimal.

Il a fallu abandonner nos travaux sur l'Arduino et s’intéresser au Raspberry PI afin de continuer le projet qui requiert une prise en charge de l'audio.

## La prise en charge du Raspberry PI par un écran VGA standard

La solution à ce problème s'est avéré plus facile qu'elle en avait l'air. Après plus d'une heure d'investigation, il fallait finalement changer une ligne du fichier de configuration qui se trouvait à la racine de la carte SD ou se trouvait Raspbian.

## L’exécution concurrente entre deux tâches

Le pupitre doit pouvoir gérer l'algorithme de lumières, dit l'éclairage, et la bande de son de la pièce de théâtre simultanément.

A partir de se postulat nous avons donc remplacé l'Arduino au profit du Raspberry PI car en effet, Arduino ne supporte aucun design-pattern lié à la concurrence. En passant par les threads voir même l’asynchronisme, aucune solution viable n'a été trouvé. Il a donc fallu se tourner vers un dispositif plus puissant, avec un environnement qui offre de plus amples libertés du point de vue des performances.

## Le choix d'une technologie afin de jouer un fichier audio sur le Raspberry PI

Voici la problématique qui a sans doute l'élément le plus retardant. En effet, il a fallu choisir la bonne librairie Python afin de gérer l'audio car certaines :

* Ne supportaient pas le support jack du Raspberry PI \(module playsound\) 
* Ne délivrait pas un son "correcte", il s'agissait de grésillement ou d'un son trop fort pour les notes graves. \(package pygame\) 
* Ne supportaient pas la mise en service dite "deamon" \(package pygame\) 
* Ne pouvaient pas "mettre en pause" l'échantillon audio sans passer par des pipes UNIX \(processus mpg123\) 
* Requéraient une dépendance à installer, ceci étant fastidieux car il fallait choisir une version exacte  de la dépendance en question \(pyglet\)

Il a donc fallu investiguer pendant un long moment pour pouvoir trouver une technologie adéquate, qu'elle soit d'un point de vue processus ou qu'il s'agisse d'un module/package Python.

Par conséquent, nous avons choisi un wrapper du programme omxplayer disponible dans les dépôts de PyPi \(support officiel pour les librairies et packages Python\). Ce dernier répond à toutes les problématiques énoncées ci-dessus tout en restant élégant du point de vue de son implémentation.

## L'incident "sur-voltage" avec le Raspberry PI 

Malgré le fait que la plupart des problématiques étaient dites "implementatif" ou logique d'un point de vue algorithmique, il s'est avéré que nous avons rencontré un problème d’inattention lors de la mise en place du circuit électrique. 

En effet nous avions alimenté le Raspberry PI avec un potentiel de 16 Volt au lieu de 5 Volt. 

Le problème a été résolu en jetant le défin raspberry PI afin de le remplacer.

