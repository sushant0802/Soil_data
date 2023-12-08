# Soil_data
#include <DHT.h>  // Including library for dht
#include "MQ135.h"
#include <ESP8266WiFi.h> 

String apiKey = "KT9DGYL03RZUB26D";     

const char *ssid =  "PM";     
const char *pass =  "Dy/dxof24*";
const char* server = "api.thingspeak.com";
const int sensorPin= 0;


#define DHTPIN 4            


DHT dht(DHTPIN, DHT11);

WiFiClient client;

void setup() 
{
       Serial.begin(115200);
       delay(10);
       dht.begin();

       Serial.println("Connecting to ");
       Serial.println(ssid);


       WiFi.begin(ssid, pass);

      while (WiFi.status() != WL_CONNECTED) 
     {
            delay(500);
            Serial.print(".");
     }
      Serial.println("");
      Serial.println("WiFi connected");

}

void loop() 
{
       int value = analogRead(A0);
       value = map(value,400,1023,100,0);
      float h = dht.readHumidity();
      float t = dht.readTemperature();


                         if (client.connect(server,80))   
                      {  

                             String postStr = apiKey;
                             postStr +="&field1=";
                             postStr += String(value);
                             postStr +="&field2=";
                             postStr += String(t);
                             postStr +="&field3=";
                             postStr += String(h);
                             postStr += "\r\n\r\n";

                             client.print("POST /update HTTP/1.1\n");
                             client.print("Host: api.thingspeak.com\n");
                             client.print("Connection: close\n");
                             client.print("X-THINGSPEAKAPIKEY: "+apiKey+"\n");
                             client.print("Content-Type: application/x-www-form-urlencoded\n");
                             client.print("Content-Length: ");
                             client.print(postStr.length());
                             client.print("\n\n");
                             client.print(postStr);

                             Serial.print("Temperature: ");
                             Serial.print(t);
                             Serial.print(" degrees Celcius, Humidity: ");
                             Serial.print(h);
                             Serial.print("%, Soil Moisture ");
                             Serial.print(value);         
                             Serial.println("%. Send to Thingspeak.");
                        }
          client.stop();

          Serial.println("Waiting...");

  // Gravimetric Soil Moisture Detection This method extracts water from a soil sample through evaporation, flushing, and a chemical reaction. The gravimetric soil moisture is calculated based on measuring the difference between the wet and dry sample weight. GWC (%) = [(mass of moist soil (g) − mass of dry soil (g)) / mass of dry soil (g)] × 100
  delay(1000);
}
