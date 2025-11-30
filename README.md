# AutoRobo
#include <Adafruit_VL53L0X.h>

Adafruit_VL53L0X lox = Adafruit_VL53L0X();

// -----------------------------
// Motor Pins
// -----------------------------
int ENA = 5;
int IN1 = 6;
int IN2 = 7;

int ENB = 3;
int IN3 = 8;
int IN4 = 9;

// -----------------------------
// Ultrasonic Right
// -----------------------------
int trigR = 10;
int echoR = 11;

// Ultrasonic Left
int trigL = 12;
int echoL = 13;

// -----------------------------
// Functions
// -----------------------------
long getDistance(int trig, int echo) {
  digitalWrite(trig, LOW);
  delayMicroseconds(2);

  digitalWrite(trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(trig, LOW);

  long duration = pulseIn(echo, HIGH, 25000);
  long distance = duration * 0.0343 / 2;
  return distance;
}

// Motor controls
void forward() {
  analogWrite(ENA, 150);
  analogWrite(ENB, 150);

  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);

  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void stopCar() {
  analogWrite(ENA, 0);
  analogWrite(ENB, 0);
}

void turnLeft() {
  analogWrite(ENA, 150);
  analogWrite(ENB, 150);

  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);

  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void turnRight() {
  analogWrite(ENA, 150);
  analogWrite(ENB, 150);

  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);

  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
}

// -----------------------------
// SETUP
// -----------------------------
void setup() {
  Serial.begin(9600);

  pinMode(ENA, OUTPUT);
  pinMode(ENB, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);

  pinMode(trigR, OUTPUT);
  pinMode(echoR, INPUT);

  pinMode(trigL, OUTPUT);
  pinMode(echoL, INPUT);

  // Start VL53L0X
  if (!lox.begin()) {
    Serial.println("Failed to boot VL53L0X");
    while (1);
  }
  Serial.println("VL53L0X Ready!");

  stopCar();
}

// -----------------------------
// LOOP
// -----------------------------
void loop() {

  // Read front distance from VL53L0X
  VL53L0X_RangingMeasurementData_t measure;
  lox.rangingTest(&measure, false);

  int frontDist = measure.RangeMilliMeter / 10;  // mm → cm

  long rightDist = getDistance(trigR, echoR);
  long leftDist = getDistance(trigL, echoL);

  Serial.print("Front: "); Serial.print(frontDist);
  Serial.print(" cm | Right: "); Serial.print(rightDist);
  Serial.print(" cm | Left: "); Serial.println(leftDist);

  // -----------------------------
  // MAIN LOGIC
  // -----------------------------

  if (frontDist <= 10 && frontDist > 0) {
    stopCar();
    delay(300);

    if (rightDist <= 10 && leftDist <= 10) {
      stopCar();
    }
    else if (rightDist <= 10) {
      turnLeft();
      delay(400);
    }
    else if (leftDist <= 10) {
      turnRight();
      delay(400);
    }
    else {
      turnLeft();     // No obstacle → move left
      delay(400);
    }
  }
  else {
    forward();
  }

  delay(50);
}
