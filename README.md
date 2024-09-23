# 7-segment



// Define segment pins for 7-segment display
byte A = 2;
byte B = 3;
byte C = 4;
byte D = 5;
byte E = 6;
byte F = 7;
byte G = 8;

// Define button pin
byte PUSHBUTTON = 9;

// Define the dice roll result
byte diceRoll = 1;

// Define an array of digits to display numbers from 0 to 9
const byte digits[10][7] = {
  // A, B, C, D, E, F, G
  {0, 0, 0, 0, 0, 0, 1},  // 0
  {1, 0, 0, 1, 1, 1, 1},  // 1
  {0, 0, 1, 0, 0, 1, 0},  // 2
  {0, 0, 0, 0, 1, 1, 0},  // 3
  {1, 0, 0, 1, 1, 0, 0},  // 4
  {0, 1, 0, 0, 1, 0, 0},  // 5
  {0, 1, 0, 0, 0, 0, 0},  // 6
  {0, 0, 0, 1, 1, 1, 1},  // 7
  {0, 0, 0, 0, 0, 0, 0},  // 8
  {0, 0, 0, 0, 1, 0, 0},  // 9
};

// Function to display a number on 7-segment display
void displayNumber(int num) {
  if (num < 0 || num > 9) return; // Check valid range for countdown
  for (int i = 0; i < 7; i++) {
    digitalWrite(A + i, digits[num][i]); // Map segments to pins
  }
}

void setup() {
  // Set segment pins as output
  for (int i = A; i <= G; i++) {
    pinMode(i, OUTPUT);
  }

  // Set button pin as input
  pinMode(PUSHBUTTON, INPUT_PULLUP); // Use internal pull-up resistor

  // Initialize Serial communication at 9600 baud rate
  Serial.begin(9600);

  // Initialize display with the first number (1)
  displayNumber(1);
}

void loop() {
  static unsigned long lastDebounceTime = 0; // last time the output pin was toggled
  static int buttonState = HIGH; // current state of the button
  static int lastButtonState = HIGH; // previous state of the button
  unsigned long debounceDelay = 50; // debounce delay in milliseconds

  int reading = digitalRead(PUSHBUTTON);

  // Check for a change in button state
  if (reading != lastButtonState) {
    lastDebounceTime = millis(); // reset the debouncing timer
  }

  // If the button state has been stable for the debounce delay
  if ((millis() - lastDebounceTime) > debounceDelay) {
    // If the button state has changed
    if (reading != buttonState) {
      buttonState = reading;

      // Only trigger if the new button state is LOW (button pressed)
      if (buttonState == LOW) {
        // Check if it's a long press
        unsigned long pressStartTime = millis();
        while (digitalRead(PUSHBUTTON) == LOW) {
          // If held down for more than 2 seconds
          if (millis() - pressStartTime >= 2000) {
            // Long press detected, count down from 9 to 0
            for (int i = 9; i >= 0; i--) {
              displayNumber(i);
              delay(1000); // Display each number for 1 second
            }
            break; // Exit while loop for long press
          }
        }

        // If it's a short press, roll the dice
        if (digitalRead(PUSHBUTTON) == HIGH) {
          diceRoll = random(1, 7); // Generate random number between 1 and 6
          displayNumber(diceRoll); // Display the dice roll number

          // Print the result to Serial Monitor
          Serial.print("Dice rolled: ");
          Serial.println(diceRoll);
        }
      }
    }
  }
  lastButtonState = reading; // Save the reading for next loop iteration
}


