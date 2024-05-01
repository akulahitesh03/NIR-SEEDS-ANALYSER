#include "AS726X.h"
#include <LiquidCrystal.h> // Include the library for LCD

AS726X sensor;
LiquidCrystal lcd(12, 11, 5, 4, 3, 2); // Initialize LCD with pins
bool refFlag = false;
bool testFlag = false;
bool avgFlag = false;
bool refTriggered = false;
bool testTriggered = false;
bool avgTriggered = false;
float refValues[6];
float testValues[6];
float percent = 0;

int refUpdateCount = 0;
int testUpdateCount = 0;
int percentUpdateCount = 0;


void setup() {
  Wire.begin();
  Serial.begin(115200);
  sensor.begin();
  lcd.begin(16, 2); // Initialize the LCD with 16 columns and 2 rows
  lcd.print("System Ready"); // Initial message on LCD
  lcd.setCursor(0, 1); // Move cursor to the second row
  lcd.print("Perform NIR Test!!");
  // Set button pin modes
  pinMode(6, INPUT_PULLUP);  // Button connected to D8
  pinMode(7, INPUT_PULLUP);  // Button connected to D9
  pinMode(8, INPUT_PULLUP); // Button connected to D10

}

void loop() {
  sensor.takeMeasurements();


  // Check if D8 is high
  if (digitalRead(6) == LOW && !refTriggered) {
    unsigned long startTime = millis();
    while (digitalRead(6) == LOW && (millis() - startTime < 100)) {
      sensor.takeMeasurements();
    }
    if (millis() - startTime >= 100) {
      updateRefValues();
    }
    refTriggered = true;
  } else if (digitalRead(6) == LOW) {
    refTriggered = false;
  }

  // Check if D9 is high
  if (digitalRead(7) == LOW && !testTriggered) {
    unsigned long startTime = millis();
    while (digitalRead(7) == LOW && (millis() - startTime < 100)) {
      sensor.takeMeasurements();
    }
    if (millis() - startTime >= 100) {
      updateTestValues();
    }
    testTriggered = true;
  } else if (digitalRead(7) == LOW) {
    testTriggered = false;
  }

  // Check if D10 is high
  if (digitalRead(8) == LOW && !avgTriggered) {
    unsigned long startTime = millis();
    while (digitalRead(8) == LOW && (millis() - startTime < 100)) {
      sensor.takeMeasurements();
    }
    if (millis() - startTime >= 100) {
      calculatePercent();
    }
    avgTriggered = true;
  } else if (digitalRead(8) == LOW) {
    avgTriggered = false;
  }
}

// Function to update REF values
void updateRefValues() {
  storeValues(refValues);
  printValuesLCD("REF", refValues);
  refFlag = true;
  refUpdateCount++;
  printUpdateCountLCD("REF", refUpdateCount);
}

// Function to update TEST values
void updateTestValues() {
  storeValues(testValues);
  printValuesLCD("TEST", testValues);
  testFlag = true;
  testUpdateCount++;
  printUpdateCountLCD("TEST", testUpdateCount);
}

// Function to calculate PERCENT
void calculatePercent() {
  percent = calculateAverageVariation(refValues, testValues);
  if (isnan(percent) || isinf(percent)) {
    lcd.clear();
    lcd.print("Perform REF/TEST");
  } else {
    lcd.setCursor(0, 1); // Move cursor to the second row
    lcd.print("Avg Variation: ");
    lcd.print(percent);
    avgFlag = true;
    percentUpdateCount++;
    printUpdateCountLCD("PERCENT", percentUpdateCount);
  }
}

// Function to store the sensor values
void storeValues(float *values) {
  values[0] = sensor.getCalibratedR();
  values[1] = sensor.getCalibratedS();
  values[2] = sensor.getCalibratedT();
  values[3] = sensor.getCalibratedU();
  values[4] = sensor.getCalibratedV();
  values[5] = sensor.getCalibratedW();
}

// Function to print the sensor values on LCD
void printValuesLCD(String label, float *values) {
  lcd.clear();
  lcd.print(label + " Reading:");
  for (int i = 0; i < 6; i++) {
    lcd.print(values[i], 2);
    lcd.print(" ");
  }
}

// Function to print the update count on LCD
void printUpdateCountLCD(String label, int count) {
  lcd.clear();
  lcd.print("Update ");
  lcd.print(count);
  lcd.print(" for ");
  lcd.print(label);

   if (label == "PERCENT") { // Check if the label is "PERCENT"
    lcd.setCursor(0, 1); // Move cursor to the second row
    lcd.print("Percent: "); // Print "Percent: "
    lcd.print(percent); // Print the percent value
  }
}

// Function to calculate the average variation percentage
float calculateAverageVariation(float *refValues, float *testValues) {
  float totalVariation = 0;
  for (int i = 0; i < 6; i++) {
    totalVariation += ((testValues[i] - refValues[i]) / refValues[i]) * 100;
  }
  return totalVariation / 6;
}

