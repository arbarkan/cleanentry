![Image](/media/clearfulllogo.png)

Clean Entry is a realization of a contactless door entry product developed over a semester in the Human-Centered Design course at UC, Berkeley. In this page, I cover highlights of the design process as well as an overview of the prototype's construction and functionality.

![Image](/media/operation.gif)

## The Research and Problem Opportunity Gap (POG)

What began as an idea to enable restroom doors with contactless entry expanded in scope significantly after conducting initial user research. Our findings after performing both virtual user interviews and in-person subject observation yielded some important insights that reflected current trends in society. For example, we discovered that most individuals are primarily concerned with the lack of cleanliness of surfaces on doors, which were largely attributed to heighten sanitation concerns due to COVID-19. Somewhat expectedly, all of our interviewees expressed preferences for doors that were automated due to increased convenience of operation and limited reliance on physical contact.

>_I always kick open public doors or use my shoulder because I'm afraid other people's germs are on them._
> - Undergraduate Student

>_I haven't used a non-automatic door since the pandemic started._
> - Young Professional

It became clear that the problem of sanitation and convenience in entryways extended beyond just the public restroom experience. After analyzing our notes from interviews and  the various social, economic, and technological factors associated to the design space, we arrived at a product opportunity gap that would guide our journey to a useful, useable, and desirable solution.

### Avoid spreading germs and viruses through door handles in public, especially for smaller spaces such as offices and bathroom entries.

## The Clean Entry Concept

In the process of generating design concepts, we tried to focus the scope of our solution to be exclusively door augmentations as opposed to full door replacements. In this way, it would be possible to accommodate a wider range of potential consumers in terms of accessibility. Other product requirements that arose from our design process were inspired by several of our prioritized value opportunity attributes. Specifically, we determined that the solution should reduce the public concern surrounding exposure to germs and viruses. It should also be useable by differently abled individuals without additional assistance. These requirements and others resulted in some innovative ideas for addressing our pproblem space. However, one concept in particular stood out from the others. We called this concept the door kit.

![Image](/media/doorkit.PNG)
#### Conceptual drawings for the door kit that involves placeable proximity sensor modules and a door actuation mechanism.

The concept of the door kit is to provide users with a set of components that allows them to easily implement custom door automation for their swivel doors. The kit leverages proximity sensing and a door actuator to provide contactless door automation. In the above conceptual drawing, the proximity sensor is integrated into a hand sanitizer in order to encourage its usage and thus promote sanitary habits in public. The sensors can be placed anywhere that might suit the customer's needs. For exxample, a sensor could be placed lower to the ground to better detect individuals who use wheelchairs as well.

## Low-Fidelity Prototype Testing

There were many aspects of our concept that we still needed to learn more about before we were ready to commit to a final design. In order to better understand the social and intangible aspects of our concept, we decided to perform some testing with a low-fidelity prototype. Using cardboard, we created some basic mock-up components that would allow us to simulate interacting with a door that has been retrofitted with our door kit. We sought to learn how user's responded to the option of opening the door using a multitude of automated contactless options such as using the hand sanitizer or just the lone proximity sensor versus the standard manual mode of swivel door operation. The images below show examples of how subjects interacted with our low-fidelity prototype.

![Image](/media/lofi_prox.jpg)
![Image](/media/lofi_sanitizer.JPG)
#### Low-fidelity prototype images showing two options for sensor placement and usage by subjects.

We learned from this low-fidelity prototyping that individuals appreciated the convenience of the automated option and that it would encourage them to be less concerned about the transmission of germs or viruses. In terms of constructive feedback, some subjects felt afraid that the automated system might be dangerous. It will be important to implement some safety precautions in the development of a final product. Additionally, there was some concern that it might not be immediately intuitive without some instruction. To address this, we would hope to include some form of large sign to indicate that the door is proximity activated in one way or another.

## The Final Prototype Design

Our final design for the prototype door kit includes the following components:
* Set of wireless proximity sensor modules
* Door actuation mechanism (scale model)
* Latch disengager
The image below shows a functional block diagram explaining the electromechanical components of the system.

[Image](/media/diagram.PNG)

