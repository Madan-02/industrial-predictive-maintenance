# industrial-predictive-maintenance
ESP32-based industrial machine health monitoring and predictive maintenance simulation
#define TEMP 35
#define VIB 34
#define LOAD 33

#define STEP 25
#define EN 27
#define RELAY 23
#define BUZZER 4
#define RED 16
#define GREEN 17
#define BUTTON 32

bool fault = false;

void setup() {
  Serial.begin(115200);

  pinMode(STEP, OUTPUT);
  pinMode(EN, OUTPUT);
  pinMode(RELAY, OUTPUT);
  pinMode(BUZZER, OUTPUT);
  pinMode(RED, OUTPUT);
  pinMode(GREEN, OUTPUT);
  pinMode(BUTTON, INPUT_PULLUP);

  digitalWrite(EN, LOW);
  digitalWrite(RELAY, HIGH);
  digitalWrite(RED, LOW);
  digitalWrite(GREEN, HIGH);

  noTone(BUZZER);
}

void loop() {

  // =========================
  // FAULT STATE
  // =========================
  if (fault) {

    digitalWrite(EN, HIGH);       // STOP MOTOR
    digitalWrite(RELAY, LOW);     // RELAY OFF
    digitalWrite(RED, HIGH);      // RED ON
    digitalWrite(GREEN, LOW);     // GREEN OFF

    tone(BUZZER, 2000);           // BUZZER ON

    // Reset using button
    if (digitalRead(BUTTON) == LOW) {
      fault = false;
      noTone(BUZZER);

      digitalWrite(EN, LOW);
      digitalWrite(RELAY, HIGH);
      digitalWrite(RED, LOW);
      digitalWrite(GREEN, HIGH);

      delay(300);
    }

    return;
  }

  // =========================
  // READ 3 POTENTIOMETERS
  // =========================

  int temperature = map(
    analogRead(TEMP), 0, 4095, 20, 100
  );

  int vibration = map(
    analogRead(VIB), 0, 4095, 0, 30
  );

  int load = map(
    analogRead(LOAD), 0, 4095, 0, 100
  );

  // =========================
  // ANOMALY SCORE
  // =========================

  int score = 0;

  // Temperature
  if (temperature >= 70)
    score += 35;
  else if (temperature >= 50)
    score += 20;

  // Vibration
  if (vibration >= 20)
    score += 35;
  else if (vibration >= 10)
    score += 20;

  // Load
  if (load >= 80)
    score += 30;
  else if (load >= 60)
    score += 20;

  // =========================
  // SERIAL OUTPUT
  // =========================

  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.print(" C | Vibration: ");
  Serial.print(vibration);
  Serial.print(" | Load: ");
  Serial.print(load);
  Serial.print(" | Score: ");
  Serial.println(score);

  // =========================
  // FAULT >= 70
  // =========================

  if (score >= 70) {

    fault = true;

    digitalWrite(EN, HIGH);       // MOTOR STOPS
    digitalWrite(RELAY, LOW);     // RELAY OFF
    digitalWrite(RED, HIGH);
    digitalWrite(GREEN, LOW);

    tone(BUZZER, 2000);           // BUZZER ON

    Serial.println("!!! FAULT - MACHINE STOPPED !!!");

    return;
  }

  // =========================
  // CRITICAL 40-69
  // =========================

  else if (score >= 40) {

    digitalWrite(RED, HIGH);
    digitalWrite(GREEN, LOW);

    noTone(BUZZER);

    Serial.println("STATUS: CRITICAL");
  }

  // =========================
  // NORMAL 0-39
  // =========================

  else {

    digitalWrite(RED, LOW);
    digitalWrite(GREEN, HIGH);

    noTone(BUZZER);

    Serial.println("STATUS: NORMAL");
  }

  // =========================
  // RUN MOTOR
  // =========================

  digitalWrite(EN, LOW);
  digitalWrite(RELAY, HIGH);

  digitalWrite(STEP, HIGH);
  delayMicroseconds(800);

  digitalWrite(STEP, LOW);
  delayMicroseconds(800);

  delay(300);
}
