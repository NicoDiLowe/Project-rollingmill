#include <Arduino.h>
#include <TM1637TinyDisplay6.h>

//LCD display
#define pinDisplayDC 0
#define pinDisplayCS 1
#define pinDisplayCLK 2
#define pinDisplayDIN 3
#define pinDisplayRST 4
#define pinDisplayBL 5
//switch
#define pinLeftSwitch 11
#define pinCalibrationSwitch 12 //center switch
#define pinRightSwitch 13
//segment display
#define DIO_L 18
#define CLK_L 19
#define DIO_R 20
#define CLK_R 21
//ADC 
#define pinLeftPotentiometer 26
#define pinRightPotentiometer 27
#define pinReferenceVoltageLevel 28

double leftPotentiometerCalibrationValue = 3388;
double rightPotentiometerCalibrationValue = 3409;
// double leftPotentiometerCalibrationValue = 1212;
// double rightPotentiometerCalibrationValue = 1222;
double conversionValueToMM = -0.01923;

TM1637TinyDisplay6 displayRight(CLK_R, DIO_R);
TM1637TinyDisplay6 displayLeft(CLK_L, DIO_L);

void setup() {
  //set the resolution to 12 bits (0-4096)
  analogReadResolution(12);
  //set pin
  pinMode(pinCalibrationSwitch, INPUT_PULLUP);
  //set display
  displayRight.begin();
  displayLeft.begin();
  displayRight.setBrightness(BRIGHT_HIGH);
  displayLeft.setBrightness(BRIGHT_HIGH);
}


std::tuple<double, double> ReadAverageValue(){
  int nPointsCount = 100000; 
  long int sumRight = 0;
  long int sumLeft = 0; 
  for(int i = 0; i < nPointsCount; i++){
    int right = analogRead(pinRightPotentiometer);
    int left = analogRead(pinLeftPotentiometer);
    sumRight += right;
    sumLeft += left;
  }
  
  double avgRight = sumRight / nPointsCount;
  double avgLeft = sumLeft / nPointsCount;

  return {avgRight, avgLeft};
}

void loop() {
  // read the analog value
  double avgRight = 0;
  double avgLeft = 0;
  std::tie(avgRight, avgLeft) = ReadAverageValue();
  

  double rightPotentiometerValue = avgRight - rightPotentiometerCalibrationValue;
  double leftPotentiometerValue = avgLeft - leftPotentiometerCalibrationValue;

  DisplayDistance(rightPotentiometerValue, leftPotentiometerValue);

  if(digitalRead(pinCalibrationSwitch) == 0) Calibration();
  
  delay(200);
}

void Calibration() {
  displayRight.clear();
  displayLeft.clear(); 
  delay(1000);
  if(digitalRead(pinCalibrationSwitch) == 0) { 
    double avgRight = 0;
    double avgLeft = 0;
    std::tie(avgRight, avgLeft) = ReadAverageValue();
    rightPotentiometerCalibrationValue = avgRight - 7;
    leftPotentiometerCalibrationValue = avgLeft - 7;
  }
  displayRight.showNumber(0.00000);
  displayLeft.showNumber(0.00000); 
  delay(2000);
}

void DisplayDistance(double rightPotentiometerValue, double leftPotentiometerValue) {
  double rightValue = ConvertToMM(rightPotentiometerValue);
  double leftValue = ConvertToMM(leftPotentiometerValue);
  //if(rightValue < 0) rightValue = 0;
  //if(leftValue < 0) leftValue = 0;

  displayRight.showNumber(rightValue);
  displayLeft.showNumber(leftValue);  
}

double ConvertToMM(double potentiometerValue) {
  // double value = potentiometerValue * conversionValueToMM - 0.15;
   double value = potentiometerValue * conversionValueToMM;
  //double value = potentiometerValue;
  return value;
}