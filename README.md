# BUILD-A-BASIC-HEATER-CONTROL-SYSTEM
#include <DHT.h>

#define DHTPIN 2         // DHT22 data pin
#define DHTTYPE DHT22

#define HEATER_PIN 3     // Heater LED
#define BUZZER_PIN 4     // Overheat buzzer

DHT dht(DHTPIN, DHTTYPE);

// Thresholds
const float LOWER_THRESHOLD = 20.0;
const float UPPER_THRESHOLD = 25.0;
const float OVERHEAT_THRESHOLD = 35.0;

enum SystemState {
  IDLE,
  HEATING,
  STABILIZING,
  TARGET_REACHED,
  OVERHEAT
};

SystemState currentState = IDLE;

void setup() {
  Serial.begin(9600);
  dht.begin();
  pinMode(HEATER_PIN, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);
}

void loop() {
  float temperature = dht.readTemperature();

  if (isnan(temperature)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println("°C");

  // State transitions
  if (temperature < LOWER_THRESHOLD) {
    currentState = HEATING;
  } else if (temperature >= LOWER_THRESHOLD && temperature < UPPER_THRESHOLD) {
    currentState = STABILIZING;
  } else if (temperature >= UPPER_THRESHOLD && temperature < OVERHEAT_THRESHOLD) {
    currentState = TARGET_REACHED;
  } else {
    currentState = OVERHEAT;
  }

  // Actions based on state
  switch (currentState) {
    case IDLE:
      digitalWrite(HEATER_PIN, LOW);
      digitalWrite(BUZZER_PIN, LOW);
      Serial.println("State: IDLE");
      break;

    case HEATING:
      digitalWrite(HEATER_PIN, HIGH);
      digitalWrite(BUZZER_PIN, LOW);
      Serial.println("State: HEATING");
      break;

    case STABILIZING:
      digitalWrite(HEATER_PIN, HIGH);
      digitalWrite(BUZZER_PIN, LOW);
      Serial.println("State: STABILIZING");
      break;

    case TARGET_REACHED:
      digitalWrite(HEATER_PIN, LOW);
      digitalWrite(BUZZER_PIN, LOW);
      Serial.println("State: TARGET REACHED");
      break;

    case OVERHEAT:
      digitalWrite(HEATER_PIN, LOW);
      digitalWrite(BUZZER_PIN, HIGH);
      Serial.println("State: OVERHEAT! Heater OFF, Buzzer ON!");
      break;
  }

  delay(1000); // 1 second delay
}
