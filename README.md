# Documentation Complète du Projet : Afficheur 7 Segments à Servomoteurs

> Teckbot Robotics Challenge – Test 3
> Réalisé par : **Hiba El Goumri** – 2e année GEE, ENSA Marrakech

---

## 🎯 Objectif du Projet

Ce projet a pour but de réaliser un **afficheur 7 segments** innovant, non lumineux, en utilisant **7 servomoteurs (SG90)**, chacun correspondant à un segment. Le dispositif devra afficher les chiffres de **0 à 9** puis de **9 à 0**, avec une temporisation de **1 seconde** entre chaque chiffre.

Le tout est piloté par un **ATmega328P (nu)**, sans carte Arduino ni breadboard. L'alimentation est assurée par une **batterie Li-ion**. Le code doit être **non bloquant** (sans `delay()`).

---

## 🔧 Architecture Matérielle

* **Microcontrôleur** : ATmega328P (programmé via ISP)
* **Affichage** : 7 servomoteurs SG90 agissant sur des segments mécaniques
* **Contrôle** : Signal PWM envoyé à chaque servo
* **Alimentation** : Batterie Li-ion + régulateur 5V (AMS1117)
* **Montage** : Veroboard ou PCB (sans Arduino ni breadboard)

---

## 📦 Liste des Composants

| Composant          | Référence                | Qté |
| ------------------ | ------------------------ | --- |
| Microcontrôleur    | ATmega328P               | 1   |
| Servomoteur        | SG90                     | 7   |
| Régulateur         | AMS1117 5V               | 1   |
| Quartz             | 16 MHz                   | 1   |
| Condensateurs      | 22pF + 100nF             | 4   |
| Résistance Pull-up | 10kΩ (reset)             | 1   |
| Batterie Li-ion    | 3.7V 18650               | 1   |
| Divers             | Veroboard, fils, headers | -   |

---

## ⚙️ Fonctionnement Global

Chaque servomoteur est lié à un segment (a à g). Selon le chiffre à afficher, une combinaison de segments est activée en déplaçant les servomoteurs en position "ON" ou "OFF" (ex : 90° ou 0°).

Le programme envoie un signal PWM à chaque servo en utilisant la bibliothèque `Servo.h`. Le passage d'un chiffre à l'autre est géré par `millis()` pour éviter les délais bloquants.

---

## 🧠 Fonctionnement détaillé du Servomoteur SG90

### Qu'est-ce qu'un servomoteur ?

Un servomoteur est un moteur équipé d'un réducteur et d'un capteur de position (potentiomètre), permettant un **contrôle précis de l'angle** de rotation via un signal PWM.

### Signal PWM

* 1 ms → 0°
* 1.5 ms → 90°
* 2 ms → 180°
* Répété toutes les 20 ms

### Application dans le projet

* Chaque servo déplace un segment entre 0° (OFF) et 90° (ON)
* Permet d'afficher n'importe quel chiffre entre 0 et 9

### Alimentation

* Ne pas alimenter les servos depuis l'ATmega328P !
* Utiliser une source externe (batterie + AMS1117)
* Masse (GND) partagée entre servo et MCU

---

## 💻 Code Arduino (fichier `code/servo_display.ino`)

* Utilise `Servo.h` pour chaque segment
* Tableau de configurations pour les chiffres 0 à 9
* Affichage croissant puis décroissant
* Utilisation de `millis()` pour gérer le délai de 1s
* Bien commenté et indenté

---

## 📁 Organisation du Dépôt GitHub

```
/
|-- README.md
|-- code/
|   |-- servo_display.ino
|   |-- README.md (explication des broches)
|-- docs/
|   |-- servo_moteur.md
|   |-- datasheets/
|       |-- SG90.pdf
|       |-- ATmega328P.pdf
|-- kicad/
|   |-- afficheur.pro
|   |-- afficheur.sch
|   |-- afficheur.pcbnew
|   |-- 3D.png
|-- media/
    |-- demo.mp4
    |-- wiring_diagram.png
```

---

## 📹 Vidéo de Démonstration

* Compteur 0 → 9 puis 9 → 0
* Chiffre toutes les secondes
* Segments bougent clairement
* Voir : `media/demo.mp4`

---

## 📝 Difficultés rencontrées

* Calibrage précis des servos
* Stabilité de l'alimentation
* Programmation du microcontrôleur nu
* Câblage clair et stable sur veroboard

---

## 🚀 Améliorations futures

* Ajouter un 2e chiffre (affichage à 2 chiffres = 14 servos)
* Créer une structure imprimée en 3D pour les segments
* Communication UART pour afficher des chiffres transmis

---

## 👩‍💻 Auteure

**Hiba El Goumri**
ENSA Marrakech, 2e année Génie Électronique Embarqué
[GitHub](https://github.com/) | [LinkedIn](https://linkedin.com/in/)

---

## 📜 Licence

Projet libre sous licence MIT. Usage pédagogique recommandé.
