#include <ESP8266WiFi.h>
#include <WiFiClientSecure.h>
#include <UniversalTelegramBot.h>

#include <Wire.h>
#include "MAX30100_PulseOximeter.h"
#include "DHT.h"

#define WIFI_SSID "ak"
#define WIFI_PASSWORD "ak123456"
// Telegram BOT Token (Get from Botfather)
#define BOT_TOKEN "5325153289:AAHElHjAahmc-hj-fHtJMCFnD2QEBR55RzA"

#define DHTPIN 14
#define DHTTYPE DHT11

String CHAT_ID = "946414007";
const unsigned long BOT_MTBS = 100; // mean time between scan messages

float BPM, SpO2;
int buzz = 10;
int mq4 = 12;
int flame = 13;
int button = 15; 
DHT dht(DHTPIN, DHTTYPE);
PulseOximeter pox;
 float t;
unsigned long bot_lasttime; // last time messages' scan has been done
X509List cert(TELEGRAM_CERTIFICATE_ROOT);
WiFiClientSecure secured_client;
UniversalTelegramBot bot(BOT_TOKEN, secured_client);

  

int g;
void setup()
{
  Serial.begin(115200);
  Serial.println();
  pinMode(buzz, OUTPUT);
  digitalWrite(buzz,LOW);
  dht.begin();
  pinMode(16, OUTPUT);
  // attempt to connect to Wifi network:
  configTime(0, 0, "pool.ntp.org");      // get UTC time via NTP
  secured_client.setTrustAnchors(&cert); // Add root certificate for api.telegram.org
  Serial.print("Connecting to Wifi SSID ");
  Serial.print(WIFI_SSID);
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  while (WiFi.status() != WL_CONNECTED)
  {
    Serial.print(".");
    delay(500);
  }
  Serial.print("\nWiFi connected. IP address: ");
  Serial.println(WiFi.localIP());

  // Check NTP/Time, usually it is instantaneous and you can delete the code below.
  Serial.print("Retrieving time: ");
  time_t now = time(nullptr);
  while (now < 24 * 3600)
  {
    Serial.print(".");
    delay(100);
    now = time(nullptr);
  }
  Serial.println(now);
  if (!pox.begin()) {
 
    for (;;);
  } 
  else 
  {
    delay(1);
  }


}

void loop()
{
  
  pox.update();
  BPM = pox.getHeartRate();
    SpO2 = pox.getSpO2();
    String A,B;
     Serial.print("BPM  :" );
     Serial.println( BPM);

     Serial.print("SpO2 : ");
     Serial.print( SpO2);
     Serial.println(" %");
  int numNewMessages = bot.getUpdates(bot.last_message_received + 1);

  float h = dht.readHumidity();
  float t = dht.readTemperature();

  Serial.print("handleNewMessages ");
  Serial.println(numNewMessages);
  if(g != numNewMessages ){
    g = numNewMessages;
    String answer;
    for (int i = 0; i < numNewMessages; i++)
    {
      telegramMessage &msg = bot.messages[i];
      Serial.println("Received " + msg.text);
      if (msg.text == "/start"){
        answer = "HI *" + msg.from_name + ". NOW THE BOT IS ACTIVED";
      }
      if (msg.text == "/status"){
        float t = dht.readTemperature();
        answer.concat(t);
      }
      if (msg.text == "/stop"){
        digitalWrite(buzz,HIGH);
        delay(500);
        digitalWrite(buzz,LOW);
        delay(200);
        digitalWrite(buzz,HIGH);
        delay(500);
        digitalWrite(buzz,LOW);
        delay(200);
        digitalWrite(buzz,HIGH);
        delay(500);
        digitalWrite(buzz,LOW);
        delay(200);
        digitalWrite(buzz,HIGH);
        delay(500);
        digitalWrite(buzz,LOW);
      }
      bot.sendMessage(msg.chat_id, answer, "Markdown");
    }}

    

    if(BPM>170 &&BPM!=0){
      
      digitalWrite(buzz,HIGH);
      delay(500);
      digitalWrite(buzz,LOW);
      delay(500);
      bot.sendMessage(CHAT_ID, "HIGH BPM");
}
   if(BPM<70 &&BPM!=0){
      ;
      digitalWrite(buzz,HIGH);
      delay(500);
      digitalWrite(buzz,LOW);
      delay(500);
      bot.sendMessage(CHAT_ID, "LOW BPM");
}
   if(SpO2<85 &&SpO2!=0){
     
      digitalWrite(buzz,HIGH);
      delay(500);
      digitalWrite(buzz,LOW);
      delay(500); 
      bot.sendMessage(CHAT_ID, "LOW SpO2");
}
   if(t>45){
     
      digitalWrite(buzz,HIGH);
      delay(500);
      digitalWrite(buzz,LOW);
      delay(500);
      bot.sendMessage(CHAT_ID, "HIGH TEMPERATURE");
}
   if(t<15){
    
      digitalWrite(buzz,HIGH);
      delay(500);
      digitalWrite(buzz,LOW);
      delay(500);
       bot.sendMessage(CHAT_ID, "LOW TEMPERATURE");
   }
   if(digitalRead( flame)== LOW){
     
      digitalWrite(buzz,HIGH);
      delay(100);
      digitalWrite(buzz,LOW);
      delay(100);
      bot.sendMessage(CHAT_ID, "flame detected");
}
    if(digitalRead (mq4) == LOW){
     
     digitalWrite(buzz,HIGH);
      delay(100);
      digitalWrite(buzz,LOW);
      delay(100);
      bot.sendMessage(CHAT_ID, "Toxic gas detected");
    }
   if(digitalRead (button) == HIGH){
    digitalWrite(buzz,HIGH);
        delay(500);
        digitalWrite(buzz,LOW);
     bot.sendMessage(CHAT_ID, "Distress");
   }
}
