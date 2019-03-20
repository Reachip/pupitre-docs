# Installer l'ensemble du projet sur un Raspberry PI

**NOTE :** Si les instructions ci-dessous vous posent problème, vous pouvez passer cette page et mettre en place pas à pas le projet.

Ce gitbook explique pas à pas via les présentations des différentes technologies et la présentation des étapes de l'élaboration du projet comment mettre en place le projet.

En contre partie, cette page propose pour les utilisateurs avancées l'ensemble des commandes et configuration à faire afin d'implémenter ce projet.

## Dépendances

### Depuis pypi.org

* pyserial 
* RPi.GPIO 
* omxplayer-wrapper

```bash
sudo pip3 install pyserial RPi.GPIO omxplayer-wrapper
```

### Sur le Raspberry PI cible

* Python &gt;= 3.5
* omxplayer

```bash
sudo apt update; sudo apt upgrade 
sudo apt install omxplayer python3
```

## Installation

```bash
git clone https://github.com/Reachip/projet-theatre
sudo python3 projet-theatre/pypitre
```

Utilisez le fichier `config.json` afin d'indiquer au programme les trois PIN GPIO qui serviront à allumer les lumières et ou trouver le fichier qui jouera la musique.

## Mise en place d'un deamon 

La mise en place de deamon est [disponible ici](technologies-utilisees/systeme-dexploitation-unix/systemd.md).  
