# Smart---RFID--Based---Automated-System
Smart RFID-Based Automated System
#include <SoftwareSerial.h>
#include <Wire.h>
#include <LCD_I2C.h>
#include "HX711.h"

// GSM
SoftwareSerial mySerial(6, 7);

// LCD
LCD_I2C lcd(0x27);

// Buttons
#define key1 2
#define key2 3
#define key3 5
#define key4 4

// Load Cell
const int dout = 9;
const int sck = 8;
HX711 scale;

// RFID Tags
char tag1[] = "4D0098AF87FD";
char tag2[] = "4D008726E10D";
char tag3[] = "4D008766BE12";

// Variables
float kg = 0, weight = 0;
unsigned int sugaramt = 0, soapamt = 0, biscuitamt = 0, total_amt = 0;
unsigned int tagg1 = 0, tagg2 = 0, tagg3 = 0;
int aa = 0, bb = 0, cc = 0;

char input[12];
int count = 0;

void setup()
{
  Serial.begin(9600);
  mySerial.begin(115200);

  Wire.begin();
  lcd.begin();
  lcd.backlight();

  lcd.print("SMART TROLLEY");
  delay(2000);
  lcd.clear();

  pinMode(key1, INPUT_PULLUP);
  pinMode(key2, INPUT_PULLUP);
  pinMode(key3, INPUT_PULLUP);
  pinMode(key4, INPUT_PULLUP);

  // Load Cell Setup
  scale.begin(dout, sck);
  scale.set_scale(2280.f);
  scale.tare();
}

void loop()
{
  readWeight();
  checkButtons();
  readRFID();
}

// ---------------- WEIGHT ----------------
void readWeight()
{
  weight = scale.get_units(10);

  if (weight < 0) weight = -weight;
  kg = (weight / 25) * 132;

  if (kg < 0.1) kg = 0;
}

// ---------------- BUTTONS ----------------
void checkButtons()
{
  if (digitalRead(key1) == LOW)
  {
    calculateAmount();
  }

  if (digitalRead(key2) == LOW)
  {
    reduceItem();
  }

  if (digitalRead(key3) == LOW)
  {
    displayData();
  }

  if (digitalRead(key4) == LOW)
  {
    sendData();
  }
}

// ---------------- CALCULATION ----------------
void calculateAmount()
{
  lcd.clear();

  if (aa == 1)
  {
    sugaramt = kg / 25;
    total_amt += sugaramt;

    lcd.print("SUGAR: ");
    lcd.print(sugaramt);
  }

  if (bb == 1)
  {
    soapamt = kg / 2;
    total_amt += soapamt;

    lcd.print("SOAP: ");
    lcd.print(soapamt);
  }

  if (cc == 1)
  {
    biscuitamt = kg / 10;
    total_amt += biscuitamt;

    lcd.print("BISCUIT: ");
    lcd.print(biscuitamt);
  }

  delay(2000);
  lcd.clear();
}

// ---------------- REDUCE ----------------
void reduceItem()
{
  if (aa == 1 && tagg1 > 0)
  {
    tagg1--;
    total_amt -= sugaramt;
    lcd.print("SUGAR REDUCED");
  }

  if (bb == 1 && tagg2 > 0)
  {
    tagg2--;
    total_amt -= soapamt;
    lcd.print("SOAP REDUCED");
  }

  if (cc == 1 && tagg3 > 0)
  {
    tagg3--;
    total_amt -= biscuitamt;
    lcd.print("BISCUIT REDUCED");
  }

  delay(2000);
  lcd.clear();
}

// ---------------- DISPLAY ----------------
void displayData()
{
  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("SU:");
  lcd.print(sugaramt);

  lcd.setCursor(8, 0);
  lcd.print("SO:");
  lcd.print(soapamt);

  lcd.setCursor(0, 1);
  lcd.print("BT:");
  lcd.print(biscuitamt);

  lcd.setCursor(8, 1);
  lcd.print("T:");
  lcd.print(total_amt);

  delay(3000);
  lcd.clear();
}

// ---------------- RFID ----------------
void readRFID()
{
  if (Serial.available())
  {
    count = 0;

    while (Serial.available() && count < 12)
    {
      input[count++] = Serial.read();
      delay(5);
    }

    if (strncmp(input, tag1, 12) == 0)
    {
      aa = 1; bb = cc = 0;
      tagg1++;
      lcd.print("SUGAR");
    }

    else if (strncmp(input, tag2, 12) == 0)
    {
      bb = 1; aa = cc = 0;
      tagg2++;
      lcd.print("SOAP");
    }

    else if (strncmp(input, tag3, 12) == 0)
    {
      cc = 1; aa = bb = 0;
      tagg3++;
      lcd.print("BISCUIT");
    }

    delay(2000);
    lcd.clear();
  }
}

// ---------------- SEND ----------------
void sendData()
{
  mySerial.println("AT+CMGF=1");
  delay(1000);

  mySerial.println("AT+CMGS=\"+91XXXXXXXXXX\"");
  delay(1000);

  mySerial.print("Total: ");
  mySerial.println(total_amt);

  mySerial.write(26);
  delay(2000);

  lcd.print("DATA SENT");
  delay(2000);
  lcd.clear();
}
