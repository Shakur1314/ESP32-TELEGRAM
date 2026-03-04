ESP32-TELEGRAM
ESP32-CAM Telegram Bot: remote control via Telegram. Features: capture photos, toggle flash, monitor status (IP/signal/memory), Chat ID auth. Uses ESP32-CAM AI-Thinker, FTDI programmer, UniversalTelegramBot v1.3+, ArduinoJson v6. Setup: create bot, get Chat ID, configure WiFi, upload code. Commands: /photo, /flash, /status.

Objective
The goal of this project was to deploy a remote-controlled surveillance node using the ESP32-CAM and the Telegram Bot API. This lab demonstrates the implementation of remote command execution, manual HTTP POST request construction, and authorized access control in an IoT environment.

Tools Used
Hardware: ESP32-CAM (AI-Thinker module), FTDI USB-to-TTL Programmer.

Software/APIs: Arduino IDE, Telegram Bot API, UniversalTelegramBot and ArduinoJson libraries.

Protocols: WiFi (WPA2), TLS/SSL (HTTPS).

Technical Implementation
1. Authorization Logic (Chat ID Filtering)
To ensure the camera only interacts with an authorized user, I implemented a filter that checks the incoming chat_id.

C++
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
2. Manual HTTP POST and Image Streaming
Instead of relying solely on a high-level library, I handled the manual construction of the multipart HTTP POST request to send image data to Telegram's servers. This demonstrates an understanding of how data is chunked and transmitted over TCP.

C++
// Constructing the multipart/form-data request
String head = "--RandomNerdTutorials\r\nContent-Disposition: form-data; name=\"chat_id\"; \r\n\r\n" + CHAT_ID + "\r\n--RandomNerdTutorials\r\nContent-Disposition: form-data; name=\"photo\"; filename=\"esp32-cam.jpg\"\r\nContent-Type: image/jpeg\r\n\r\n";
String tail = "\r\n--RandomNerdTutorials--\r\n";

clientTCP.println("POST /bot"+BOTtoken+"/sendPhoto HTTP/1.1");
clientTCP.println("Host: " + String(myDomain));
clientTCP.println("Content-Length: " + String(totalLen));
clientTCP.println("Content-Type: multipart/form-data; boundary=RandomNerdTutorials");
clientTCP.println();
clientTCP.print(head);

// Sending the image buffer in 1024-byte chunks
uint8_t *fbBuf = fb->buf;
size_t fbLen = fb->len;
for (size_t n=0; n<fbLen; n=n+1024) {
  if (n+1024<fbLen) {
    clientTCP.write(fbBuf, 1024);
    fbBuf += 1024;
  }
  else if (fbLen%1024>0) {
    size_t remainder = fbLen%1024;
    clientTCP.write(fbBuf, remainder);
  }
}  
clientTCP.print(tail);
Technical Reflections
Why I used WiFiClientSecure
I utilized WiFiClientSecure to ensure all data is encrypted via TLS (Transport Layer Security). This prevents unauthorized interception of the surveillance feed during transit.

Buffer Management
Handling image data on an IoT device requires careful memory management. I ensured esp_camera_fb_return(fb) was called immediately after transmission to free up the frame buffer and prevent memory allocation failures or brownouts.

Real-World Application (SOC Perspective)
As a BCC Cybersecurity graduate, I recognize that unsecured IoT devices are prime targets for lateral movement within a network. This lab highlights:

Access Control: Hardcoded Chat ID prevents unauthorized remote command execution.

Encrypted Transmission: Using port 443 (HTTPS) ensures confidentiality.

Robust Logic: Manual chunking of data prevents packet loss during high-latency transmissions.

Troubleshooting and Roadblocks
Brownout Errors
Fixed by disabling the brownout detector via software. This was necessary because the camera's power draw during a WiFi burst often dropped the voltage below the default threshold.

Timeout Logic
I implemented a 10-second timeout loop to handle responses from Telegram’s API, ensuring the device doesn't hang if the connection is slow.

C++
while ((startTimer + waitTime) > millis()){
  while (clientTCP.available()) {
    char c = clientTCP.read();
    // Logic to capture response body...
    startTimer = millis(); 
  }
  if (getBody.length()>0) break;
}
