//DetacherClass.ion
#include "RelayClass.h"
#include "LedClass.h"
#include "Detacher.h"
#include "Time.h"

DeviceManager deviceManager;
Relay relay;
TIME timeManager;  
int ledPin = 2;
int relayPin = 4;
LED led(2);  

void setup() {

}

void loop() {
    led.turnOnTimed(5);

    while (!led.checkForTimeUp()) {
        
    }

}

//LedClass.cpp
#include "esp32-hal.h"
#include "esp32-hal-gpio.h"
#include "LedClass.h"
#include "Arduino.h"
LED::LED(){
  
}

LED::LED(int _pin)
{
  attach(_pin);
}

void LED::attach(int _pin)
{
  pinMode(_pin, OUTPUT);
}


void LED::toggleOn(int _pin)
{
  digitalWrite(_pin, HIGH);
}

void LED::toggleOff(int _pin)
{
  digitalWrite(_pin, LOW);
}

void LED::Reverse(bool value, int _pin)
{
  digitalWrite(_pin, !value);
  state = !value;  
}

int LED::getState(bool state)
{
  if(state == 1)
  {
    return 1;
  }
  return 0;
}

void LED::turnOffGradually(int _pin)
{
  for (int i = 100; i > 0; i--) {
    analogWrite(_pin, i); 
    delay(1000);
  }
}
void LED::turnOnGradually(int _pin)
{
  for (int i = 0; i < 100; i++) {
    analogWrite(_pin, i); 
    delay(1000);
  }
}


void LED::toggleSwitch(int array[], int arraySize, int frequency, int _pin) {
  for (int i = 0; i < arraySize; i++) {
    if (array[i] == 1) 
    {
      toggleOff(_pin);
    } 
    else if(array[i] == 0)
    {
      toggleOn(_pin);
    }
    else {}
    delay(frequency); 
  }
}

void LED::turnOnTimed(float timeInSeconds) {
  startTime = millis();  
  interval = timeInSeconds * 1000;  
  turnOnGradually(2);
}

bool LED::checkForTimeUp() {
  if (millis() - startTime >= interval) {
      turnOffGradually(2);
      return true;  
  }
  return false;  
}


//LedClass.h
#include "soc/soc_caps.h"
#ifndef LedClass_h
#define LedClass_h

#include "Arduino.h"

class LED
{
  public:
    LED();
    LED(int _pin);
    void attach(int _pin);
    void toggleOn(int _pin);
    void toggleOff(int _pin);
    void Reverse(bool value, int _pin);
    int getState(bool state);
    void turnOffGradually(int _pin);
    void turnOnGradually(int _pin);
    void toggleSwitch(int array[], int arraySize, int frequency, int _pin);
    void turnOnTimed(float timeInSeconds);
    bool checkForTimeUp();
  private:  
   unsigned long startTime;
   unsigned long interval;
   bool state;
};
#endif 

//RelayClass.cpp
#include "esp32-hal.h"
#include "esp32-hal-gpio.h"
#include "RelayClass.h"
#include "Arduino.h"

Relay::Relay(){
  
}

Relay::Relay(int relayPin)
{
  attach(relayPin);
}

void Relay::attach(int relayPin)
{
  pinMode(relayPin, OUTPUT);
}


void Relay::relayToggleOn(int relayPin)
{
  digitalWrite(relayPin, HIGH);
}

void Relay::relayToggleOff(int relayPin)
{
  digitalWrite(relayPin, LOW);
}

void Relay::relayReverse(bool relayValue, int relayPin)
{
  digitalWrite(relayPin, !relayValue);
  relayValue = !relayValue;  
}

int Relay::relayGetState(bool relayState)
{
  if(relayState == 1)
  {
    return 1;
  }
  return 0;
}

void Relay::relayToggleSwitch(int relayArray[], int relayArraySize, int relayFrequency, int relayPin) 
{
  for (int i = 0; i < relayArraySize; i++) {
    if (relayArray[i] == 1) 
    {
      relayToggleOff(relayPin);
    } 
    else if(relayArray[i] == 0)
    {
      relayToggleOn(relayPin);
    }
    else {}
    delay(relayFrequency); 
  }
}

void Relay::relayTurnOff(int relayPin)
{
  for(int i = 0; i < 180; i++)
  {
    digitalWrite(relayPin, HIGH);
  }
}
void Relay::relayTurnOn(int relayPin)
{
  for(int i = 180; i < 0; i--)
  {
    digitalWrite(relayPin, LOW);
  }
}

/RelayClass.h
#include "soc/soc_caps.h"
#ifndef RelayClass_h
#define RelayClass_h

#include "Arduino.h"

class Relay
{
  public:
    bool relayState;
    Relay();
    Relay(int relayPin);
    void attach(int relayPin);
    void relayToggleOn(int relayPin);
    void relayToggleOff(int relayPin);
    void relayReverse(bool relayValue, int relayPin);
    int relayGetState(bool relayState);
    void relayTurnOff(int relayPin);
    void relayTurnOn(int relayPin);
    void relayToggleSwitch(int relayArray[], int relayArraySize, int relayFrequency, int relayPin);
};

#endif 

//Detacher.h


#include "LedClass.h"
#include "RelayClass.h"

#include "soc/soc_caps.h"
#ifndef Detacher_h
#define Detacher_h

#include "esp32-hal.h"
#include "esp32-hal-gpio.h"
#include "Detacher.h"
#include "Arduino.h"

class DeviceManager {
  public:
    DeviceManager() {}

    void detachAll(LED led, Relay relay, int ledPin, int relayPin) {
        detachLEDs(led, ledPin);
        detachRelays(relay, relayPin);
    }

  private:
    void detachLEDs(LED led, int ledPin) {
        led.attach(ledPin); 
        led.toggleOff(ledPin); 
    }

    void detachRelays(Relay relay, int relayPin) {
        relay.attach(relayPin); 
        relay.relayToggleOff(relayPin); 
    }
};
#endif

//TimeClass.cpp
#include "Time.h"
#include "esp32-hal.h"
#include "esp32-hal-gpio.h"
#include "Arduino.h"

float TIME::MillisecondsToSeconds(int milliseconds) {
    float second = float(milliseconds) / 1000;
    return second;
}

bool TIME::IsTimePassedFromInterval(float startTimeInSeconds, float intervalInSeconds, unsigned long currentMillis) {
    float endTimeSeconds = startTimeInSeconds + intervalInSeconds;
    float currentSeconds = MillisecondsToSeconds(currentMillis);

    if (currentSeconds < endTimeSeconds) {
        return true;
    } else {
        return false;
    }
}

//TimeClass.h
#include "soc/soc_caps.h"
#ifndef TIME_h
#define TIME_h
#include "Arduino.h"

class TIME {
  public:
    float MillisecondsToSeconds(int milliseconds);
    bool IsTimePassedFromInterval(float startTimeInSeconds, float intervalInSeconds, unsigned long currentMillis);
  private:
    int ledPin;
};

#endif