Each component is installable by the end user and the door kit results in a fully automated, proximity-activated swivel door for a small public space. The only component in the above list not completely explained previously is the latch disengager, which is used to prevent the standard swivel door from engaging its bolt in the frame of the door. This allows the door actuation mechanism to control the opening and closing of the door independent of the original door latch.

To realize our final design, we decided to build both a proximity sensor module and a scale model of the door actuation mechanism. First, I will describe the proximity sensor module design. The proximity sensing is accomplished using an [infrared (IR) sensor from Pololu](https://www.pololu.com/product/1134) and transmits a digital signal when activated. This signal is encoded using a dedicated HT12E encoder IC. The encoded message is transmitted over 433MHz radio frequency (RF) using the [Superheterodyne 433MHz RF transmitter/receiver pair](https://www.amazon.com/RioRand-Superheterodyne-transmitter-receiver-3400/dp/B00HEDRHG6). The entire circuit is placed in a 3D-printed case, with the IR sensor centrally located and facing outward. Images of the open and closed sensor module can be seen below.

![Image](/media/module_open.jpg)
![Image](/media/module_closed.jpg)

Next, I will describe the door actuation mechanism. This part of the prototype is designed to be a scale model of our intended product due to budget and time constraints. The actuator is driven by a small servomotor connected to a 3d-printed 2-bar linkage. This linkage is what imparts force on the door in order to open and close it in its frame. The RF receiver component from the RF link pair is included in the door actuation mechanism. An HT12D is used to decode the signal from the RF receiver and send a digital signal to a [Huzzah32 Feather microcontroller](https://learn.adafruit.com/adafruit-huzzah32-esp32-feather), which controls the logic of the system.

![Image](/media/door.jpg)

Upon receiving a digital signal from the decoder IC, a hardware interrupt is triggered on the microcontroller, and the system enters its opening and closing routine. The routine sends a PWM signal (typical for servo control at 50Hz with approximately 1-2ms pulse length) to control the servo motor between its full range of motion for the swivel door, pausing briefly at full open to allow time for entry. The code used for controlling the door actuation mechanism can be seen below.

```
// Basic script for controlling door mechanism via wireless proximity sensor module
//

#define SERVOMIN    2000  // This is the 'minimum' pulse length for range of motion
#define SERVOMAX    4000  // This is the 'maximum' pulse length for range of motion
#define LED_PIN     13    // LED read out for debug and indication
#define SERVO_PIN   27    // Servo PWM pin
#define RF_PIN      15    // Digital input from RF decoder

#define CHANNEL_0   0     // Use channel 0
#define FREQ_SERVO  50    // Servo PWM frequency ~50Hz  
#define RESOLUTION  16    // Use 16-bit resolution for PWM

bool door_opening = false;  // State of door operation (normally closed and false)
bool sensed = false;        // State of sensor module
uint16_t val = 0;           // Store current position

// Define hardware interrupt handler for RF comm
void IRAM_ATTR detectOpen() {
  if (~door_opening) {
    sensed = true;
  }
}

void setup() {
  // Begin setup routine

  Serial.begin(9600);

  // Assign RF_PIN as digital input
  pinMode(RF_PIN, INPUT);
  pinMode(RF_PIN, INPUT_PULLUP);
  
  // Setup PWM output
  ledcSetup(0, 50, 16);
  ledcAttachPin(SERVO_PIN, 0);

  // Setup hardware interrupt
  attachInterrupt(digitalPinToInterrupt(RF_PIN), detectOpen, CHANGE);

  delay(2000);
  
}

void loop() {

  if (sensed) {
    door_opening = true;
    Serial.println("Opening door!");
    for (uint16_t command = SERVOMIN; command <= SERVOMAX; ){
      ledcWrite(0, command);
      command = command + 5;
      Serial.println(command);
      delay(5);
      val = command;
    }
    delay(100);
    ledcWrite(0, 0);
    Serial.println("Door has been opened.");
    delay(3000);
    for (uint16_t command = val; command >= SERVOMIN; ){
      ledcWrite(0, command);
      command = command - 5;
      Serial.println(command);
      delay(5);
    }
    delay(100);
    ledcWrite(0, 0);
    Serial.println("Door has been closed.");
    sensed = false;
  }

  
}
```
