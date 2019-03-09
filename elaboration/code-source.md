# Code source

[Suite à plusieurs problématiques rencontrées ](https://projet-theatre.gitbook.io/pupitre/~/edit/drafts/-LYJXe9jh4gCNtIX17MU/documentation-techniques/probleme-s-rencontre-s)lors du projet, nous avons abandonné le développement du projet à l'aide de l'environnement Arduino au profit du Raspberry PI. **Ainsi, le code-source Arduino est malheureusement manquant.** 

## Avec le Raspberry PI

Le code source du projet pupitre implémenté en langage Python est disponible à cette adresse : 

{% embed url="https://github.com/projet-theatre/pupitre" %}

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

