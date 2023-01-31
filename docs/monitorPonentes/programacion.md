
# **Pulsometro.ino**

~~~
/*
 This example sketch gives you exactly what the SparkFun Pulse Oximiter and
 Heart Rate Monitor is designed to do: read heart rate and blood oxygen levels.
 
 Author: Elias Santistevan
 Date: 8/2019
 SparkFun Electronics

 Programa adaptado para la placa ESP32STEAMahers
 Manuel Hidalgo - LeoBot
 Mayo 2022
*/

#include <SparkFun_Bio_Sensor_Hub_Library.h>
#include <Wire.h>

// Reset pin, MFIO pin
int resPin = 25;
int mfioPin = 17;

// Takes address, reset pin, and MFIO pin.
SparkFun_Bio_Sensor_Hub bioHub(resPin, mfioPin); 

bioData body;  
// ^^^^^^^^^
//Informacion biometrica
//
// body.heartrate  - Heartrate
// body.confidence - Confidence in the heartrate value
// body.oxygen     - Blood oxygen level
// body.status     - Has a finger been sensed?

//Sensor sudoracion
const int GSR=35;
int sensorValue=0;
int gsr_average=0;

//Bluetooth BLE
#include "BluetoothSerial.h"
BluetoothSerial SerialBT;

void setup(){

  Serial.begin(115200);
  SerialBT.begin("Pulsometro"); //Bluetooth device name

  Wire.begin();
  int result = bioHub.begin();
  if (result == 0) // Zero errors!
    Serial.println("Sensor started!");
  else
    Serial.println("Could not communicate with the sensor!!!");
 
  Serial.println("Configuring Sensor...."); 
  int error = bioHub.configBpm(MODE_ONE); // Configuring just the BPM settings. 
  if(error == 0){ // Zero errors!
    Serial.println("Sensor configured.");
  }
  else {
    Serial.println("Error configuring sensor.");
    Serial.print("Error: "); 
    Serial.println(error); 
  }

  // Data lags a bit behind the sensor, if you're finger is on the sensor when
  // it's being configured this delay will give some time for the data to catch up. 
  Serial.println("Loading up the buffer with data....");
  delay(4000); 
  
}

void loop(){

    // Information from the readBpm function will be saved to our "body"
    // variable.  
    body = bioHub.readBpm();
    Serial.print("Heartrate: ");
    Serial.println(body.heartRate); 
    Serial.print("Confidence: ");
    Serial.println(body.confidence); 
    Serial.print("Oxygen: ");
    Serial.println(body.oxygen); 
    Serial.print("Status: ");
    Serial.println(body.status); 

    //media de 10 medidas sensor GSR
    long sum=0;
    for(int i=0;i<10;i++)           
      {
        sensorValue=analogRead(GSR);
        sum += sensorValue;
        delay(5);
      }
   gsr_average = sum/10;
   Serial.print("Valor medio GSR: ");
   Serial.println(gsr_average);
   Serial.println();

//Enviar datos Bluetooth
  SerialBT.print(body.heartRate);
  SerialBT.print(";");
  SerialBT.print(body.confidence);
  SerialBT.print(";");
  SerialBT.print(body.oxygen);
  SerialBT.print(";");
  SerialBT.print(body.status);
  SerialBT.print(";");
  SerialBT.print(gsr_average);
  SerialBT.println(";");
  
  
   
   delay(180); 
    
}
~~~