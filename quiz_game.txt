#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// I²C LCD: 20×4 display
LiquidCrystal_I2C lcd(0x27, 20, 4);

// Remapped button pins so A–D correspond to A3–A0 respectively
const int BTN_A = A3;
const int BTN_B = A2;
const int BTN_C = A1;
const int BTN_D = A0;

// Shift-register pins for LED animation
const int dataPin  = 8;
const int latchPin = 12;
const int clockPin = 9;

// Game states
enum GameState { WELCOME, QUIZ };
GameState currentState = WELCOME;

// Blinking variables
unsigned long previousMillis = 0;
bool textVisible = true;
const int blinkInterval = 500;  // ms

// Quiz variables
int currentQuestion = 0;
const int totalQuestions = 5;

// Helper to clear all LEDs
void clearLeds() {
  digitalWrite(latchPin, LOW);
  shiftOut(dataPin, clockPin, MSBFIRST, 0x00);
  shiftOut(dataPin, clockPin, MSBFIRST, 0x00);
  shiftOut(dataPin, clockPin, MSBFIRST, 0x00);
  digitalWrite(latchPin, HIGH);
}

void setup() {
  // buttons
  pinMode(BTN_A, INPUT_PULLUP);
  pinMode(BTN_B, INPUT_PULLUP);
  pinMode(BTN_C, INPUT_PULLUP);
  pinMode(BTN_D, INPUT_PULLUP);

  // LCD
  Wire.begin();
  lcd.init();
  lcd.backlight();

  // shift register outputs - initialize as early as possible
  pinMode(dataPin, OUTPUT);
  pinMode(latchPin, OUTPUT);
  pinMode(clockPin, OUTPUT);
  
  // Force all pins to known states immediately
  digitalWrite(dataPin, LOW);
  digitalWrite(clockPin, LOW);
  digitalWrite(latchPin, LOW);
  
  // Startup animation to take control of LEDs immediately
  startupAnimation();
  
  delay(500);  // small startup delay
  showWelcomeSequence();
}

void loop() {
  unsigned long now = millis();
  if (currentState == WELCOME && now - previousMillis >= blinkInterval) {
    previousMillis = now;
    textVisible = !textVisible;
    if (textVisible) lcd.display();
    else             lcd.noDisplay();
  }

  if (currentState == WELCOME) {
    handleWelcomeState();
  } else {
    handleQuizState();
  }
}

void showWelcomeSequence() {
  lcd.clear();
  clearLeds();              // ensure LEDs stay off at start
  lcd.setCursor(5, 0);  lcd.print("Welcome!");
  
  // LED fill animation during welcome message
  for (int i = 0; i < 24; i++) {
    uint32_t mask = 0;
    // Light up all LEDs from 0 to current position
    for (int j = 0; j <= i; j++) {
      mask |= ((uint32_t)1 << j);
    }
    
    byte hb = (mask >> 16) & 0xFF;
    byte mb = (mask >> 8) & 0xFF;
    byte lb = mask & 0xFF;
    digitalWrite(latchPin, LOW);
    shiftOut(dataPin, clockPin, MSBFIRST, hb);
    shiftOut(dataPin, clockPin, MSBFIRST, mb);
    shiftOut(dataPin, clockPin, MSBFIRST, lb);
    digitalWrite(latchPin, HIGH);
    delay(80);  // ~2 seconds total for fill (24 * 80ms = 1920ms)
  }
  
  // Pause with all LEDs on for 1 second
  delay(1000);
  lcd.clear();
  clearLeds();  // Turn off LEDs for quiz portion
  lcd.setCursor(4, 0);  lcd.print("Quiz time...");
  delay(2000);
  lcd.setCursor(3, 2);  lcd.print("Are you ready?");
  delay(2000);
  lcd.clear();
  lcd.setCursor(6, 0);  lcd.print("If so...");
  delay(1500);
  lcd.setCursor(2, 2);  lcd.print("Press any button");
  
  // Flash the "Press any button" message
  unsigned long flashStart = millis();
  bool buttonPressed = false;
  while (!buttonPressed) {
    if (millis() - flashStart >= 500) {  // Flash every 500ms
      flashStart = millis();
      textVisible = !textVisible;
      if (textVisible) {
        lcd.setCursor(2, 2);  lcd.print("Press any button");
      } else {
        lcd.setCursor(2, 2);  lcd.print("                ");  // Clear the line
      }
    }
    
    // Check for button press
    if (!digitalRead(BTN_A) || !digitalRead(BTN_B) || 
        !digitalRead(BTN_C) || !digitalRead(BTN_D)) {
      buttonPressed = true;
    }
  }
  lcd.clear();
  lcd.setCursor(4, 0);  lcd.print("Let's Begin!");
  delay(1500);

  currentState = QUIZ;
  showQuestion();
}

