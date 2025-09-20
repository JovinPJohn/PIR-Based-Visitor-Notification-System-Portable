SENDER ESP32

#include <WiFi.h>
#include <PubSubClient.h>

#define PIR_PIN 2

const char* ssid = "";
const char* password = "";
const char* mqtt_server = "broker.hivemq.com";
const char* topic = "motion/sensor/alert";

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  Serial.begin(115200);
  pinMode(PIR_PIN, INPUT);
  
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");
  
  client.setServer(mqtt_server, 1883);
  
  while (!client.connected()) {
    if (client.connect("PIRSensor")) {
      Serial.println("Connected to MQTT");
    } else {
      delay(5000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    client.connect("PIRSensor");
  }
  client.loop();
  
  if (digitalRead(PIR_PIN)) {
    Serial.println("PIR triggered - sending MQTT message");
    client.publish(topic, "MOTION_DETECTED");
    delay(1000);
  }
}


RECEIVER ESP32

#include <WiFi.h>
#include <PubSubClient.h>

#define BUZZER_PIN 4

const char* ssid = "";
const char* password = "";
const char* mqtt_server = "broker.hivemq.com";
const char* topic = "motion/sensor/alert";

WiFiClient espClient;
PubSubClient client(espClient);

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.println("Motion alert received!");
  digitalWrite(BUZZER_PIN, HIGH);
  delay(2000);
  digitalWrite(BUZZER_PIN, LOW);
}

void setup() {
  Serial.begin(115200);
  pinMode(BUZZER_PIN, OUTPUT);
  
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  
  while (!client.connected()) {
    if (client.connect("BuzzerReceiver")) {
      client.subscribe(topic);
      Serial.println("Connected and subscribed to MQTT");
    } else {
      delay(5000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    client.connect("BuzzerReceiver");
    client.subscribe(topic);
  }
  client.loop();
}
