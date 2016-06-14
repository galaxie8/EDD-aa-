# EDD-aa-
//arduino code, determines if PIR sensor is on, and if no acceleration (using MPU6050), then finVal= 1
//finVal is sent to connecting MIT App Inventor Code that can manually override signals by user using a button
//if button pressed, out is 0 and module turned off for 5 minutes, otherwise finVal=out
//out1 blinks lights in vehicle
//out 2 turns on sirens in vehicle

int pirInput = A3;
int accValue;
int accValue1;
int pirValue;
int finVal;
String t;
int out2;
int out = A0;
int out1=A1;
int OldAcX;
int OldAcY;
//_bluetooth.enable();
int state = LOW;

#include "I2Cdev.h"
#include "MPU6050.h"

#if I2CDEV_IMPLEMENTATION == I2CDEV_ARDUINO_WIRE
#include "Wire.h"
#endif

MPU6050 accelgyro;
const int MPU = 0x68; // I2C address of the MPU-6050
int16_t AcX,AcY,AcZ,Tmp,GyX,GyY,GyZ;

#define OUTPUT_READABLE_ACCELGYRO


void setup() {

  Wire.begin();
 Wire.beginTransmission(MPU);
  Wire.write(0x6B);  // PWR_MGMT_1 register
  Wire.write(0);     // set to zero (wakes up the MPU-6050)
  //Wire.endTransmission(true);
  accelgyro.initialize();
  Serial.begin(9600);
  delay(750);
  pinMode(out, OUTPUT);
  pinMode(pirInput, INPUT);
  pinMode(out1,OUTPUT);
  digitalWrite(finVal, 0);
  digitalWrite(pirInput, LOW);
  digitalWrite(out2, 0);
  digitalWrite(out, LOW); 
  digitalWrite(out1, LOW); 
  digitalWrite(accValue, LOW);
  digitalWrite(accValue1, 0);
}

void loop() {

  Wire.beginTransmission(MPU);
  Wire.write(0x3B);  // starting with register 0x3B (ACCEL_XOUT_H)
  Wire.endTransmission(false);
  Wire.requestFrom(MPU,14,true);  // request a total of 14 registers
  AcX=Wire.read()<<8|Wire.read();  // 0x3B (ACCEL_XOUT_H) & 0x3C (ACCEL_XOUT_L)     
  AcY=Wire.read()<<8|Wire.read();  // 0x3D (ACCEL_YOUT_H) & 0x3E (ACCEL_YOUT_L)
 
Serial.print("AcX = "); Serial.print(AcX-OldAcX);
  Serial.print(" | AcY = "); Serial.print(AcY-OldAcY);

//if motion is changing in any direction 
  if (abs(AcX - OldAcX)>200|| abs(AcY - OldAcY)>200)
  {
    digitalWrite(accValue, LOW);
    digitalWrite(accValue1, 0);
    Serial.println(" Motion Detected");
  }
  else {
    digitalWrite(accValue, HIGH);
    digitalWrite(accValue1, 1);
    Serial.println(" No Motion Detected");
  }
      Serial.println(accValue1);

  //pirValue = digitalRead(pirInput);
  digitalWrite(pirValue, (digitalRead(pirInput)));
  if(pirValue==HIGH){
  Serial.println("pir HIGH");
  }
  else{
    Serial.println("pir LOW");
  }
  if ( accValue1>0 && pirValue==HIGH)
  {
   digitalWrite(finVal, 1);
    Serial.println("Threat Detected");
  }
  else {
    digitalWrite(finVal, 0);
    Serial.println("No Threat Detected");
  }
  
while(Serial.available()){
t+=(char)Serial.read();
}
  //turns alarm on/off
  if(t=="on"){
  digitalWrite(out, HIGH);
  digitalWrite(out2, 1);
  delay(1000);
  }
  
  else {
    digitalWrite(out, LOW);
    digitalWrite(out2, 0);
    delay(1000);
  }
  
 
  if (out2 >0) //blinks led on external breadboard
  {
    digitalWrite(out1, HIGH);
    delay(1000);
    digitalWrite(out1, LOW);
    delay(1000);
  }
  //new accValues
  OldAcX = AcX;
  OldAcY = AcY;
}
