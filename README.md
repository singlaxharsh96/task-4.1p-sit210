# task-4.1p-sit210

#include <Wire.h>
#include <BH1750.h>


// PIN DEFINITIONS


const int PIR_PIN = 2;
const int BUTTON_PIN = 3;

const int LED1_PIN = 5;
const int LED2_PIN = 6;


// BH1750


BH1750 lightMeter;


// INTERRUPT FLAGS


volatile bool motionDetected = false;
volatile bool buttonPressed = false;



// PIR INTERRUPT


void motionISR()
{
  motionDetected = true;
}



// BUTTON INTERRUPT


void buttonISR()
{
  buttonPressed = true;
}



// SETUP


void setup()
{
  Serial.begin(9600);

  // PIR
  pinMode(PIR_PIN, INPUT);

  // Button
  pinMode(BUTTON_PIN, INPUT_PULLUP);

  // LEDs
  pinMode(LED1_PIN, OUTPUT);
  pinMode(LED2_PIN, OUTPUT);

  // Start with lights OFF
  digitalWrite(LED1_PIN, LOW);
  digitalWrite(LED2_PIN, LOW);

  // Start I2C
  Wire.begin();

  // Start BH1750
  if (lightMeter.begin(BH1750::CONTINUOUS_HIGH_RES_MODE))
  {
    Serial.println("BH1750 initialised.");
  }
  else
  {
    Serial.println("BH1750 not detected.");
  }

  // PIR interrupt
  attachInterrupt(
    digitalPinToInterrupt(PIR_PIN),
    motionISR,
    RISING
  );

  // Button interrupt
  attachInterrupt(
    digitalPinToInterrupt(BUTTON_PIN),
    buttonISR,
    FALLING
  );

  Serial.println("Task 4.1P Interrupt System Started.");
  Serial.println("Waiting for motion or button press...");
}



// MAIN LOOP


void loop()
{
  // Read light level
  float lux = lightMeter.readLightLevel();

  Serial.print("Light level: ");
  Serial.print(lux);
  Serial.println(" lux");


  
  // MOTION DETECTED
  

  if (motionDetected)
  {
    noInterrupts();
    motionDetected = false;
    interrupts();

    Serial.println("Motion interrupt detected.");

    // Motion turns lights ON permanently
    turnLightsOn();

    Serial.println("Motion detected - lights ON.");
  }


  
  // BUTTON PRESSED
  

  if (buttonPressed)
  {
    noInterrupts();
    buttonPressed = false;
    interrupts();

    Serial.println("Button interrupt detected.");

    // Toggle the lights
    if (digitalRead(LED1_PIN) == LOW)
    {
      turnLightsOn();
      Serial.println("Button pressed - lights ON.");
    }
    else
    {
      turnLightsOff();
      Serial.println("Button pressed - lights OFF.");
    }
  }

  delay(200);
}



// TURN LIGHTS ON


void turnLightsOn()
{
  digitalWrite(LED1_PIN, HIGH);
  digitalWrite(LED2_PIN, HIGH);
}



// TURN LIGHTS OFF


void turnLightsOff()
{
  digitalWrite(LED1_PIN, LOW);
  digitalWrite(LED2_PIN, LOW);
}
