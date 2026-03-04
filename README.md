# ESP32-TELEGRAM

ESP32-CAM Telegram Bot: remote control via Telegram. Features: capture photos, toggle flash, monitor status (IP/signal/memory), Chat ID auth. Uses ESP32-CAM AI-Thinker, FTDI programmer, UniversalTelegramBot v1.3+, ArduinoJson v6. Setup: create bot, get Chat ID, configure WiFi, upload code. Commands: /photo, /flash, /status.

---

## Objective
The goal of this project was to deploy a remote-controlled surveillance node using the ESP32-CAM and the Telegram Bot API. This lab demonstrates the implementation of remote command execution, manual HTTP POST request construction, and authorized access control in an IoT environment.

## Tools Used
* **Hardware**: ESP32-CAM (AI-Thinker module), FTDI USB-to-TTL Programmer.
* **Software/APIs**: Arduino IDE, Telegram Bot API, UniversalTelegramBot and ArduinoJson libraries.
* **Protocols**: WiFi (WPA2), TLS/SSL (HTTPS).

---

## Technical Implementation

### 1. Authorization Logic (Chat ID Filtering)
To ensure the camera only interacts with an authorized user, I implemented a filter that checks the incoming chat_id.

```cpp
// Print the received message and handle logic
String text = bot.messages[i].text;
Serial.println(text);

String from_name = bot.messages[i].from_name;
if (text == "/start") {
  String welcome = "Welcome, " + from_name + "\n";
  welcome += "Use the following commands to interact with the ESP32-CAM \n";
  welcome += "/photo : takes a new photo\n";
  welcome += "/flash : toggles flash LED \n";
  bot.sendMessage(CHAT_ID, welcome, "");
}
if (text == "/flash") {
  flashState = !flashState;
  digitalWrite(FLASH_LED_PIN, flashState);
  Serial.println("Change flash LED state");
}
if (text == "/photo") {
  sendPhoto = true;
  Serial.println("New photo request");
}
