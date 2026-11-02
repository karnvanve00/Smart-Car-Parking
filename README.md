# Smart-Car-Parking
ESP32-based Smart Car Parking System using IR sensors, a servo motor, LCD display, and LEDs for automatic vehicle detection and gate control.



This lone to START Code
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <ESP32Servo.h>

// Hardware instances
LiquidCrystal_I2C lcd(0x27, 16, 2);
Servo gateServo;

// Pin Definitions
const int irEntry = 13;
const int irExit  = 12;
const int servoPin = 14;
const int redLed   = 25;
const int greenLed = 26;

void setup() {
  Serial.begin(115200);
  
  pinMode(irEntry, INPUT);
  pinMode(irExit, INPUT);
  pinMode(redLed, OUTPUT);
  pinMode(greenLed, OUTPUT);
  
  gateServo.attach(servoPin);
  
  // Initial State: Gate Closed, Red Light On
  gateServo.write(0); 
  digitalWrite(redLed, HIGH);
  digitalWrite(greenLed, LOW);

  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Traffic Control");
  lcd.setCursor(0, 1);
  lcd.print("System Ready...");
  delay(2000);
  lcd.clear();
}

void loop() {
  int entryVal = digitalRead(irEntry);

  // Default State: Red Light On, Gate Closed
  if (entryVal == HIGH) {
    digitalWrite(redLed, HIGH);
    digitalWrite(greenLed, LOW);
    gateServo.write(0);
    
    lcd.setCursor(0, 0);
    lcd.print("  PARKING SLOT  ");
    lcd.setCursor(0, 1);
    lcd.print(" STATUS: EMPTY  ");
  } 
  // Car Arrives
  else {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("CAR DETECTED!   ");
    lcd.setCursor(0, 1);
    lcd.print("PLEASE WAIT...  ");
    delay(1000);

    // Open Gate and Switch to Green
    gateServo.write(90);
    digitalWrite(redLed, LOW);
    digitalWrite(greenLed, HIGH);
    
    lcd.setCursor(0, 1);
    lcd.print("PROCEED NOW >>  ");

    // Wait for the car to trigger the exit sensor
    while(digitalRead(irExit) == HIGH) {
      delay(50); 
    }

    // Car is currently passing the gate
    lcd.setCursor(0, 1);
    lcd.print("CAR PASSING...  ");
    
    // Wait for the car to clear the exit sensor fully
    while(digitalRead(irExit) == LOW) {
      delay(50);
    }

    // Car has cleared: Switch back to Red and Close Gate
    delay(1000); // Buffer delay
    digitalWrite(greenLed, LOW);
    digitalWrite(redLed, HIGH);
    gateServo.write(0);
    
    lcd.clear();
    lcd.print("GATE CLOSING...");
    delay(2000);
  }
  
  delay(100); 
}
