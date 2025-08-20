#include <ESP8266WiFi.h>
#include <ESPAsyncTCP.h>
#include <ESPAsyncWebServer.h>
#include <LiquidCrystal.h>

// LCD setup
LiquidCrystal lcd(D0, D1, D2, D3, D4, D5);

// WebServer
AsyncWebServer server(80);

// WiFi credentials
const char* ssid = "HARSHITH_K";
const char* password = "Sagarshr";

// Parameters
const char* PARAM_INPUT_1 = "input1";
const char* PARAM_LED = "led";

// LED pin
const int ledPin = D6;  // you can change to any GPIO pin

// HTML page
const char index_html[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html>
<head>
  <title>Smart Notice Board + LED</title>
</head>
<body>
  <h1>Smart Notice Board</h1>
  <form action="/get">
    <label>Enter Text to Display:</label><br>
    <input type="text" name="input1">
    <input type="submit" value="Send">
  </form>
  <h2>LED Control</h2>
  <a href="/led?state=on"><button>Turn ON</button></a>
  <a href="/led?state=off"><button>Turn OFF</button></a>
</body>
</html>
)rawliteral";

// Not Found handler
void notFound(AsyncWebServerRequest *request) {
  request->send(404, "text/plain", "Not found");
}

void setup() {
  Serial.begin(115200);

  // LCD setup
  lcd.begin(16, 2);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Smart Notice Board");

  // LED setup
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);

  // WiFi
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  if (WiFi.waitForConnectResult() != WL_CONNECTED) {
    Serial.println("WiFi Failed!");
    return;
  }
  Serial.println();
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());

  // Root page
  server.on("/", HTTP_GET, [](AsyncWebServerRequest *request){
    request->send_P(200, "text/html", index_html);
  });

  // Handle text input
  server.on("/get", HTTP_GET, [] (AsyncWebServerRequest *request) {
    String message;
    if (request->hasParam(PARAM_INPUT_1)) {
      message = request->getParam(PARAM_INPUT_1)->value();
      lcd.clear();
      lcd.setCursor(0,0);
      lcd.print(message);
    } else {
      message = "No message sent";
    }
    Serial.println("LCD Message: " + message);
    request->send(200, "text/html", index_html);
  });

  // Handle LED control
  server.on("/led", HTTP_GET, [] (AsyncWebServerRequest *request) {
    if (request->hasParam("state")) {
      String state = request->getParam("state")->value();
      if (state == "on") {
        digitalWrite(ledPin, HIGH);
        Serial.println("LED ON");
      } else {
        digitalWrite(ledPin, LOW);
        Serial.println("LED OFF");
      }
    }
    request->send(200, "text/html", index_html);
  });

  // Not Found
  server.onNotFound(notFound);

  // Start server
  server.begin();
}

void loop() {
  // Scroll LCD text
  for (int positionCounter = 0; positionCounter < 29; positionCounter++) {
    lcd.scrollDisplayLeft();
    delay(500);
  }
}