void handleWelcomeState() {
  // nothing here; button press handled above
}

void showQuestion() {
  lcd.clear();
  switch (currentQuestion) {
    case 0:
      lcd.setCursor(6,1);  lcd.print("Question");
      delay(2000);
      lcd.clear();
      lcd.setCursor(1,0);  lcd.print("A) Answer B) Answer");
      lcd.setCursor(1,2);  lcd.print("C) Answer D) Answer");
      break;

    case 1:
      lcd.setCursor(6,1);  lcd.print("Question");
      delay(2000);
      lcd.clear();
      lcd.setCursor(1,0);  lcd.print("A) Answer B) Answer");
      lcd.setCursor(1,2);  lcd.print("C) Answer D) Answer");
      break;

    case 2:
      lcd.setCursor(6,1);  lcd.print("Question");
      delay(2000);
      lcd.clear();
      lcd.setCursor(1,0);  lcd.print("A) Answer B) Answer");
      lcd.setCursor(1,2);  lcd.print("C) Answer D) Answer");
      break;

    case 3:
      lcd.setCursor(6,1);  lcd.print("Question");
      delay(2000);
      lcd.clear();
      lcd.setCursor(1,0);  lcd.print("A) Answer B) Answer");
      lcd.setCursor(1,2);  lcd.print("C) Answer D) Answer");
      break;

    case 4:
      lcd.setCursor(6,1);  lcd.print("Question");
      delay(2000);
      lcd.clear();
      lcd.setCursor(1,0);  lcd.print("A) Answer B) Answer");
      lcd.setCursor(1,2);  lcd.print("C) Answer D) Answer");
      break;
  }
}

void handleQuizState() {
  if (!digitalRead(BTN_A)) { processAnswer('A'); while (!digitalRead(BTN_A)); }
  else if (!digitalRead(BTN_B)) { processAnswer('B'); while (!digitalRead(BTN_B)); }
  else if (!digitalRead(BTN_C)) { processAnswer('C'); while (!digitalRead(BTN_C)); }
  else if (!digitalRead(BTN_D)) { processAnswer('D'); while (!digitalRead(BTN_D)); }
}

