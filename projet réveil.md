#include <LedControl.h>
#include <Wire.h>
#include <RTClib.h>
#include <Arduino.h>
#include <Adafruit_BusIO_Register.h>


// Déclaration de la fonction displayTime
void displayTime(int hour, int minute, int second);

// Définition des broches pour le MAX7219
#define DATA_IN   4   // MOSI (broche de données)
#define CLK        5   // SCK (broche de l'horloge)
#define LOAD       6   // CS (broche de sélection)

#define MAX_DEVICES 4  // Nombre de modules MAX7219 connectés (ici 4 modules)

// Création de l'objet LedControl pour gérer le MAX7219
LedControl lc = LedControl(DATA_IN, CLK, LOAD, MAX_DEVICES);

// Création d'un objet pour le RTC (DS3231)
RTC_DS3231 rtc;

void setup() {
  // Initialisation des modules MAX7219
  for (int i = 0; i < MAX_DEVICES; i++) {
    lc.shutdown(i, false);    // Activer les modules
    lc.setIntensity(i, 8);     // Réglage de la luminosité (de 0 à 15)
    lc.clearDisplay(i);        // Effacer l'affichage au démarrage
  }

  // Initialisation de la communication avec le module RTC
  if (!rtc.begin()) {
    Serial.println("Impossible de trouver le module RTC");
    while (1); // Bloque le programme si le RTC n'est pas détecté
  }

  // Vérification que le module RTC fonctionne et que l'heure est correcte
  if (rtc.lostPower()) {
    // Si l'horloge a perdu l'alimentation, réinitialiser l'heure
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
  }
}

void loop() {
  // Obtenir l'heure actuelle du module RTC
  DateTime now = rtc.now();

  // Récupérer les heures, minutes et secondes
  int hour = now.hour();
  int minute = now.minute();
  int second = now.second();

  // Afficher l'heure sur les 4 modules MAX7219 (HH:MM:SS)
  displayTime(hour, minute, second);
  delay(1000); // Attendre 1 seconde avant de mettre à jour l'affichage
}

// Définition de la fonction displayTime
void displayTime(int hour, int minute, int second) {
  // Diviser l'heure, les minutes et les secondes en chiffres individuels
  int hourTens = hour / 10;    // Chiffre des dizaines de l'heure
  int hourOnes = hour % 10;    // Chiffre des unités de l'heure
  int minuteTens = minute / 10; // Chiffre des dizaines des minutes
  int minuteOnes = minute % 10; // Chiffre des unités des minutes
  int secondTens = second / 10; // Chiffre des dizaines des secondes
  int secondOnes = second % 10; // Chiffre des unités des secondes

  // Afficher les heures sur le premier module MAX7219
  lc.setDigit(0, 0, hourTens, false);  // Module 1 - Chiffre des dizaines de l'heure
  lc.setDigit(0, 1, hourOnes, false);  // Module 1 - Chiffre des unités de l'heure

  // Afficher les minutes sur le deuxième module MAX7219
  lc.setDigit(0, 2, minuteTens, false); // Module 2 - Chiffre des dizaines des minutes
  lc.setDigit(0, 3, minuteOnes, false); // Module 2 - Chiffre des unités des minutes

  // Afficher les secondes sur le troisième module MAX7219
  lc.setDigit(1, 0, secondTens, false);  // Module 3 - Chiffre des dizaines des secondes
  lc.setDigit(1, 1, secondOnes, false);  // Module 3 - Chiffre des unités des secondes
}
