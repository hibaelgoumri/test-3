# 📘 Documentation Complète du Projet : Afficheur 7 Segments à Servomoteurs

> **Test 3 – Teckbot Robotics Challenge**

---

## 🎯 Objectif du Projet

Créer un afficheur 7 segments mécanique utilisant **7 servomoteurs **, pilotés par un **ATmega328P** nu (sans carte Arduino), pour afficher les chiffres de **0 à 9**, puis **de 9 à 0**. Le tout doit être alimenté par une **batterie Li-ion**, et le code doit être **non bloquant** (sans `delay()`), avec un affichage toutes les secondes.

 ## 🗂️ Sommaire
 
 [🎯 Cahier des charges](#-cahier-des-charges)    
 [🔧 Architecture Générale](#-architecture-générale)  
 [⚙️ Fonctionnement Global](#-fonctionnement-global)  
 [🧠 Fonctionnement détaillé du Servomoteur SG90](#-fonctionnement-détaillé-du-servomoteur-sg90)  
 [🛠️ Utilisation de Deux Types de Servomoteurs](#️-utilisation-de-deux-types-de-servomoteurs-sg90--parallax)  
 [📚 Bibliothèque nécessaire](#-bibliothèque-nécessaire)  
 [💻 Code Arduino (version multi-servos)](#-code-arduino-version-multi-servos)  
 [🧪 Test et Démonstration](#-test-et-démonstration)

 ## 🎯 Cahier des charges  
### Objectifs fonctionnels :  
- Créer un afficheur 7 segments mécanique utilisant **des servomoteurs **
- Contrôler les servomoteurs avec un **ATmega328P** nu (sans carte Arduino)
- Générer un affichage fluide des chiffres de **0 à 9 puis 9 à 0**, **sans blocage**
- Utiliser un signal PWM avec la bibliothèque `Servo.h`
- Concevoir un circuit imprimé (PCB) optimisé et compact
- Alimenter le système avec une batterie Li-ion **via un régulateur AMS1117**
- 
### Contraintes techniques :  
- Le système doit être **autonome** (sans PC ni carte Arduino)
- Les segments doivent être **entièrement visibles mécaniquement**
- Aucun `delay()` ne doit être utilisé dans le code (gestion avec `millis()`)
- Le PCB doit intégrer le régulateur, le quartz, les résistances et les condensateurs
- L’ensemble doit être testé **en simulation et en maquette** avant production

---

## 🔧 Architecture Générale

### 🧩 Matériel Utilisé

| Composant          | Référence            | Qté |
| ------------------ | -------------------- | --- |
| Microcontrôleur    | ATmega328P           | 1   |
| Servomoteur        | SG90/Parallax        | 7   |
| Régulateur         | LM7805  5V           | 1   |
| Quartz             | 16 MHz               | 1   |
| Condensateurs      | 22pF                 | 2   |
| Résistance Pull-up | 10kΩ (reset)         | 1   |
| Batterie Li-ion    | 3.7V                 | 1   |
| Veroboard          | -                    | 1   |
| Fils & connecteurs | -                    | -   |

---

## ⚙️ Fonctionnement Global

Le principe de fonctionnement repose sur l’utilisation de 7 servomoteurs, chacun étant affecté à un segment de l’afficheur (de a à g). Ces segments ne sont pas lumineux, mais déplacés mécaniquement à l’aide des bras des servomoteurs, ce qui crée une représentation physique du chiffre souhaité. Pour chaque chiffre (de 0 à 9), une configuration particulière des segments doit être activée ou désactivée. Cela signifie que certains servomoteurs se positionnent dans une position visible (par exemple 90° pour un segment « allumé ») tandis que d’autres se rétractent (0° pour un segment « éteint »). L’ensemble est piloté par un microcontrôleur ATmega328P, qui génère un signal PWM adapté à chaque servo pour lui indiquer sa position. Le code est conçu pour éviter toute fonction bloquante comme delay() ; les changements de chiffres sont gérés toutes les secondes à l’aide de la fonction millis(), garantissant un comportement fluide et réactif du système.  

---

## 🧠 Fonctionnement détaillé du Servomoteur SG90

### Qu'est-ce qu'un servomoteur ?

Un servomoteur est un moteur équipé d’un réducteur et d’un potentiomètre qui permet un **contrôle précis de l’angle de rotation**, généralement entre **0° et 180°**.  
![image](https://github.com/user-attachments/assets/0b8a4a6c-d0d1-4a34-965a-7431a619ccf9)


### Caractéristiques techniques du SG90

| Caractéristique        | Valeur            |
| ---------------------- | ----------------- |
| Dimensions             | 22 x 11.5 x 27 mm |
| Poids                  | 9 g               |
| Tension d’alimentation | 4.8 V à 6 V       |
| Vitesse                | 0.12 s / 60°      |
| Couple                 | 1.2 kg/cm         |
| Angle de rotation      | 0° à 180°         |

### Connexion

| Fil    | Fonction     | Connexion MCU             |
| ------ | ------------ | ------------------------- |
| Marron | Masse (GND)  | GND                       |
| Rouge  | Alimentation | +5 V régulée (AMS1117)    |
| Orange | Signal PWM   | Broche numérique (ex: D3) |

### Signal PWM

Le servomoteur SG90 est commandé par un signal PWM (Pulse Width Modulation), qui est une suite d'impulsions répétées périodiquement. La largeur de l'impulsion (temps pendant lequel le signal est à l'état haut) détermine l'angle de positionnement du servomoteur. En général :  
* Une impulsion de **1 ms** positionne l'axe à **0°** (gauche)  
* Une impulsion de **1.5 ms** positionne l'axe à **90°** (milieu)  
* Une impulsion de **2 ms** positionne l'axe à **180°** (droite)  
Ce signal est **répété toutes les 20 ms**, soit une fréquence de **50 Hz**. Le microcontrôleur doit maintenir cette fréquence et adapter la durée de l'impulsion pour indiquer la position voulue. Si la fréquence est trop basse ou si le signal n’est pas stable, le servomoteur risque de vibrer ou de perdre sa position.  
### 🔄 Code pour initialisation du servomoteur SG90

```cpp
// Test utilisation servomoteur SG90
#include <Servo.h>
Servo monservo; // Crée l’objet pour contrôler le servomoteur

void setup() {
  monservo.attach(9);     // Utilise la broche 9 pour le contrôle
  monservo.write(0);      // Positionne le servomoteur à 0° (repos)
}

void loop() {
  // Boucle vide pour ce test
}
```
> Ce code représente la toute première étape de l’utilisation d’un servomoteur. Il permet de l’attacher à une broche numérique (ici D9) et de le positionner à un angle précis (ici 0°).  
> Ce code utilise `millis()` pour gérer la temporisation, ce qui permet d'éviter toute fonction bloquante comme `delay()`.  

---
## 🛠️ Utilisation de Deux Types de Servomoteurs (SG90 & Parallax)
Pendant le développement, nous n’avons pas pu trouver 7 servomoteurs SG90 identiques. Pour contourner ce problème, nous avons utilisé une combinaison de servomoteurs SG90 et Parallax Continuous Rotation.
### ➕ **Adaptation du fonctionnement** :
Les SG90 ont été utilisés normalement pour représenter les segments ON/OFF via des angles précis.  
Les Parallax Continuous Rotation (rotation continue) ont été calibrés pour tourner brièvement dans un sens ou l’autre (ON ou OFF), puis s’arrêter.  
### 🧠 **Fonctionnement détaillé du Servomoteur Parallax Continuous Rotation**
#### Qu’est-ce qu’un servo à rotation continue ?
Ce type de servo ne peut pas se positionner à un angle fixe. Il tourne dans un sens ou l’autre à une certaine vitesse selon le signal PWM reçu.  
![image](https://github.com/user-attachments/assets/a3d58c60-1039-43cd-9092-1484068ae4a4)

#### Caractéristiques techniques

| Caractéristique        | Valeur             |
|------------------------|--------------------|
| Type                   | Rotation continue  |
| Tension                | 4.8 – 6 V          |
| Signal neutre          | 1500 µs (arrêt)    |
| Vitesse max            | ~60 tr/min         |
| Contrôle               | Par largeur d’impulsion |
| Angle de positionnement | Non applicable    |

####  Signal de contrôle

| PWM (µs)     | Effet                 |
|--------------|-----------------------|
| < 1500 µs    | Rotation sens horaire |
| > 1500 µs    | Rotation sens antihoraire |
| = 1500 µs    | Arrêt (neutre)        |


#### 💡 Utilisation dans notre projet

| État du segment | Signal PWM         | Détail                      |
|-----------------|--------------------|-----------------------------|
| ON              | 1300 µs pendant 200 ms | Tourne brièvement (sens 1) |
| OFF             | 1700 µs pendant 200 ms | Tourne brièvement (sens 2) |
| Arrêt           | 1500 µs              | Stop rotation              |

⚠️ Comme ces moteurs ne retournent pas à une position fixe, chaque moteur a été calibré manuellement pour assurer l’alignement visuel des segments.

---

## 📚 Bibliothèque nécessaire

### Servo.h

La bibliothèque `Servo.h` est une bibliothèque native d'Arduino qui permet de contrôler facilement les servomoteurs à l’aide d’un signal PWM. Elle prend en charge :

* L’attachement d’un servomoteur à une broche numérique via `servo.attach(pin);`
* L’envoi d’un angle de rotation avec `servo.write(angle);`
* La gestion du signal PWM en arrière-plan sans que l’utilisateur ait à le générer manuellement

🔧 Cette bibliothèque est **incluse par défaut** avec l’IDE Arduino. Aucun téléchargement supplémentaire n’est nécessaire.  


---

## 💻 Code Arduino (version multi-servos)  
Dans ce projet, le code Arduino est structuré pour contrôler 7 servomoteurs, chacun correspondant à un segment (de a à g) de l’afficheur 7 segments mécanique.
Voici les points clés du code :
- **Utilisation de la bibliothèque Servo.h**
Cette bibliothèque facilite le contrôle des servomoteurs en générant automatiquement le signal PWM nécessaire sur les broches numériques du microcontrôleur.  
- **Création de 7 objets Servo distincts**
Chaque segment (a, b, c, d, e, f, g) est associé à un objet Servo différent. Cela permet de commander individuellement chaque servomoteur en lui envoyant un angle spécifique.  
- **Définition d’un tableau chiffres[10][7]** 
Ce tableau contient la configuration des segments pour chaque chiffre de 0 à 9.  
Chaque ligne du tableau correspond à un chiffre.  
Chaque colonne correspond à un segment (a à g).  
La valeur 1 signifie que le segment doit être activé (servo en position « ON », par exemple 90°).  
La valeur 0 signifie que le segment doit être désactivé (servo en position « OFF », par exemple 0°).  
Cela permet d’activer ou désactiver facilement les segments nécessaires pour afficher un chiffre donné en parcourant simplement ce tableau.  
- **Gestion de la temporisation avec millis()**  
Pour que le programme reste réactif et évite les blocages, la fonction millis() est utilisée pour déclencher le changement de chiffre toutes les secondes.  
millis() retourne le nombre de millisecondes écoulées depuis le démarrage du programme.  
En stockant la dernière valeur de millis() lors d’un changement, on peut comparer à la valeur actuelle pour savoir quand une seconde s’est écoulée sans utiliser delay().  
Cela permet au microcontrôleur de continuer à gérer les servomoteurs et autres tâches sans interruption ni blocage.  

### Tableau logique des chiffres (Segments activés)

| Chiffre | A | B | C | D | E | F | G |
|--------:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|   0     | ✔️ | ✔️ | ✔️ | ❌ | ❌ | ❌ | ❌ |
|   1     | ❌ | ✔️ | ✔️ | ❌ | ❌ | ❌ | ✔️ |
|   2     | ✔️ | ✔️ | ❌ | ❌ | ❌ | ✔️ | ✔️ |
|   3     | ✔️ | ✔️ | ✔️ | ❌ | ✔️ | ✔️ | ✔️ |
|   4     | ❌ | ✔️ | ✔️ | ❌ | ❌ | ❌ | ✔️ |
|   5     | ✔️ | ❌ | ✔️ | ❌ | ✔️ | ❌ | ✔️ |
|   6     | ✔️ | ❌ | ✔️ | ❌ | ❌ | ❌ | ✔️ |
|   7     | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ | ❌ |
|   8     | ✔️ | ✔️ | ✔️ | ❌ | ❌ | ❌ | ✔️ |
|   9     | ✔️ | ✔️ | ✔️ | ❌ | ✔️ | ❌ | ✔️ |

Chaque ligne correspond à un chiffre et chaque colonne (a–g) représente un segment :

* **✔️** = segment activé (servo en position ON)
* **❌** = segment désactivé (servo en position OFF)
### Le code 
```cpp
#include <Servo.h>  // Inclusion de la bibliothèque standard Servo

// Définition des angles pour contrôler les segments
#define ANGLE_ON  0     // Angle correspondant à un segment "allumé"
#define ANGLE_OFF 90    // Angle correspondant à un segment "éteint"

// Déclaration du nombre total de segments (7 segments A à G)
const int NUM_SEGMENTS = 7;

// Tableau des broches reliées aux servomoteurs pour les segments A à G
const int servoPins[NUM_SEGMENTS] = {2, 3, 4, 5, 6, 7, 8};

// Tableau d’objets Servo, un pour chaque segment
Servo servos[NUM_SEGMENTS];

// Définition de l’état ON (0°) ou OFF (90°) pour chaque chiffre de 0 à 9
// Chaque ligne correspond à un chiffre, chaque colonne à un segment : A B C D E F G
const bool digits[10][7] = {
  {0, 0, 0, 1, 1, 1, 1}, // 0
  {1, 0, 0, 0, 0, 0, 1}, // 1
  {0, 0, 1, 1, 1, 0, 0}, // 2
  {0, 0, 0, 1, 0, 0, 0}, // 3
  {1, 0, 0, 0, 0, 1, 0}, // 4
  {0, 1, 0, 1, 0, 1, 0}, // 5
  {0, 1, 0, 1, 1, 1, 0}, // 6
  {0, 0, 0, 0, 0, 0, 1}, // 7
  {0, 0, 0, 1, 1, 1, 0}, // 8
  {0, 0, 0, 1, 0, 1, 0}  // 9
};

// Variables pour gérer la temporisation sans bloquer le code
unsigned long previousMillis = 0;         // Mémorise le temps du dernier changement
const unsigned long interval = 1000;      // Intervalle entre les changements (1 seconde)

// Variables pour suivre le chiffre actuel et la direction du comptage
int digit = 0;        // Chiffre actuellement affiché
int direction = 1;    // 1 = incrémenter, -1 = décrémenter

// Fonction d'initialisation : attache chaque servo à sa broche
void setup() {
  for (int i = 0; i < NUM_SEGMENTS; i++) {
    servos[i].attach(servoPins[i]);  // Initialise chaque servo
  }

  displayDigit(digit);  // Affiche le premier chiffre (0)
}

// Boucle principale qui s’exécute en continu
void loop() {
  unsigned long currentMillis = millis();  // Temps écoulé depuis le démarrage

  // Si une seconde s'est écoulée, on change de chiffre
  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;  // Met à jour le temps du dernier changement

    digit += direction;  // Incrémente ou décrémente le chiffre

    // Si on dépasse 9, on repart dans l'autre sens (compte à rebours)
    if (digit > 9) {
      digit = 9;
      direction = -1;
    }

    // Si on descend en dessous de 0, on repart vers le haut
    else if (digit < 0) {
      digit = 0;
      direction = 1;
    }

    // Affiche le chiffre actuel sur les servos
    displayDigit(digit);
  }
}

// Fonction qui positionne les servos en fonction du chiffre à afficher
void displayDigit(int d) {
  for (int i = 0; i < NUM_SEGMENTS; i++) {
    // Si la valeur est 0 → servo à ANGLE_ON (segment allumé)
    // Si la valeur est 1 → servo à ANGLE_OFF (segment éteint)
    servos[i].write(digits[d][i] ? ANGLE_ON : ANGLE_OFF);
  }
}

```
---

## 🧪 Test et Démonstration

https://github.com/user-attachments/assets/130669d9-22cf-476d-a163-3bf61610d7c9
## realisation de PCB 
### Schématique
 
footprints
 
Organisation du PCB :
 
Routage par freerouting :
 
Edge.cuts
Dimension de la coupure sont : 66.5/48.5 mm
 
Vu 3D :

