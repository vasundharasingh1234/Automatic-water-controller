#include <EEPROM.h>
#include <LiquidCrystal.h>

LiquidCrystal lcd(2, 3, 4, 5, 6, 7);  // RS, EN, D4, D5, D6, D7

long duration, inches;
int set_val, percentage;
bool state, pump;

void setup() {
  lcd.begin(16, 2);
  lcd.print("WATER LEVEL");
  lcd.setCursor(0, 1);
  lcd.print("PUMP:OFF MANUAL");

  pinMode(8, OUTPUT);       // Trigger pin of ultrasonic sensor
  pinMode(9, INPUT);        // Echo pin of ultrasonic sensor
  pinMode(10, INPUT_PULLUP); // Set button
  pinMode(11, INPUT_PULLUP); // Mode button (Auto/Manual)
  pinMode(12, OUTPUT);      // Pump relay control

  set_val = EEPROM.read(0);  // Read saved tank height
  if (set_val > 150) set_val = 150;
}

void loop() {
  // Ultrasonic distance measurement
  digitalWrite(8, LOW);
  delayMicroseconds(2);
  digitalWrite(8, HIGH);
  delayMicroseconds(10);
  digitalWrite(8, LOW);

  duration = pulseIn(9, HIGH);
  inches = microsecondsToInches(duration);

  // Convert distance to percentage
  percentage = map(inches, set_val, 0, 0, 100);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("WATER LEVEL:");
  lcd.setCursor(13, 0);
  lcd.print(percentage);
  lcd.print("%");

  // Pump automatic control
  if (percentage < 30 && digitalRead(11)) pump = 1;
  if (percentage > 90) pump = 0;
  digitalWrite(12, pump);

  // Display pump status
  lcd.setCursor(5, 1);
  if (pump == 1) lcd.print("ON ");
  else lcd.print("OFF");

  // Display mode
  lcd.setCursor(9, 1);
  if (digitalRead(11)) lcd.print("MANUAL");
  else lcd.print("AUTO");

  // Save tank height when set button pressed
  if (digitalRead(10) && !state && digitalRead(11)) {
    state = 1;
    set_val = inches;
    EEPROM.write(0, set_val);
  }

  // Manual mode pump toggle
  if (digitalRead(10) && state && digitalRead(11)) {
    state = 1;
    pump = !pump;
  }

  if (digitalRead(10)) state = 0;

  delay(500);
}

// Function to convert ultrasonic time to inches
long microsecondsToInches(long microseconds) {
  return microseconds / 74 / 2;
}

