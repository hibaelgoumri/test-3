# 📘 Documentation Complète du Projet : Afficheur 7 Segments à Servomoteurs

> **Test 3 – Teckbot Robotics Challenge**

---

## 🎯 Objectif du Projet

Créer un afficheur 7 segments mécanique utilisant **7 servomoteurs SG90**, pilotés par un **ATmega328P** nu (sans carte Arduino), pour afficher les chiffres de **0 à 9**, puis **de 9 à 0**. Le tout doit être alimenté par une **batterie Li-ion**, et le code doit être **non bloquant** (sans `delay()`), avec un affichage toutes les secondes.

---

## 🔧 Architecture Générale

### 🧩 Matériel Utilisé

| Composant          | Référence    | Qté |
| ------------------ | ------------ | --- |
| Microcontrôleur    | ATmega328P   | 1   |
| Servomoteur        | SG90         | 7   |
| Régulateur         | LM7805  5V   | 1   |
| Quartz             | 16 MHz       | 1   |
| Condensateurs      | 22pF         | 2   |
| Résistance Pull-up | 10kΩ (reset) | 1   |
| Batterie Li-ion    | 3.7V         | 1   |
| Veroboard          | -            | 1   |
| Fils & connecteurs | -            | -   |

---

## ⚙️ Fonctionnement Global

Le principe de fonctionnement repose sur l’utilisation de 7 servomoteurs SG90, chacun étant affecté à un segment de l’afficheur (de a à g). Ces segments ne sont pas lumineux, mais déplacés mécaniquement à l’aide des bras des servomoteurs, ce qui crée une représentation physique du chiffre souhaité. Pour chaque chiffre (de 0 à 9), une configuration particulière des segments doit être activée ou désactivée. Cela signifie que certains servomoteurs se positionnent dans une position visible (par exemple 90° pour un segment « allumé ») tandis que d’autres se rétractent (0° pour un segment « éteint »). L’ensemble est piloté par un microcontrôleur ATmega328P, qui génère un signal PWM adapté à chaque servo pour lui indiquer sa position. Le code est conçu pour éviter toute fonction bloquante comme delay() ; les changements de chiffres sont gérés toutes les secondes à l’aide de la fonction millis(), garantissant un comportement fluide et réactif du système.  

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

## 📚 Bibliothèque nécessaire

### Servo.h

La bibliothèque `Servo.h` est une bibliothèque native d'Arduino qui permet de contrôler facilement les servomoteurs à l’aide d’un signal PWM. Elle prend en charge :

* L’attachement d’un servomoteur à une broche numérique via `servo.attach(pin);`
* L’envoi d’un angle de rotation avec `servo.write(angle);`
* La gestion du signal PWM en arrière-plan sans que l’utilisateur ait à le générer manuellement

🔧 Cette bibliothèque est **incluse par défaut** avec l’IDE Arduino. Aucun téléchargement supplémentaire n’est nécessaire.

> ⚠️ Avec un ATmega328P nu, il faut s’assurer que le bootloader et les bons fuse bits sont configurés pour supporter le signal PWM sur les broches utilisées.

---

## 💻 Code Arduino (version multi-servos)

* Utilise la bibliothèque `Servo.h`
* 7 objets Servo (a à g)
* Tableau `chiffres[10][7]` : chaque ligne est une configuration ON/OFF
* Temporisation gérée avec `millis()`

### Tableau logique des chiffres (Segments activés)

| Chiffre | a | b | c | d | e | f | g |
| ------- | - | - | - | - | - | - | - |
| 0       | 1 | 1 | 1 | 1 | 1 | 1 | 0 |
| 1       | 0 | 1 | 1 | 0 | 0 | 0 | 0 |
| 2       | 1 | 1 | 0 | 1 | 1 | 0 | 1 |
| 3       | 1 | 1 | 1 | 1 | 0 | 0 | 1 |
| 4       | 0 | 1 | 1 | 0 | 0 | 1 | 1 |
| 5       | 1 | 0 | 1 | 1 | 0 | 1 | 1 |
| 6       | 1 | 0 | 1 | 1 | 1 | 1 | 1 |
| 7       | 1 | 1 | 1 | 0 | 0 | 0 | 0 |
| 8       | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| 9       | 1 | 1 | 1 | 1 | 0 | 1 | 1 |

Chaque ligne correspond à un chiffre et chaque colonne (a–g) représente un segment :

* **1** = segment activé (servo en position ON)
* **0** = segment désactivé (servo en position OFF)


---

## 🧪 Test et Démonstration

