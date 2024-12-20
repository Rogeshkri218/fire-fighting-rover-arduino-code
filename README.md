# fire-fighting-rover-arduino-code
#define enA 10 // Enable1 L298 Pin enA
#define in1 9  // Motor1 L298 Pin in1
#define in2 8  // Motor1 L298 Pin in2
#define in3 7  // Motor2 L298 Pin in3
#define in4 6  // Motor2 L298 Pin in4
#define enB 5  // Enable2 L298 Pin enB
#define ir_R A0
#define ir_F A1
#define ir_L A2
#define servo A4
#define pump A5

int Speed = 160; // Write The Duty Cycle 0 to 255 Enable for Motor Speed
int s1, s2, s3;

void setup() {
  // Initialize serial communication
  Serial.begin(9600); 

  // Pin mode declarations
  pinMode(ir_R, INPUT);
  pinMode(ir_F, INPUT);
  pinMode(ir_L, INPUT);
  pinMode(enA, OUTPUT);
  pinMode(in1, OUTPUT);
  pinMode(in2, OUTPUT);
  pinMode(in3, OUTPUT);
  pinMode(in4, OUTPUT);
  pinMode(enB, OUTPUT);
  pinMode(servo, OUTPUT);
  pinMode(pump, OUTPUT);

  // Servo movement to initial positions
  for (int angle = 90; angle <= 140; angle += 5) {
    servoPulse(servo, angle);
  }
  for (int angle = 140; angle >= 40; angle -= 5) {
    servoPulse(servo, angle);
  }
  for (int angle = 40; angle <= 95; angle += 5) {
    servoPulse(servo, angle);
  }

  // Set initial motor speeds
  analogWrite(enA, Speed); 
  analogWrite(enB, Speed); 
  delay(500);
}

void loop() {
  // Read sensor values
  s1 = analogRead(ir_R);
  s2 = analogRead(ir_F);
  s3 = analogRead(ir_L);

  // Print sensor values for debugging
  Serial.print(s1);
  Serial.print("\t");
  Serial.print(s2);
  Serial.print("\t");
  Serial.println(s3);
  delay(50);

  // Decision making based on sensor values
  if (s1 < 250) {
    Stop();
    digitalWrite(pump, HIGH);
    sweepServo(40, 90, servo);
  } 
  else if (s2 < 350) {
    Stop();
    digitalWrite(pump, HIGH);
    sweepServo(90, 140, servo);
    sweepServo(140, 40, servo);
    sweepServo(40, 90, servo);
  } 
  else if (s3 < 250) {
    Stop();
    digitalWrite(pump, HIGH);
    sweepServo(90, 140, servo);
    sweepServo(140, 90, servo);
  } 
  else if (s1 >= 251 && s1 <= 700) {
    digitalWrite(pump, LOW);
    backword();
    delay(100);
    turnRight();
    delay(200);
  } 
  else if (s2 >= 251 && s2 <= 800) {
    digitalWrite(pump, LOW);
    forword();
  } 
  else if (s3 >= 251 && s3 <= 700) {
    digitalWrite(pump, LOW);
    backword();
    delay(100);
    turnLeft();
    delay(200);
  } 
  else {
    digitalWrite(pump, LOW);
    Stop();
  }
  
  delay(10);
}

// Function to control servo pulse
void servoPulse(int pin, int angle) {
  int pwm = (angle * 11) + 500; // Convert angle to pulse width in microseconds
  digitalWrite(pin, HIGH);
  delayMicroseconds(pwm);
  digitalWrite(pin, LOW);
  delay(50); // Refresh cycle for servo
}

// Function to sweep servo between two angles
void sweepServo(int startAngle, int endAngle, int pin) {
  int step = (startAngle < endAngle) ? 3 : -3; 
  for (int angle = startAngle; angle != endAngle; angle += step) {
    servoPulse(pin, angle);
  }
}

// Function to move forward
void forword() {
  digitalWrite(in1, HIGH); // Right Motor forward
  digitalWrite(in2, LOW);  // Right Motor backward
  digitalWrite(in3, LOW);  // Left Motor backward
  digitalWrite(in4, HIGH); // Left Motor forward
}

// Function to move backward
void backword() {
  digitalWrite(in1, LOW);  // Right Motor forward
  digitalWrite(in2, HIGH); // Right Motor backward
  digitalWrite(in3, HIGH); // Left Motor backward
  digitalWrite(in4, LOW);  // Left Motor forward
}

// Function to turn right
void turnRight() {
  digitalWrite(in1, LOW);  // Right Motor forward
  digitalWrite(in2, HIGH); // Right Motor backward
  digitalWrite(in3, LOW);  // Left Motor backward
  digitalWrite(in4, HIGH); // Left Motor forward
}

// Function to turn left
void turnLeft() {
  digitalWrite(in1, HIGH); // Right Motor forward
  digitalWrite(in2, LOW);  // Right Motor backward
  digitalWrite(in3, HIGH); // Left Motor backward
  digitalWrite(in4, LOW);  // Left Motor forward
}

// Function to stop the motors
void Stop() {
  digitalWrite(in1, LOW); // Right Motor forward
  digitalWrite(in2, LOW); // Right Motor backward
  digitalWrite(in3, LOW); // Left Motor backward
  digitalWrite(in4, LOW); // Left Motor forward
}
