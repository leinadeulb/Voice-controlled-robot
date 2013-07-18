Voice-controlled-robot
#if defined(ARDUINO) && ARDUINO >= 100
  #include "Arduino.h"
  #include "SoftwareSerial.h"
  #include <Servo.h>
  SoftwareSerial port(12,13);
#else // Arduino 0022 - use modified NewSoftSerial
  #include "WProgram.h"
  #include "NewSoftSerial.h"
  NewSoftSerial port(12,13);
#endif

#include "EasyVR.h"
Servo myservo;
Servo myservo1;
EasyVR easyvr(port);

//Groups and Commands
enum Groups
{
  GROUP_0  = 0,
  GROUP_1  = 1,
  //GROUP_2  = 2,
};

enum Group0 
{
  G0_ARDUINO = 0,
};

enum Group1 
{
  G1_FORWARD = 0,
  G1_BACKWARD = 1,
  G1_RIGHT = 2,
  G1_LEFT = 3,
  G1_STOP = 4,
};

/*enum Group2 
{
  G2_FORWARD = 0,
  G2_BACKWARD = 1,
  G2_RIGHT = 2,
  G2_LEFT = 3,
  G2_STOP = 4,
};
*/

EasyVRBridge bridge;

int8_t group, idx;

void setup()
{
  
   myservo.attach(10);  // attaches the servo on pin 3 to the servo object 

  myservo.write(90);  // set servo to mid-point
  
  
  myservo1.attach(11);
  
  myservo1.write(90);
  
  
  
  
  // bridge mode?
  if (bridge.check())
  {
    cli();
    bridge.loop(0, 1, 12, 13);
  }
  // run normally
  Serial.begin(9600);
  port.begin(9600);

  if (!easyvr.detect())
  {
    Serial.println("EasyVR not detected!");
    for (;;);
  }

  easyvr.setPinOutput(EasyVR::IO1, LOW);
  Serial.println("EasyVR detected!");
  easyvr.setTimeout(5);
  easyvr.setLanguage(0);

  group = EasyVR::TRIGGER; //<-- start group (customize)
  
   pinMode(10, OUTPUT); 
  //digitalWrite(10, LOW);    // set the LED off
  
  pinMode(11, OUTPUT); 
  //digitalWrite(11, LOW);    // set the LED off
}

void action();

void loop()
{
  easyvr.setPinOutput(EasyVR::IO1, HIGH); // LED on (listening)

  Serial.print("Say a command in Group ");
  Serial.println(group);
  easyvr.recognizeCommand(group);

  do
  {
    // can do some processing while waiting for a spoken command
  }
  while (!easyvr.hasFinished());
  
  easyvr.setPinOutput(EasyVR::IO1, LOW); // LED off

  idx = easyvr.getWord();
  if (idx >= 0)
  {
    // built-in trigger (ROBOT)
    // group = GROUP_X; <-- jump to another group X
    return;
  }
  idx = easyvr.getCommand();
  if (idx >= 0)
  {
    // print debug message
    uint8_t train = 0;
    char name[32];
    Serial.print("Command: ");
    Serial.print(idx);
    if (easyvr.dumpCommand(group, idx, name, train))
    {
      Serial.print(" = ");
      Serial.println(name);
    }
    else
      Serial.println();
    easyvr.playSound(0, EasyVR::VOL_FULL);
    // perform some action
    action();
  }
  else // errors or timeout
  {
    if (easyvr.isTimeout())
      Serial.println("Timed out, try again...");
    int16_t err = easyvr.getError();
    if (err >= 0)
    {
      Serial.print("Error ");
      Serial.println(err, HEX);
    }
  }
}

void action()
{
    switch (group)
    {
    case GROUP_0:
      switch (idx)
      {
      case G0_ARDUINO:
        // write your action code here
        group = GROUP_1;// <-- or jump to another group X for composite commands
        
        break;
      }
      break;
    case GROUP_1:
      switch (idx)
      {
      case G1_FORWARD:
        // write your action code here
        myservo.write(0);
        myservo1.write(180);
         group = GROUP_1;// <-- or jump to another group X for composite commands
       
         
        break;
      case G1_BACKWARD:
        myservo.write(180);
        myservo1.write(0);
         group = GROUP_1;// <-- or jump to another group X for composite commands
         break;
         
         case G1_RIGHT:
        myservo.write(0);
        myservo1.write(0);
          delay(1000);
        myservo.write(90);
        myservo1.write(90);
         group = GROUP_1;// <-- or jump to another group X for composite commands
        break;
      case G1_LEFT:
        myservo.write(180);
        myservo1.write(180);
          delay (1000);
        myservo.write(90);
        myservo1.write(90);
         group = GROUP_1;// <-- or jump to another group X for composite commands
        break;
      case G1_STOP:
        myservo.write(90);
        myservo1.write(90);
         group = GROUP_1;// <-- or jump to another group X for composite commands
        break;
      
         
        break;
      }
      break;
    
    /*case GROUP_2:
      switch (idx)
      {
      case G2_FORWARD:
        myservo.write(0);
        myservo1.write(180);
        
         group = GROUP_0;// <-- or jump to another group X for composite commands
        break;
      case G2_BACKWARD:
        myservo.write(180);
        myservo1.write(0);
         group = GROUP_0;// <-- or jump to another group X for composite commands
        break;
      case G2_RIGHT:
        // write your action code here
        // group = GROUP_X; <-- or jump to another group X for composite commands
        break;
      case G2_LEFT:
        // write your action code here
        // group = GROUP_X; <-- or jump to another group X for composite commands
        break;
      case G2_STOP:
        // write your action code here
        // group = GROUP_X; <-- or jump to another group X for composite commands
        break;
      }
      break;
      */
    }
}

