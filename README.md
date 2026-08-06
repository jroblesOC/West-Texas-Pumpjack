West-Texas-Pumpjack
Desktop Pumpjack Powered by TT Motor and ATTiny85 MCU

ATtiny85 Mini Pumpjack

A compact, 3D-printed pumpjack demonstration controlled by an **ATtiny85** microcontroller.  
The model uses a **TT geared DC motor**, an **IRFZ44N N-channel MOSFET**, a potentiometer for speed control, a push button for start/stop control, and an LED to indicate when the motor is commanded to run.

The entire control circuit is assembled on a **170-point mini breadboard**.

## Project Purpose

This project demonstrates several basic industrial-control and embedded-system concepts:

- Starting and stopping a DC motor with a push button
- Using a MOSFET as an electronic motor switch
- Controlling motor speed with PWM
- Reading an analog potentiometer
- Providing an operating-status indication with an LED
- Using a small microcontroller in a mechanical demonstration model

## Main Features

- Push-button start/stop toggle
- Adjustable pumpjack speed
- LED motor-running indication
- Compact ATtiny85-based controller
- 3D-printed pumpjack mechanism
- Breadboard-mounted control circuit

## How It Works

1. The push button is connected between the ATtiny85 input and ground.
2. The ATtiny85 internal pull-up resistor keeps the input HIGH when the button is not pressed.
3. Pressing the button changes the input to LOW and toggles the `motorRunning` flag.
4. When the motor is enabled, the ATtiny85 reads the potentiometer.
5. The potentiometer reading is converted to a PWM value from 0 to 255.
6. The PWM signal drives the MOSFET gate.
7. The MOSFET controls current through the TT motor.
8. The LED turns on whenever the motor is commanded to run.

## Pin Assignments

| Function | ATtiny85 Arduino Pin |
|---|---:|
| MOSFET / motor PWM output | `0` |
| Potentiometer analog input | `A2` |
| Push button input | `3` |
| Running-indicator LED | `1` |

> ATtiny85 pin labels can vary by Arduino core and programmer configuration. Verify the physical pin mapping for the core selected in the Arduino IDE.

## Required Wiring

### TT Motor and MOSFET

- ATtiny85 pin `0` to the MOSFET gate
- MOSFET source to ground
- MOSFET drain to the negative motor terminal
- Positive motor terminal to the motor supply positive
- ATtiny85 ground and motor-supply ground connected together

### Potentiometer

- One outer terminal to VCC
- Other outer terminal to ground
- Center wiper to `A2`

### Push Button

- One side to ATtiny85 pin `3`
- Other side to ground
- The sketch uses `INPUT_PULLUP`, so an external pull-up resistor is not required

### LED

- ATtiny85 pin `1` to the LED through a current-limiting resistor
- LED cathode to ground

## Recommended Protection Components

A DC motor is an inductive load. Install a **flyback diode** directly across the motor terminals:

- Diode cathode to motor positive
- Diode anode to the MOSFET drain / motor negative

A gate resistor and gate pull-down resistor are also recommended:

- 100–220 ohm resistor between the ATtiny85 PWM pin and MOSFET gate
- 10 kΩ resistor between the MOSFET gate and ground

## Important MOSFET Note

The **IRFZ44N is not specified as a logic-level MOSFET**. It may operate with a small TT motor when driven from a 5 V ATtiny85 output, but it may not turn fully on and can run warmer than expected.

A better logic-level substitute is:

- IRLZ44N
- FQP30N06L
- Another logic-level N-channel MOSFET rated for the motor current

## Software Correction

In the supplied sketch, the PWM command is accidentally included after a `//` comment:

```cpp
int motSpeed = map(potValue, 0, 1023, 0, 255); // torque starts at 150       analogWrite(motPin, motSpeed);
```

Because everything after `//` is ignored, the motor does not receive the PWM command while `motorRunning` is true.

Use:

```cpp
int motSpeed = map(potValue, 0, 1023, 0, 255);
analogWrite(motPin, motSpeed);
```

For more reliable starting torque, use a minimum PWM value:

```cpp
int motSpeed = map(potValue, 0, 1023, 150, 255);
analogWrite(motPin, motSpeed);
```

## Corrected Sketch

```cpp
#include <avr/wdt.h>

// Motor-control PWM pin
const int motPin = 0;

// Potentiometer analog pin
const int potPin = A2;

// Push-button pin
const int buttonPin = 3;

// Motor-running indicator LED
const int ledPin = 1;

bool motorRunning = false;

void setup() {
  // Disable watchdog timer after reset
  MCUSR &= ~(1 << WDRF);
  WDTCR |= (1 << WDCE) | (1 << WDE);
  WDTCR = 0x00;

  pinMode(motPin, OUTPUT);
  pinMode(buttonPin, INPUT_PULLUP);
  pinMode(ledPin, OUTPUT);

  analogWrite(motPin, 0);
  digitalWrite(ledPin, LOW);
}

void loop() {
  // LOW means the button is pressed
  if (digitalRead(buttonPin) == LOW) {
    delay(200);  // Simple debounce
    motorRunning = !motorRunning;

    // Wait for button release
    while (digitalRead(buttonPin) == LOW) {
      delay(10);
    }
  }

  if (motorRunning) {
    digitalWrite(ledPin, HIGH);

    int potValue = analogRead(potPin);
    int motSpeed = map(potValue, 0, 1023, 150, 255);

    analogWrite(motPin, motSpeed);
  } else {
    analogWrite(motPin, 0);
    digitalWrite(ledPin, LOW);
  }
}
```

## Programming Requirements

- Arduino IDE
- ATtiny85 board support package
- Compatible ATtiny85 programmer or development board
- Correct clock and brownout settings for the selected ATtiny85 configuration

## Power Considerations

- Do not power the TT motor directly from an ATtiny85 output pin.
- Use the MOSFET to switch motor current.
- Use a power source that can supply the TT motor starting current.
- Connect the controller ground and motor-supply ground together.
- Add a capacitor near the motor supply if motor noise resets the ATtiny85.
- A 0.1 µF ceramic capacitor across the motor terminals can help reduce electrical noise.

## Repository Structure

```text
ATtiny85-Pumpjack/
├── README.md
├── PARTS_LIST.md
├── BOM.md
├── firmware/
│   └── ATtiny85_Pumpjack.ino
├── 3d-models/
│   ├── STL-files
│   └── source-files
├── images/
│   ├── completed-project
│   └── wiring
└── docs/
    └── schematic
```

## Possible Future Improvements

- Replace the IRFZ44N with a logic-level MOSFET
- Add non-blocking button debounce
- Add a soft-start ramp
- Add a minimum-speed adjustment
- Add motor-current measurement
- Add a small speed display
- Replace the breadboard with a soldered perfboard or PCB

## License

Add the license selected for the project, such as the MIT License for software and a suitable Creative Commons license for the 3D-print files.
ing README (1).md…]()
