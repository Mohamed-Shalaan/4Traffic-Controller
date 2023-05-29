#include<TimerOne.h>

// Each clolor index in the array
const int RED = 0;
const int YELLOW = 1;
const int GREEN = 2;

//  The ultrasonic sensors directions
const int UP = 0;
const int RIGHT = 1;
const int DOWN = 2;
const int LEFT = 3;


// The timer period is set at 100 milli seconds.
const int TIME_PERIOD_MILLIS = 100;

// Distance under which it will look for vehicles.
const int FARTHEST_DISTANCE = 5;
const int signalsPins[4][3] = {{6, 5, 4}, {17, 18, 19}, {14, 15, 16}, {13, 12, 11} };

// The time that the light takes in millisecods.
int greenDelay = 5000;
int yellowDelay = 2000;

// Trigger and echo for each ultrasonic sensor. 
volatile int[] triggerpin = { 8, 2, 0, 10};
volatile int[] echopin = { 7, 3, 1, 9};

// Variables for storing the distance covered
volatile int[] SignalsDistance;

void setup (){
  Serial.begin (115200);
  //Begin using the timer. This function must be called first.
  Timer1.initialize (TIME_PERIOD_MILLIS * 1000);

  //Run a function each time the timer period finishes.
  Timer1.attachInterrupt (softInterr);

  // Declaring LED pins as output
    for (int dir = 0; dir < 4; i++){
        for (int color = 0; color < 3; j++){
	        pinMode (signalsPins[dir][color], OUTPUT);
	    }
    }

  // Declaring ultrasonic sensor pins as input/output
    for (int i = 0; i < 4; i++) {
        pinMode (triggerpin[i], OUTPUT);
        pinMode (echopin[i], INPUT);
    }
}

// This is interrupt function and it will run each time the timer period finishes.
void softInterr (){
  // Reading from first ultrasonic sensor (Up)
  SignalsDistance[UP] = readDistanceFrom(UP);

  // Reading from second ultrasonic sensor (Right)
  SignalsDistance[RIGHT] = readDistanceFrom(RIGHT);

  // Reading from third ultrasonic sensor (Down)
  SignalsDistance[DOWN] = readDistanceFrom(DOWN);

  // Reading from fourth ultrasonic sensor (Left)
  SignalsDistance[LEFT] = readDistanceFrom(LEFT);

  // Print distance values on serial monitor for debugging
  print (SignalsDistance);
}

void loop (){
    // If there are vehicles at any signal
    for (int dir = 0; dir < count; i++) {
        if (SignalsDistance[dir] < FARTHEST_DISTANCE){
	        signalsFunction(dir);
	    }
    }
}

void signalsFunction (int direction) {
  Serial.println (direction);
  resetAll ();
  // Make RED LED LOW and make Green HIGH for 5 seconds
  digitalWrite (signals[direction][RED], LOW);
  digitalWrite (signals[direction][GREEN], HIGH);
  delay (greenDelay);
  // if there are vehicels at other signals
  bool anotherRoadHasTraffic = false;
  for (int dir = 0; dir < 4; i++) {
      if(dir!= direction && SignalsDistance[i] < FARTHEST_DISTANCE){
          anotherRoadHasTraffic=true;
          break;
      }
  }
  if (anotherRoadHasTraffic){
      // Make Green LED LOW and make yellow LED HIGH for 2 seconds
      digitalWrite (signals[direction][GREEN], LOW);
      digitalWrite (signals[direction][YELLOW], HIGH);
      delay (yellowDelay);
    }
}

int readDistanceFrom (int direction) {
  digitalWrite (triggerpin[direction], LOW);
  delayMicroseconds (2);
  digitalWrite (triggerpin[direction], HIGH);
  delayMicroseconds (10);
  digitalWrite (triggerpin[direction], LOW);
  int t = pulseIn (echopin[direction], HIGH);
  return t * 0.034/2;
}

// Function to make all LED's LOW except RED one's.
void resetAll (){
  for (int i = 0; i < 4; i++){
        for (int dir = 0; dir < 4; dir++) {
            digitalWrite (signalsPins[dir][RED], HIGH);
            digitalWrite (signalsPins[dir][YELLOW], LOW);
            digitalWrite (signalsPins[dir][GREEN], LOW);
        }
    }
}

void print (int[] distances){
  Serial.print ("\tUP: ");
  Serial.println (distances[UP]);
  Serial.print ("Left: ");
  Serial.print (distances[LEFT]);
  Serial.print (" \tRight: ");
  Serial.println (distances[RIGHT]);
  Serial.print ("Down: ");
  Serial.println (distances[DOWN]);
}
