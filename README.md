# smart-bin
//PROJECT NAME : SMART DUSTBIN DESIGN
//FUNCTION:-
//OPEN OR CLOSE THE DUSTBIN AUTAMATIC
//MEASURE THE LEVEL OF GARBAGE
//WARM UP THE PEOPLE TO CLEAN THE DUSTBIN
//HINT:-
//GREEN LED MEANS LOW LEVEL - NO NEED TO EMPTY THE DUSTBIN
//ORANGE LED MEANS MIDDLE LEVEL - BE CARE TO EMPTY THE DUSTBIN
//RED LED MEANS HIGH LEVEL - NEED TO EMPTY IMMEDIATELY
//AT THE SAME TIME SOUND PRODUCE TO WARM PEOPLE TO DO CLEAN UP
//THE BUZZER CAN TURN OFF MANUALLY
//GREEN  : 0 ~ 25
//ORANGE : 26 ~ 49
//RED    : > 49

#include <LiquidCrystal.h>
LiquidCrystal lcd (2, 3, 4, 5, 6, 7);

#include <Servo.h>
Servo myServo;

#define pinServo 13
#define pinMotion 10
int angle = 0;
int sensorState;

#define pinTrig 12
#define pinEcho 11
float duration;
//distance from top of lid to the garbage at the bottom
int distance;
int nearLib = 25;  //indicate garbage near the lid
int farLib = 50;   //indicate garbage far away from lid

#define greenLED A5
#define orangeLED A4
#define redLED A3

#define adjustTone A1
#define buzzer 9
int buzzerTone;

void setup()
{
  Serial.begin(9600); 
  lcd.begin(16, 2);
  
  pinMode(greenLED, OUTPUT);
  pinMode(orangeLED, OUTPUT);
  pinMode(redLED, OUTPUT);
  
  pinMode(pinTrig, OUTPUT);
  pinMode(pinEcho, INPUT);
  
  pinMode(buzzer, OUTPUT);
  pinMode(adjustTone, INPUT);
  
  pinMode(pinMotion, INPUT);
  myServo.attach(pinServo);
  myServo.write(angle);
}

void loop()
{
  //PART:-CHECK THE LEVEL OF RUBBISH INSIDE DUSTBIN
  int TONE = analogRead(adjustTone);
  buzzerTone = map(TONE, 0, 1023, 0, 255);
  
  digitalWrite(pinTrig, LOW);
  delayMicroseconds(2);
  digitalWrite(pinTrig, HIGH);
  delayMicroseconds(10);
  digitalWrite(pinTrig, LOW);

  duration = pulseIn(pinEcho, HIGH);
  distance = (duration * 0.0343) / 2;
  
  Serial.print("Distance rubbish to the top of the dustbin = ");
  Serial.print(distance);
  Serial.println();
  
  //situation when garbage level HIGH
  if(distance <= nearLib){
    analogWrite(buzzer, buzzerTone); //design to turn off manually
    digitalWrite(greenLED, LOW);
    digitalWrite(orangeLED, LOW);
    digitalWrite(redLED, LOW);
    delay(500);
    digitalWrite(redLED, HIGH);
    delay(500);
    lcd.setCursor(0,0);
    lcd.print("      FULL!");
    lcd.setCursor(0,1);
    lcd.print("   CLEAN NOW");
    delay(250);
    lcd.clear();
  }
  //situation when garbage level MIDDLE
  else if((distance > nearLib)&&(distance < farLib)){
    digitalWrite(buzzer, LOW);
    digitalWrite(greenLED, LOW);
    digitalWrite(orangeLED, HIGH);
    digitalWrite(redLED, LOW);
    lcd.setCursor(0,0);
    lcd.print("       BE");
    lcd.setCursor(0,1);
    lcd.print("   ATTENTION");
    delay(1500);
    lcd.clear();
  }
  //situation when rubbish is LOW
  else if(distance >= farLib){
    digitalWrite(buzzer, LOW);
    digitalWrite(greenLED, HIGH);
    digitalWrite(orangeLED, LOW);
    digitalWrite(redLED, LOW);
    lcd.setCursor(0,0);
    lcd.print("  HAVE A NICE");
    lcd.setCursor(0,1);
    lcd.print("      DAY");
    delay(2000);
    lcd.clear();
  }
  
  //PART:-AUTOMATIC OPEN OR CLOSE THE DUSTBIN
  sensorState = digitalRead(pinMotion);
  if(sensorState == HIGH){
	Serial.println("Motion detected!");
    myServo.write(180);
    delay(1000);
  }
  else if(sensorState == LOW){
    myServo.write(0);
  }
}
