# Internet of Things Manual 2
This manual helps you to connect your Arduino with a Telegram bot.

<p align="center">
<img src="https://i0.wp.com/randomnerdtutorials.com/wp-content/uploads/2020/07/ESP32-ESP8266-NodeMCU-Telegram-Control-Outputs-Overview.png?resize=828%2C561&quality=100&strip=all&ssl=1"  width="500" >
</p>

### 1. Creating a Telegram bot

1. Install telegram on your device, can be your phone or pc. (tip: You will need to copy some long numbers so it's a good idea to connect it to your pc)
2. Search for user: "BotFather" and start a conversation. This bot will help you set up your own bot in Telegram.

<p align="center">
<img src="https://i0.wp.com/randomnerdtutorials.com/wp-content/uploads/2020/06/Telegram-Botfather.png?w=362&quality=100&strip=all&ssl=1"  width="300" >
</p>

3. Click the "/start" message and then type "/newbot". This should open the instructions to create your bot. Give it a name and a username.
4. If you managed this you will get a link to acces your bot and a bot token (the number after "HTTP API:"). Save the bot token, you will need it later.

### 2. Get your Telegram User ID

We will now make it that only you can interact with your bot. For this we need to whitelist your Telegram User ID.

1. Search for user: "myidbot" and start a conversation.
<p align="center">
<img src="https://i0.wp.com/randomnerdtutorials.com/wp-content/uploads/2020/06/Telegram-ID-Bot.png?w=348&quality=100&strip=all&ssl=1"  width="300" >
</p>
2. Type "/getid" and the bot will reply with your user ID. Save this for later too.

### 3. Sketch

1. Create a new sketch in Arduino IDE and copy and paste this code:
```javascript
/*
  Rui Santos
  Complete project details at https://RandomNerdTutorials.com/telegram-control-esp32-esp8266-nodemcu-outputs/
  
  Project created using Brian Lough's Universal Telegram Bot Library: https://github.com/witnessmenow/Universal-Arduino-Telegram-Bot
  Example based on the Universal Arduino Telegram Bot Library: https://github.com/witnessmenow/Universal-Arduino-Telegram-Bot/blob/master/examples/ESP8266/FlashLED/FlashLED.ino
*/

#ifdef ESP32
  #include <WiFi.h>
#else
  #include <ESP8266WiFi.h>
#endif
#include <WiFiClientSecure.h>
#include <UniversalTelegramBot.h>   // Universal Telegram Bot Library written by Brian Lough: https://github.com/witnessmenow/Universal-Arduino-Telegram-Bot
#include <ArduinoJson.h>

// Replace with your network credentials
const char* ssid = "REPLACE_WITH_YOUR_SSID";
const char* password = "REPLACE_WITH_YOUR_PASSWORD";

// Initialize Telegram BOT
#define BOTtoken "XXXXXXXXXX:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"  // your Bot Token (Get from Botfather)

// Use @myidbot to find out the chat ID of an individual or a group
// Also note that you need to click "start" on a bot before it can
// message you
#define CHAT_ID "XXXXXXXXXX"

#ifdef ESP8266
  X509List cert(TELEGRAM_CERTIFICATE_ROOT);
#endif

WiFiClientSecure client;
UniversalTelegramBot bot(BOTtoken, client);

// Checks for new messages every 1 second.
int botRequestDelay = 1000;
unsigned long lastTimeBotRan;

const int ledPin = 2;
bool ledState = LOW;

// Handle what happens when you receive new messages
void handleNewMessages(int numNewMessages) {
  Serial.println("handleNewMessages");
  Serial.println(String(numNewMessages));

  for (int i=0; i<numNewMessages; i++) {
    // Chat id of the requester
    String chat_id = String(bot.messages[i].chat_id);
    if (chat_id != CHAT_ID){
      bot.sendMessage(chat_id, "Unauthorized user", "");
      continue;
    }
    
    // Print the received message
    String text = bot.messages[i].text;
    Serial.println(text);

    String from_name = bot.messages[i].from_name;

    if (text == "/start") {
      String welcome = "Welcome, " + from_name + ".\n";
      welcome += "Use the following commands to control your outputs.\n\n";
      welcome += "/led_on to turn GPIO ON \n";
      welcome += "/led_off to turn GPIO OFF \n";
      welcome += "/state to request current GPIO state \n";
      bot.sendMessage(chat_id, welcome, "");
    }

    if (text == "/led_on") {
      bot.sendMessage(chat_id, "LED state set to ON", "");
      ledState = HIGH;
      digitalWrite(ledPin, ledState);
    }
    
    if (text == "/led_off") {
      bot.sendMessage(chat_id, "LED state set to OFF", "");
      ledState = LOW;
      digitalWrite(ledPin, ledState);
    }
    
    if (text == "/state") {
      if (digitalRead(ledPin)){
        bot.sendMessage(chat_id, "LED is ON", "");
      }
      else{
        bot.sendMessage(chat_id, "LED is OFF", "");
      }
    }
  }
}

void setup() {
  Serial.begin(115200);

  #ifdef ESP8266
    configTime(0, 0, "pool.ntp.org");      // get UTC time via NTP
    client.setTrustAnchors(&cert); // Add root certificate for api.telegram.org
  #endif

  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, ledState);
  
  // Connect to Wi-Fi
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  #ifdef ESP32
    client.setCACert(TELEGRAM_CERTIFICATE_ROOT); // Add root certificate for api.telegram.org
  #endif
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi..");
  }
  // Print ESP32 Local IP Address
  Serial.println(WiFi.localIP());
}

void loop() {
  if (millis() > lastTimeBotRan + botRequestDelay)  {
    int numNewMessages = bot.getUpdates(bot.last_message_received + 1);

    while(numNewMessages) {
      Serial.println("got response");
      handleNewMessages(numNewMessages);
      numNewMessages = bot.getUpdates(bot.last_message_received + 1);
    }
    lastTimeBotRan = millis();
  }
}

2. Enter your wifi credentials:
```javascript
const char* ssid = "REPLACE_WITH_YOUR_SSID";
const char* password = "REPLACE_WITH_YOUR_PASSWORD";
```
Enter your bot token here:
```javascript
#define BOTtoken "XXXXXXXXXX:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"  // your Bot Token (Get from Botfather)
```
And your User ID:
```javascript
#define CHAT_ID "XXXXXXXXXX"
```
Now you can turn your led on and off with talking to your own Telegram bot. If it doesn't work try to switch HIGH and LOW in the ledstates like here:
```javascript
bool ledState = HIGH;
```
In some boards it works the other way around.
Source:[Randomnerdtutorials.com](https://randomnerdtutorials.com/telegram-control-esp32-esp8266-nodemcu-outputs/)

# Connecting Ledstrip

Now we will connect our ledstrip to our board and change it using our Telegram bot. In this manual we will turn on a rainbow mode.

1. Connect your Ledstrip to your board
2. Open the "strandtest" sketch using File>Examples>Adafruit Neopixel>strandtest
Here will be some lines of code that we will copy to our project.
3. First, paste this in the top of your code:
```javascript
#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
#include <avr/power.h>
#endif
#define LED_PIN    D5
#define LED_COUNT 15
Adafruit_NeoPixel strip(LED_COUNT, LED_PIN, NEO_GRB + NEO_KHZ800);
```
make sure the LED_PIN and the LED_COUNT are right.

4. Now, in this part of the commands you need to add two lines:
```javascript
if (text == "/start") {
      String welcome = "Welcome, " + from_name + ".\n";
      welcome += "Use the following commands to control your outputs.\n\n";
      welcome += "/led_on to turn GPIO ON \n";
      welcome += "/led_off to turn GPIO OFF \n";
      welcome += "/led_rainbow_on to turn GPIO rainbow mode on \n";
      welcome += "/led_rainbow_off to turn GPIO rainbow mode off \n";
      welcome += "/state to request current GPIO state \n";
      bot.sendMessage(chat_id, welcome, "");
    }
    ```
You can see we added the rainbow on and off mode.
5. Below that we will add two if statements to call the right funcions when using the commands:
```javascript
if (text == "/led_rainbow_on") {
      bot.sendMessage(chat_id, "LED state set to RAINBOW ON", "");
      rainbow(10);             // Flowing rainbow cycle along the whole strip
      digitalWrite(ledPin, ledState);
    }

     if (text == "/led_rainbow_off") {
      bot.sendMessage(chat_id, "LED state set to RAINBOW OFF", "");
      strip.clear();
      strip.show();
      digitalWrite(ledPin, ledState);
    }
```
The first statements calls a statement called "rainbow" and that will start the rainbow mode. The second one clears the ledstrip.

6. In the setup we will add some lines to setup the ledstrip:
```javascript
strip.begin();           // INITIALIZE NeoPixel strip object (REQUIRED)
  strip.show();            // Turn OFF all pixels ASAP
  strip.setBrightness(50); // Set BRIGHTNESS to about 1/5 (max = 255)
  stop();
  ```
  The last stop() function I added myself because I ran into a problem where after the setup the first light was on. This function clears the whole strip.
  
  7. At last, all the way at the bottom of our code we will add the two functions we use:
  ```javascript
  void rainbow(int wait) {
  for(long firstPixelHue = 0; firstPixelHue < 5*65536; firstPixelHue += 256) {
    strip.rainbow(firstPixelHue);
    strip.show();
    delay(wait);
  }
}
void stop(){
  strip.clear();
  strip.show();
}
  ```
  The first one is the one that makes the rainbow effect and the second one is a simple function that clears the whole strip.
  