void processAnswer(char answer) {
  lcd.clear();
  lcd.setCursor(0,0);

  // Replace 'A' with the correct answer for each question
  switch (currentQuestion) {
    case 0:
      if (answer == 'A') {  // Change to correct answer letter
        lcd.print("Correct!");
        correctAnswerBlinks();
      } else {
        lcd.print("Incorrect!");
      }
      break;
    case 1:
      if (answer == 'A') {  // Change to correct answer letter
        lcd.print("Correct!");
        correctAnswerBlinks();
      } else {
        lcd.print("Incorrect!");
      }
      break;
    case 2:
      if (answer == 'A') {  // Change to correct answer letter
        lcd.print("Correct!");
        correctAnswerBlinks();
      } else {
        lcd.print("Incorrect!");
      }
      break;
    case 3:
      if (answer == 'A') {  // Change to correct answer letter
        lcd.print("Correct!");
        correctAnswerBlinks();
      } else {
        lcd.print("Incorrect!");
      }
      break;
    case 4:
      if (answer == 'A') {  // Change to correct answer letter
        lcd.print("Correct!");
        correctAnswerBlinks();
      } else {
        lcd.print("Incorrect!");
      }
      break;
  }

  delay(2000);
  currentQuestion++;

  if (currentQuestion < totalQuestions) {
    showQuestion();
  } else {
    // Quiz complete
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Quiz complete!");
    delay(2000);
    lcd.clear();
    lcd.setCursor(6,1);
    lcd.print("Message");

    // LED animation sequence: 3 cycles of (circle + dancing)
    for (int cycle = 0; cycle < 3; cycle++) {
      // Circle animation (3 times)
      for (int circleRound = 0; circleRound < 3; circleRound++) {
        for (int i = 0; i < 24; i++) {
          uint32_t mask = (uint32_t)1 << i;
          byte hb = (mask >> 16) & 0xFF;
          byte mb = (mask >> 8) & 0xFF;
          byte lb = mask & 0xFF;
          digitalWrite(latchPin, LOW);
          shiftOut(dataPin, clockPin, MSBFIRST, hb);
          shiftOut(dataPin, clockPin, MSBFIRST, mb);
          shiftOut(dataPin, clockPin, MSBFIRST, lb);
          digitalWrite(latchPin, HIGH);
          delay(33);  // 50/1.5 ≈ 33ms for 1.5x speed
        }
      }
      
      // Dancing animation - alternating odds/evens (3 times)
      for (int danceRound = 0; danceRound < 3; danceRound++) {
        // Show odd LEDs (bits 0, 2, 4, 6, ...)
        digitalWrite(latchPin, LOW);
        shiftOut(dataPin, clockPin, MSBFIRST, 0xAA); // 10101010
        shiftOut(dataPin, clockPin, MSBFIRST, 0xAA); // 10101010
        shiftOut(dataPin, clockPin, MSBFIRST, 0xAA); // 10101010
        digitalWrite(latchPin, HIGH);
        delay(200);  // 300/1.5 = 200ms for 1.5x speed
        
        // Show even LEDs (bits 1, 3, 5, 7, ...)
        digitalWrite(latchPin, LOW);
        shiftOut(dataPin, clockPin, MSBFIRST, 0x55); // 01010101
        shiftOut(dataPin, clockPin, MSBFIRST, 0x55); // 01010101
        shiftOut(dataPin, clockPin, MSBFIRST, 0x55); // 01010101
        digitalWrite(latchPin, HIGH);
        delay(200);  // 300/1.5 = 200ms for 1.5x speed
      }
    }

    // turn all LEDs off
    clearLeds();
    // prompt to play again with flashing text
    lcd.clear();
    lcd.setCursor((20-11)/2,0); lcd.print("Play again?");
    
    // Flash the "Press any button" message
    unsigned long flashStart = millis();
    bool buttonPressed = false;
    textVisible = true;  // Reset text visibility
    
    while (!buttonPressed) {
      if (millis() - flashStart >= 500) {  // Flash every 500ms
        flashStart = millis();
        textVisible = !textVisible;
        if (textVisible) {
          lcd.setCursor((20-16)/2,2); lcd.print("Press any button");
        } else {
          lcd.setCursor((20-16)/2,2); lcd.print("                ");  // Clear the line
        }
      }
      
      // Check for button press
      if (!digitalRead(BTN_A) || !digitalRead(BTN_B) || 
          !digitalRead(BTN_C) || !digitalRead(BTN_D)) {
        buttonPressed = true;
      }
    }

    // restart quiz
    currentQuestion = 0;
    currentState = WELCOME;
    showWelcomeSequence();
  }
}

void runLedAnimation() {
  // unused
}

void startupAnimation() {
  // Clear any random states immediately
  clearLeds();
  delay(50);
}

void correctAnswerBlinks() {
  // 3 fast blinks on all LEDs
  for (int blink = 0; blink < 3; blink++) {
    // All LEDs on
    digitalWrite(latchPin, LOW);
    shiftOut(dataPin, clockPin, MSBFIRST, 0xFF);
    shiftOut(dataPin, clockPin, MSBFIRST, 0xFF);
    shiftOut(dataPin, clockPin, MSBFIRST, 0xFF);
    digitalWrite(latchPin, HIGH);
    delay(150);
    
    // All LEDs off
    clearLeds();
    delay(150);
  }
}

void singleBlink() {
  // Single blink for fake-out correct answer (unused in template)
  digitalWrite(latchPin, LOW);
  shiftOut(dataPin, clockPin, MSBFIRST, 0xFF);
  shiftOut(dataPin, clockPin, MSBFIRST, 0xFF);
  shiftOut(dataPin, clockPin, MSBFIRST, 0xFF);
  digitalWrite(latchPin, HIGH);
  delay(150);
  
  clearLeds();
  delay(150);
}
