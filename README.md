# arduino-uno
 gate with  arduino uno 
/*
 * --------------------------------------------------------------------------------------------------------------------
 * Example sketch/program showing how to read new NUID from a PICC to serial.
 * --------------------------------------------------------------------------------------------------------------------
 * This is a MFRC522 library example; for further details and other examples see: https://github.com/miguelbalboa/rfid
 * 
 * Example sketch/program showing how to the read data from a PICC (that is: a RFID Tag or Card) using a MFRC522 based RFID
 * Reader on the Arduino SPI interface.
 * 
 * When the Arduino and the MFRC522 module are connected (see the pin layout below), load this sketch into Arduino IDE
 * then verify/compile and upload it. To see the output: use Tools, Serial Monitor of the IDE (hit Ctrl+Shft+M). When
 * you present a PICC (that is: a RFID Tag or Card) at reading distance of the MFRC522 Reader/PCD, the serial output
 * will show the type, and the NUID if a new card has been detected. Note: you may see "Timeout in communication" messages
 * when removing the PICC from reading distance too early.
 * 
 * @license Released into the public domain.
 * 
 * Typical pin layout used:
 * -----------------------------------------------------------------------------------------
 *             MFRC522      Arduino       Arduino   Arduino    Arduino          Arduino
 *             Reader/PCD   Uno/101       Mega      Nano v3    Leonardo/Micro   Pro Micro
 * Signal      Pin          Pin           Pin       Pin        Pin              Pin
 * -----------------------------------------------------------------------------------------
 * RST/Reset   RST          9             5         D9         RESET/ICSP-5     RST
 * SPI SS      SDA(SS)      10            53        D10        10               10
 * SPI MOSI    MOSI         11 / ICSP-4   51        D11        ICSP-4           16
 * SPI MISO    MISO         12 / ICSP-1   50        D12        ICSP-1           14
 * SPI SCK     SCK          13 / ICSP-3   52        D13        ICSP-3           15
 */

#include <SPI.h>
#include <MFRC522.h>
#include <Servo.h>

// servo motors are defined below
Servo EnteranceServo;
Servo ExitServo;
// position of motor


// Rfid is defined below
#define SS_PIN 10
#define RST_PIN 5
 
//MFRC522 rfid(SS_PIN,RST_PIN); // Instance of the class

MFRC522 rfid(10,8); // Instance of the class
MFRC522 rfid2(9,8); // Instance of the class

//MFRC522::MIFARE_Key key; 

// Init array that will store new NUID 
byte enterancePICC[4];
byte exitPICC[4];
int ParkQuantive = 5;
int Current = 0;


void setup() {

  pinMode(A0,INPUT);//Enterance gate sensor
  pinMode(A1,INPUT);//Exit gate sensor
  pinMode(13, OUTPUT); //set ledPin as OUTPUT 
  EnteranceServo.attach(12); 
  ExitServo.attach(11);
  
  pinMode(LED_BUILTIN, OUTPUT);
  Serial.begin(9600);
  SPI.begin(); // Init SPI bus


}
 
void loop() {

    enteranceGate();
    
    exitGate();      
}

void enteranceGate(){

    rfid.PCD_Init(); // Init MFRC522 
      
    // Look for new cards
    if ( ! rfid.PICC_IsNewCardPresent())
    return;


    // Verify if the NUID has been readed
    if (! rfid.PICC_ReadCardSerial())
    return;

    
    if (!(rfid.uid.size == 0)){
      
      Serial.println(F("From enterance gate."));
      printHex(rfid.uid.uidByte, rfid.uid.size);
      
      
      if(Current == ParkQuantive){
    
        Serial.println(F("Sorry! There is no space."));
        //delay(500);    
    
      }else if((Current<ParkQuantive) && (Serial.parseInt() == 1)){

            gateControl(EnteranceServo,"Enterance");   
       
      }
      
      // Halt PICC
      rfid.PICC_HaltA();

      // Stop encryption on PCD
      rfid.PCD_StopCrypto1();     
      
    }
  
}

void exitGate(){  
    
    rfid2.PCD_Init(); // Init MFRC522 
      
    // Look for new cards
    if (! rfid2.PICC_IsNewCardPresent())
    return;

    // Verify if the NUID has been readed
    if (! rfid2.PICC_ReadCardSerial())
    return;
  

    if (!(rfid2.uid.size == 0)){
      
      Serial.println(F("From exit gate."));
      printHex(rfid2.uid.uidByte, rfid2.uid.size);      

      if(Current == 0){

      Serial.println(F("Do not play with system there is no car to exit"));
      //delay(500);
      
      }else if(Serial.parseInt() == 1){

        gateControl(ExitServo,"Exit");           

      }
    }  
  // Halt PICC
  rfid2.PICC_HaltA();  

  // Stop encryption on PCD
  rfid2.PCD_StopCrypto1();
  
}

void gateControl(Servo servoMotor, String gate){

       int distance;
       servoMotor.write(10);
       Serial.print("The ");Serial.print(gate);Serial.println(" opened.");
       delay(2000);
       if (gate == "Enterance"){
          Current = Current + 1;
          distance = analogRead(A0); 

          while(analogRead(A0)<=20){
            digitalWrite(13, HIGH);
            Serial.print("Distance: ");Serial.print(distance);Serial.println(". Please, enter inside. do not wait under the gate");
            delay(1000);            
          }
                
          digitalWrite(13, LOW);
          servoMotor.write(90);    
          Serial.print("The ");Serial.print(gate);Serial.println(" closed.");
        
                 
       }else if(gate == "Exit"){
          Current = Current - 1;
          distance = analogRead(A1);

          while(analogRead(A1)<=20){
            digitalWrite(13, HIGH);
            Serial.print("Distance: ");Serial.print(distance);Serial.println(". Please, enter inside. do not wait under the gate");
            delay(1000);   

          }

        
          digitalWrite(13, LOW);
          servoMotor.write(90);    
          Serial.print("The ");Serial.print(gate);Serial.println(" closed.");
       
                  
       }      
              
    

      Serial.print(Current);
      Serial.println(F(" Cars stay in the otopark."));  
}

void printHex(byte *buffer, byte bufferSize) {
  
  for (byte i = 0; i < bufferSize; i++) {
    Serial.print(buffer[i] < 0x10 ? " 0" : " ");
    Serial.print(buffer[i], HEX);
    
  }
  Serial.println(" ");
}

/**
 * Helper routine to dump a byte array as dec values to Serial.
 */
void printDec(byte *buffer, byte bufferSize) {
  
  for (byte i = 0; i < bufferSize; i++) {
    Serial.print(buffer[i] < 0x10 ? " 0" : " ");
    Serial.print(buffer[i], DEC); 
  }
  Serial.println(" ");
}
