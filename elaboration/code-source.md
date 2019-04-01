# Code source

[Suite à plusieurs problématiques rencontrées ](probleme-s-rencontre-s.md)lors du projet, nous avons abandonné le développement du projet à l'aide de l'environnement Arduino au profit du Raspberry PI. **Ainsi, le code-source Arduino est malheureusement manquant.** 

## Avec le Raspberry PI

Le code source du projet pupitre implémenté en langage Python est disponible à cette adresse : https://github.com/projet-theatre/pupitre

**A ne pas oublier :** L'arboresence suivante est proposé afin de regrouper tous les fichiers nécessaires au projet 

```
pupitre
├── LICENSE
├── Makefile
├── README.md
├── __init__.py
├── __main__.py
├── config.json
└── core
    ├── __init__.py
    ├── event.py
    └── light.py
```

Le fichier Makefile est expliqué plus loin sur ce gitbook. Il n'est pas obligatoire mais il permet de gagner en productivité.

```js
// Fichier config.json 
// Il permet de livrer un programme générique,
// donc facilement personnalisable sans devoir éditer directement 
// l'implémentation en Python.
{
    // Les PIN en liaison avec les différents relais de lumières
    // Ici, il en existe trois mais il peux en avoir plus ou moins.
    "leds_handlers": [17, 20, 21], 

    // Le chemin absolu vers le fichier audio.
    "music_path": "/home/pi/buzzer.mp3",

    // Le nombre de fois ou les animations de lumières 
    // seront effectuées. 
    "animation_iterations": 5 
}
```

```python 

# Fichier __main__.py
import json
import time
import logging
from logging.handlers import RotatingFileHandler
from pathlib import Path

from omxplayer.player import OMXPlayer
import RPi.GPIO as GPIO

from core import event
from core import light

# Mise en place d'un fichier de log 
# afin d'obtenir un rapport d'anomalies 
# probables.
logger = logging.getLogger()
logger.setLevel(logging.DEBUG)

formatter = logging.Formatter("%(asctime)s :: %(levelname)s :: %(message)s")
file_handler = RotatingFileHandler("/home/pi/pupitre.log", "a", 1000000, 1)
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(formatter)
logger.addHandler(file_handler)

stream_handler = logging.StreamHandler()
stream_handler.setLevel(logging.DEBUG)
logger.addHandler(stream_handler)

logger.info("le script est initialisé")

event = event.Event()
event.start()


# Ouverture de fichier de configuration 
with open("/home/pi/pupitre/config.json") as json_file:
    global json_datas
    # On récupère dans un dictionnaire 
    # les pins chargés d'allumer les lumères 
    # puis le chemin absolu vers le fichier audio.
    json_datas = json.load(json_file) 

GPIO.setmode(GPIO.BCM)
[
    GPIO.setup(handler, GPIO.OUT, initial=GPIO.HIGH)
    for handler in json_datas["leds_handlers"]
]

logger.info("GPIO initialisés")


@event.on_with_just_sound
def with_just_sound():
  """ Quand l'utilisateur ne demande qu'à avoir du son ... """
  logger.info("demande de jouer le son")
  path = Path(json_datas["music_path"])
  OMXPlayer(path)
  logger.info("jouer le son fini")


@event.on_with_sound_and_leds
def with_sound_and_led():
  """ Quand l'utilisateur demande à avoir du son  et l'animation des lumières ... """
  logger.info("jouer le son et animer les lumières demandé")
  path = Path(json_datas["music_path"])
  OMXPlayer(path)
  light.animation(5, json_datas["leds_handlers"])
  logger.info("jouer le son et animer les lumières fini")


@event.on_with_leds
def with_leds():
  """ Quand l'utilisateur ne demande qu'à avoir l'animation des lumières ... """
  logger.info("animer les lumières")
  light.animation(json_datas["animation_iterations"], json_datas["leds_handlers"])
  logger.info("animer les lumières fini")


event.join()
```

