# Arduino-project
  This is a project for auto/manual window blinds control using arduino UNO board.
  The code and the hardware part was done by me

  
#include <Servo.h>
#include <LiquidCrystal_I2C.h>
#include <Wire.h>
#include <DS3231.h>
#include "Keyboard.h"

int led = 9;
int senspin_out = A0;
int senspin_in = A1;
int servo_pin = 8;
int val1 = 575;
int RECV_PIN = 11;
int angle = 0;
int mem = 0;
int  serialData = 0;
int data = 0;
boolean blinds_open = true;
boolean manual_mode = false;
Servo servo;
LiquidCrystal_I2C lcd(0x3F, 16, 2);
DS3231 rtc(A2, A3);

void setup() {

  lcd.init();
  lcd.backlight();
  pinMode(led, OUTPUT);
  Serial.begin(9600);
  servo.attach(servo_pin);
  servo.write(angle);
  rtc.begin();

}
void manual()
{
    if (Serial.available() > 0) 

 {

 serialData = Serial.read(); 
 Serial.println(serialData); 



 data = data +1; 
 }

}
void loop() {
 if (Serial.available() > 0) {
    char input = Serial.read();
    if (input == 'M') {
      manual_mode = !manual_mode; 
      if (manual_mode) {
        lcd.setCursor(0, 1);
        lcd.print("Mod: Manual ");
      } else {
        lcd.setCursor(0, 1);
        lcd.print("Mod: Automat");
      }
    } else if (manual_mode) {
      if (input == 'O') {
        openBlinds();
      } else if (input == 'C') {
        closeBlinds();
      }
      if(input == '1'){
        digitalWrite(led, HIGH);
      }
      else if(input == '2'){
         digitalWrite(led, LOW);
      }
    }
  }

  if (!manual_mode) {
    int light_out = getlight(senspin_out);
    Serial.println(light_out);
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Lumina: ");
    if (light_out < 100) {
      lcd.print("Zi");
    } else {
      lcd.print("Noapte");
    }
    lcd.setCursor(0, 1);
    lcd.print("Led: ");
    ledctrl(led);

  controlblinds();
   Serial.print("Time:  ");
 Serial.print(rtc.getTimeStr());
 
 Serial.print("Date: ");
 Serial.print(rtc.getDateStr());
 
  delay(5000);

}
}
void controlblinds() {
  int light_out = getlight(senspin_out);
  if (light_out < val1 && !blinds_open && rtc.getTimeStr()  <= "18:00:00" || rtc.getTimeStr() == "08:00:00") {
    openBlinds();
  } else if (light_out >= val1 && blinds_open || rtc.getTimeStr() == "18:00:00") {
    closeBlinds();
  }
  servo.write(mem);
}

void openBlinds() {
  for(angle = 0; angle <= 90 ; angle ++)
  {
  servo.write(angle);
  delay(15);
  }
   mem = angle;
  blinds_open = true;
}

void closeBlinds() {
   for(angle = 90; angle >= 0 ; angle --){

   
  servo.write(angle);
  delay(15);
  }
   mem = angle;
  blinds_open = false;
}

void ledctrl(int led) {
  int light_in = getlight(senspin_in);
  if (light_in > 100) {
    digitalWrite(led, HIGH);
    lcd.print("on");
  } else {
    digitalWrite(led, LOW);
    lcd.print("off");
  }
}

int getlight(int senspin) {
  int light = analogRead(senspin);
  return light;
}
