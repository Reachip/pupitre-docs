# Docker

![](../.gitbook/assets/docker.jpg)

Docker est une technologie de virtualisation. Elle permet de créer un système isolé à travers le concept de container Docker et d'installer n'importe quel système d'exploitation sur ce dernier. 

L'avantage étant donc de pouvoir installé n'importe quel version de n'importe quel programme sans effet de bord. Il était donc important de s’intéresser à cette technologie afin de pouvoir installer les dépendances du projet et garder une certaine flexibilité dans la façon de maintenir les différents scripts du pupitre. 

## Docker sur un Raspberry PI 

Malheureusement, il s'est avéré que Docker n'était pas supporté par le modèle de Raspberry PI utilisé pour ce projet \(Raspberry PI 2 model B+ édition 2014\). 

En effet, Docker subissait une erreur de segmentation, aucune issue n'a été trouvé afin de résoudre ce problème à ce jour. 

## L'alternatif

Aucune alternatif n'a été trouvé afin de pallier à la problématique de Docker. Il a fallu s'adapter à cette contrainte tout gardant à l'esprit que notre système doit rester "propre" afin d'éviter tout effet de bord. 

