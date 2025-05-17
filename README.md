# UART
#include <WiFi.h>
#include <WebServer.h>

const char* ssid = "oppo reno 5 5G";
const char* password = "vhann1";

WebServer server(80);

void setup() {
  Serial.begin(115200); // Debug console
  Serial2.begin(9600, SERIAL_8N1, 16, 17); // UART2 on GPIO16(RX), GPIO17(TX)
  
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  Serial.println("\nWiFi Connected!");
  Serial.print("IP: "); Serial.println(WiFi.localIP());

  server.on("/", HTTP_GET, [](){
    server.send(200, "text/html", 
      "<h1>LED Control</h1>"
      "<a href='/on'><button>ON</button></a>"
      "<a href='/off'><button>OFF</button></a>");
  });

  server.on("/on", [](){
    Serial2.println("ON"); // Send to Arduino
    server.send(200, "text/plain", "LED ON");
  });

  server.on("/off", [](){
    Serial2.println("OFF");
    server.send(200, "text/plain", "LED OFF");
  });

  server.begin();
}

void loop() {
  server.handleClient();
}



void setup() {
  Serial.begin(9600);  // For debugging (optional)
  pinMode(8, OUTPUT); // LED on Pin 8
}

void loop() {
  if (Serial.available()) {  // If data received from ESP32
    String command = Serial.readStringUntil('\n');
    command.trim();  // Remove extra spaces/newlines

    if (command == "ON") {
      digitalWrite(8, HIGH);  
      Serial.println("LED ON");
    } 
    else if (command == "OFF") {
      digitalWrite(8, LOW);   // Turn LED OFF
      Serial.println("LED OFF");
    }
  }
}
