// Demonstration of:
// The Microchip Technology Inc. 24AA02E48 is a 2 Kbit
// Electrically Erasable PROM. The device is organized as two
// blocks of 128 x 8-bit memory with a 2-wire serial interface.
// Low-voltage design permits operation down to 1.7V, with
// maximum standby and active currents of only 1 µA and 1 mA,
// respectively. The 24AA02E48 also has a page write capability
// for up to 8 bytes of data. The 24AA02E48 is available in the
// standard 8-pin SOIC and 5-lead SOT-23 packages

// Layout:
// 2 x 128 x 8 bit = 2 * 1024 bit
// 8-byte page write

// Control byte: 1010 0000
// MS 4-bits are control code. LS 4-bits are chip select (MS) and R/W.
// Chip Select bits are A2 A1 A0.
// example (A2-A0 set low or NA):
//  1010-0001 READ
//  1010-0000 WRITE

// Datasheet:
// http://ww1.microchip.com/downloads/en/DeviceDoc/20002124E.pdf
#include <Wire.h>

#define SUCCESS 1
#define ERROR 0

void setup(){
  uint8_t buffer[] = {0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF};
  
  Wire.begin();
  //TWIInit();
  
  //EEWriteByte(0,0x22);
  //delay(10);
  //EEWriteByte(0x0,buffer,8);
  //delay(10);

  Serial.begin(9600);
  pinMode(13, OUTPUT);
  
  //wait for Serial connection
  while(!Serial);
}

void TWIInit(void)
{
    //set SCL to 400kHz
    TWSR = 0x00;
    TWBR = 0x0C;
    //enable TWI
    TWCR = (1<<TWEN);
}
void TWIStart(void)
{
    TWCR = (1<<TWINT)|(1<<TWSTA)|(1<<TWEN);
    while ((TWCR & (1<<TWINT)) == 0);
}
//send stop signal
void TWIStop(void)
{
    TWCR = (1<<TWINT)|(1<<TWSTO)|(1<<TWEN);
}
void TWIWrite(uint8_t u8data)
{
    TWDR = u8data;
    TWCR = (1<<TWINT)|(1<<TWEN);
    while ((TWCR & (1<<TWINT)) == 0);
}
uint8_t TWIReadACK(void)
{
    TWCR = (1<<TWINT)|(1<<TWEN)|(1<<TWEA);
    while ((TWCR & (1<<TWINT)) == 0);
    return TWDR;
}
//read byte with NACK
uint8_t TWIReadNACK(void)
{
    TWCR = (1<<TWINT)|(1<<TWEN);
    while ((TWCR & (1<<TWINT)) == 0);
    return TWDR;
}
uint8_t TWIGetStatus(void)
{
    uint8_t status;
    //mask status
    status = TWSR & 0xF8;
    return status;
}
uint8_t EEReadByte(uint8_t u8addr, uint8_t *u8data)
{
    //uint8_t databyte;
    TWIStart();
    if (TWIGetStatus() != 0x08)
        return ERROR;
        
    //select device
    TWIWrite(160);
    if (TWIGetStatus() != 0x18)
        return ERROR;
        
    //send the address
    TWIWrite(u8addr);
    if (TWIGetStatus() != 0x28)
        return ERROR;

    TWIStart();
    if (TWIGetStatus() != 0x10)
        return ERROR;
    //select device (and read bit)
    TWIWrite(160|1);
    if (TWIGetStatus() != 0x40)
        return ERROR;
    
    *u8data = TWIReadNACK();
    if (TWIGetStatus() != 0x58)
        return ERROR;
        
    TWIStop();
    return SUCCESS;
}
uint8_t EEWriteByte(uint8_t u8addr, uint8_t u8data)
{
    //uint8_t databyte;
    TWIStart();
    if (TWIGetStatus() != 0x08)
        return ERROR;
        
    //select device
    TWIWrite(160);
    if (TWIGetStatus() != 0x18)
        return ERROR;
        
    //send the address
    TWIWrite(u8addr);
    if (TWIGetStatus() != 0x28)
        return ERROR;

    //send the data
    TWIWrite(u8data);
    if (TWIGetStatus() != 0x28)
        return ERROR;
 
    TWIStop();
    return SUCCESS;
}
uint8_t EEWriteByte(uint8_t u8addr, uint8_t u8data[], uint8_t count)
{
  if(count > 8)
    return ERROR;
    
    //uint8_t databyte;
    TWIStart();
    if (TWIGetStatus() != 0x08)
        return ERROR;
        
    //select device
    TWIWrite(160);
    if (TWIGetStatus() != 0x18)
        return ERROR;
        
    //send the address
    TWIWrite(u8addr);
    if (TWIGetStatus() != 0x28)
        return ERROR;

    //send the data
    for(int i = 0; i < count; i++){
      TWIWrite(u8data[i]);
      if (TWIGetStatus() != 0x28)
          return ERROR;
    }
 
    TWIStop();
    return SUCCESS;
}

void loop0(){
  Serial.println();  Serial.println();
  digitalWrite(13, HIGH);

  uint8_t u8ebyte;
  
  int count = 0;
  while (count <= 0xFF){
    if(EEReadByte(count++, &u8ebyte)){
      if(u8ebyte < 0x10) Serial.print("0");
      Serial.print(u8ebyte, HEX);
    }
    else{
      Serial.print("__");
    }
    
    if(count % 8) Serial.print(" ");
    if(count % 8 == 0 && count % 16) Serial.print(" - ");
    if(count % 16 == 0) Serial.println();
  }
    

  digitalWrite(13, LOW);

  delay(5000);
}  
void loop(){
  digitalWrite(13, HIGH);
  
  //Read and display the EEPROM contents
  Wire.beginTransmission(0x50);
  Wire.write(byte(0xF0));
  Wire.endTransmission();
  Wire.requestFrom(0x50, 0x10);    // request 6 bytes from slave device #2
  
  int count=0;
  while(Wire.available() && count <= 256)    // slave may send less than requested
  { 
    if(count % 16 == 0){
      if(count > 0) Serial.println();
      Serial.print("0000");
      if(count == 0) Serial.print("0");
      Serial.print(count,HEX);
      Serial.print(": ");
    }

    count++;
    byte c = Wire.read(); // receive a byte as character
    if(c<0x10) Serial.print("0");
    Serial.print(c, HEX);         // print the character

    if(count % 8 == 0 && count % 16 > 0)
        Serial.print(" - ");
    else
      Serial.print(" ");
      
  }
 
  
  Serial.println();
  Serial.println();
  
  Serial.print("TWSR: ");
  Serial.println(TWSR, HEX);
  Serial.print("TWBR: ");
  Serial.println(TWBR, HEX);
  Serial.print("TWCR: ");
  Serial.println(TWCR, HEX);
  
  Serial.println();

  
  
  //Read and display the EEPROM contents
//  Wire.beginTransmission(161);
//  Wire.write(byte(0x00));
//  char c = Wire.read();
//  Serial.print(c, HEX);
//  Serial.println();
//  Wire.endTransmission();


  digitalWrite(13, LOW);

  //wait 5 seconds before going for it again
  delay(5000);
}
  
  