```python 
# Fichier core/event.py
import logging
import threading
from logging.handlers import RotatingFileHandler

import serial
import RPi.GPIO as GPIO


class Event(threading.Thread):
    """ 
    Cette classe se sert du pattern dispatcher  
    afin de répondre à des événements. 
    
    Dans le cas présent, l'envoie de données par le terminal 
    mobile destinée au terminal bluetooth du Raspberry.
    """ 
    def __init__(self):
        threading.Thread.__init__(self)

        # Stockage des fonctions qui seront appellées si tel ou tel nombre est envoyé
        self.callbacks = {} 

        # Mise en place d'un système de log, qui s'avère important  
        self.logger = logging.getLogger()
        self.logger.setLevel(logging.DEBUG)

        formatter = logging.Formatter("%(asctime)s :: %(levelname)s :: %(message)s")
        file_handler = RotatingFileHandler("/home/pi/pupitre.log", "a", 1000000, 1)
        file_handler.setLevel(logging.DEBUG)
        file_handler.setFormatter(formatter)
        self.logger.addHandler(file_handler)

        stream_handler = logging.StreamHandler()
        stream_handler.setLevel(logging.DEBUG)
        self.logger.addHandler(stream_handler)

        try: # Tentative 
          # bluetooth_module fait référence au module bluetooth du Rasoberry 
          self.bluetooth_module = serial.Serial(
              port="/dev/ttyAMA0", # Définition du PIN on l'on va lire.
              baudrate=9600,
              parity=serial.PARITY_NONE,
              stopbits=serial.STOPBITS_ONE,
              bytesize=serial.EIGHTBITS,
              timeout=1,
          )

        except Exception as error: # Si une erreur se produit
          # On écrit sur le fichier de log
          self.logger.warning("Erreur concernant le serial du Raspberry : " + error)

    def on_with_sound_and_leds(self, callback):
        """ 
        Décorateur qui stockera la fonction passé en 
        appel de retour dans le dictionnaire. Cette 
        dernière pourra être appellé graçe à la clé b"2" 

        Quand la donnée "2" sera envoyée, on appellera 
        la fonction passer au décorateur.
        """

        # Quand la donnée "2" sera envoyée, on appellera 
        # la fonction passer au décorateur.
        self.callbacks[b"2"] = callback 
        return callback

    def on_with_just_sound(self, callback):
        """ 
        Décorateur qui stockera la fonction passé en 
        appel de retour dans le dictionnaire. Cette 
        dernière pourra être appellé graçe à la clé b"1" 

        Quand la donnée "1" sera envoyée, on appellera 
        la fonction passer au décorateur.
        """

        self.callbacks[b"1"] = callback
        return callback

    def on_with_leds(self, callback):
       """ 
        Décorateur qui stockera la fonction passé en 
        appel de retour dans le dictionnaire. Cette 
        dernière pourra être appellé graçe à la clé b"3" 

        Quand la donnée "3" sera envoyée, on appellera 
        la fonction passer au décorateur.
        """
    
        self.callbacks[b"3"] = callback
        return callback

    def run(self):
        """ 
        Méthode qui vérifiera indéfiniment la valeur envoyée 
        par bluetooth en passant cette dernière au dictionnaire 
        qui vérifiera si la valeur envoyée correspond à 
        une fonction. Si c'est le cas, on appelle la 
        fonction prévu à cet effet.
        """
        while True:
            try:
                req = self.bluetooth_module.readline()
                self.callbacks[req]()

            except KeyboardInterrupt:
                # On réinitialise les PIN du Raspberry
                GPIO.cleanup() 

            except KeyError:
                pass

```

```python
# Fichier core/light.py
import time
import RPi.GPIO as GPIO


def animation(period, handlers): 
    """ 
    Méthode générique réalisant l'animation 
    des lumières (effet mors avec les 
    lumières qui s'allument et s'éteignent très vite). 

    La periode correspond au nombre de fois que l'animation 
    sera réalisé puis handlers correspond à une liste de PIN
    ou sont branchées les lumières.
    """
    for loop in range(period):
        for handler in handlers:
            GPIO.output(handler, GPIO.LOW)
            time.sleep(0.1)

        for handler in reversed(handlers):
            GPIO.output(handler, GPIO.HIGH)
            time.sleep(0.1)
```

## Avec l'environnement Arduino

### Le programme en langage C

Voici le code source du programme téléverser dans la carte Arduino destiné à gérer les leds du pupitre et ainsi les allumer et les éteindre aléatoirement :

```c
void setup() {
  pinMode(2, OUTPUT); 
  pinMode(3, OUTPUT); 
  pinMode(4, OUTPUT); 
  randomSeed(analogRead(0)); 
}

void loop() {   
  const byte LED_PIN = random(2, 4); // On séléctionne un nombre qui réprésente un PIN de LED
  delay(1000); // On attend 1 seconde

  /*
   * On initialise une variable "state" qui aléatoirement
   * possédera la valeur HIGH (led allumé) ou LOW (led étinte).
   */
  byte state = 0; 

  /* 
  * On définit aléatoirement la valeur de la variable state 
  * graçe à cette condition 
  */ 
  if (random(0, 10) > 5)
    state = HIGH; 

  else
    state = LOW; 
 /* 
 * Si la valeur généré alétoirement est supérieur à 5, 
 * On fait une pause de x temps ou x est aussi aléatoirement 
 * compris entre 1 et 3 secondes
 */

  if (random(0, 10) > 5)
    delay(random(1000, 3000)); 

  /* La led séléctionné aléatoirement s'étindera ou s'allumera 
  aléatoirement, comme convenu */
  digitalWrite(LED_PIN, state); 
}
```



