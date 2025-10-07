#include <WiFi.h>
#include <WebServer.h>

// Motor Pins
#define ENA 23
#define IN1 22
#define IN2 21
#define ENB 19
#define IN3 18
#define IN4 5

#define s 150
#define t 200

// Create Access Point
const char* ssid = "ESP32_Robot";
const char* password = "12345678";

WebServer server(80);  // HTTP server on port 80

// HTML Page
const char* html = R"rawliteral(
<!DOCTYPE html>
<html>
<head>
  <title>ESP32 Robot Control</title>
  <style>
    body { text-align: center; font-family: sans-serif; }
    button {
      width: 100px; height: 50px; margin: 10px;
      font-size: 18px;
    }
  </style>
</head>
<body>
  <h2>ESP32 Wi-Fi Robot</h2>
  <button onclick="sendCmd('F')">Forward</button><br>
  <button onclick="sendCmd('L')">Left</button>
  <button onclick="sendCmd('S')">Stop</button>
  <button onclick="sendCmd('R')">Right</button><br>
  <button onclick="sendCmd('B')">Backward</button>

  <script>
    function sendCmd(cmd) {
      fetch('/' + cmd);
    }
  </script>
</body>
</html>
)rawliteral";

// -------- Setup --------
void setup() {
  Serial.begin(115200);

  // Motor Pins
  pinMode(ENA, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(ENB, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);

  // Wi-Fi Access Point
  WiFi.softAP(ssid, password);
  IPAddress IP = WiFi.softAPIP();
  Serial.print("AP IP address: ");
  Serial.println(IP);

  // Routes
  server.on("/", []() { server.send(200, "text/html", html); });
  server.on("/F", []() { moveForward(); server.send(200, "text/plain", "OK"); });
  server.on("/B", []() { moveBackward(); server.send(200, "text/plain", "OK"); });
  server.on("/L", []() { turnLeft(); server.send(200, "text/plain", "OK"); });
  server.on("/R", []() { turnRight(); server.send(200, "text/plain", "OK"); });
  server.on("/S", []() { stopMotors(); server.send(200, "text/plain", "OK"); });

  server.begin();
  Serial.println("Web server started");
}

void loop() {
  server.handleClient();
}

// -------- Motor Control --------
void moveForward() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  analogWrite(ENA, s);

  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
  analogWrite(ENB, s);
}

void moveBackward() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  analogWrite(ENA, s);

  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
  analogWrite(ENB, s);
}

void turnLeft() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  analogWrite(ENA, t);

  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
  analogWrite(ENB, t);
}

void turnRight() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  analogWrite(ENA, t);

  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
  analogWrite(ENB, t);
}

void stopMotors() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  analogWrite(ENA, 0);

  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
  analogWrite(ENB, 0);
}
