# TomatoCube Introduction to MIDI Programming with MicroPython on the BBC® Micro:bit

## Overview

[**MIDI**](https://en.wikipedia.org/wiki/MIDI), which stands for **Musical Instrument Digital Interface**, is a standard for communication between electronic musical devices, computers, and other sound-related equipment. Think of it as a "language" that allows different devices to talk to each other to produce, control, and manipulate musical sounds. Using MIDI, we can make a simple device like the BBC Micro:bit play musical notes and sounds, or even control real musical instruments! How cool is that!

In this guide, we'll show you how to use [**MicroPython**](https://en.wikipedia.org/wiki/MicroPython)—a version of Python designed for small devices—to program the Micro:bit to communicate with a computer or musical instrument via MIDI. Make sure you know how to use MicroPython on your BBC Micro:bit before proceeding any further.

## Why MIDI?

- MIDI doesn’t carry actual sound, but rather instructions. This means MIDI data tells an instrument *what* to play (like a note or drum sound), *when* to play it, and *how long* to hold it.
- MIDI data is often used to create music by connecting a device (like a keyboard or computer) to speakers, instruments, or synthesizers.
- Don't re-invent the wheel as was often said, with the Micro:bit, we can stick to this standard & start learning how to create sounds and control musical instruments with code!

## Equipment Needed

To set up MIDI communication between the Micro:bit and a computer, here’s what you’ll need:

1. BBC Micro:Bit (Both v1 & v2 will perform equally well): This is the main microcontroller board we'll be using.

2. TomatoCube's Midi Instrument Expansion Board.

3. USB MIDI-IN/MIDI-OUT Cable: This cable connects the Micro:bit via the Midi instrument Expansion Board to a computer or MIDI-compatible instrument. This cable will work Auto-Magically out of the box on your Mac, iPad & even your Windows Machine.

4. MicroPython Environment: We’ll be using MicroPython to code on the Micro:bit, it would make your life so much easier when a Great editor is thrown into the mix. [Thonny](https://thonny.org) is highly recommended for its thigh integration into the Micro:Bit & Rapberry Pico Python's ecosystem, give it a try.

5. Piezo Input Module & Play-Doh: Step #7 & Step #8

   



## Getting Started

Before you start coding, ensure your Micro:bit is connected to your computer via USB and your USB MIDI-IN/MIDI-OUT cable is ready (We will plug it in in Step #3).

### Step 1: Setting Up the MicroPython Environment

1. Go to the MicroPython Editor - Thonny

   1. Connect your Micro:bit to the editor by first installing the MicroPython interpreter to your Micro:Bit
      `Tools > Options > Interpreter > select "MicroPython (BBC micro:bit)" > "Install or Update MicroPython"`
      <img src=".\assets\Thonny_microbitInterpreter.png" alt="Thonny_microbit" style="zoom: 60%;"  />
   1. Select your Micro:bit board version, select the latest interpreter version, then hit "Install"      
      <img src=".\assets\Thonny_microbitInterpreter_Install.png" alt="Thonny_microbit_Install" style="zoom: 60%;"  />

2. Choose the correct **"Port"** under the Port configuration drop-down list.<img src=".\assets\Thonny_microbitPort.png" alt="Thonny_microbit_Port" style="zoom: 60%;"  />

3. You’re now ready to start coding in MicroPython!



### Step 2: Hello World in MicroPython

In MicroPython, we’ll use simple code that pays homage to the classic of coding tradition - Hello World. The code will make the Micro:Bit display a Greetings + Smiley face then we will sprinkle on a bit of magic on top, roll a dice when **Button A** is pressed.

```python
from microbit import *
import random

faces = [Image('00000:00000:00900:00000:00000:'),
        Image('00009:00000:00000:00000:90000:'),
        Image('00009:00000:00900:00000:90000:'),
        Image('90009:00000:00000:00000:90009:'),
        Image('90009:00000:00900:00000:90009:'),
        Image('90009:00000:90009:00000:90009:')]
   
def RandomImages(n, delay):
    for i in range(0,n):
            display.show(random.choice(faces))
            sleep(delay)
            display.clear()
            sleep(delay)

display.scroll('Hello!')
display.show(Image.HAPPY)
while True:
    if button_a.was_pressed():
        RandomImages(10, 50)
        rolled = random.choice(faces)
        display.show(rolled)
        
```



### Step 3: Writing Your First MIDI Sender Code

Back in our MicroPython Editor, we’ll now use another simple code to make the Micro:bit send MIDI messages. Here’s an example to get us started with. The code uses the Micro:Bit's buttons to start and stop **"Pressing"** a note. Pressing **Button A** starts playing the note, and **Button B** stops it.

```python
from microbit import *
import music

# Define MIDI settings
midi_channel = 1  # MIDI channels range from 1-16
midi_instrument = 10	# Instrument type range from 1-20
note_on = 0x90  # MIDI command for "note on" (start playing a note)
note_off = 0x80  # MIDI command for "note off" (stop playing a note)
midi_channel_prog = 0xC0  # MIDI command for changeing the channel Instruments range from 1-20

# Choose a note (e.g., Middle C)
note = 0x60  # MIDI number for Middle C

# Prevent micro:bit being stuck in MIDI (UART) Mode
if button_a.is_pressed():
    while True:
        display.show(Image.SKULL)
        # Do Nothing Here

# Initialize the UART Port for Midi Sending
uart.init(baudrate=31250, bits=8, parity=None, stop=1, tx=pin15)
uart.write(bytes([midi_channel_prog | (midi_channel - 1), midi_instrument]))

while True:
    if button_a.is_pressed():
        # Send "note on" command for Middle C
        uart.write(bytes([note_on | (midi_channel - 1), note, 127]))  # 127 = max volume
        display.show(Image.HAPPY)
    elif button_b.is_pressed():
        # Send "note off" command for Middle C
        uart.write(bytes([note_off | (midi_channel - 1), note, 0]))  # 0 = min volume
        display.show(Image.SAD)
    else:
        display.clear()
    sleep(10)
```

#### Further Code Explanation

- **MIDI Commands**: MIDI uses "commands" to communicate. For example:
- `0x90` tells the device to turn a note on (start playing a sound).
  - `0x80` tells it to turn a note off (stop playing the sound).
- **Note Selection**: `note = 60` represents the note "Middle C." You can change this number to play different notes. Refer to **Appendix A** below for more.
- **Volume Control**: `127` and `0` are maximum and minimum volume values, respectively.
- **MicroPython REPL Escape sequence**: Since MIDI communication on the Micro:Bit uses the UART component to send data, redirecting UART to **pin15** for MIDI means losing access to the REPL console. This could make it challenging to exit your current Python code or load new code. To avoid being "stuck" in MIDI mode, code has been added to allow easy access to the REPL console. Simply press the **RESET **button on the back of the Micro, while holding **Button A** on the front to switch back to the REPL console. A **Skull** would show up on the LED display.



### Step 4: Connecting the Micro:Bit to Your Midi capable Computer/Instrument

1. USB MIDI-IN/MIDI-OUT Cable: Plug the USB end of the USB MIDI-IN/MIDI-OUT cable into your computer’s USB port and the other end, one of the two 5-pin MIDI DIN connectors labeled "RX" into the Transmitter side of the Midi instrument Expansion Board's. This will, in turn, use a UART (Universal Asynchronous Receiver-Transmitter) pins from the Micro:Bit.

2. Open a MIDI Application: You can use a variety of MIDI software, such as the free Shared Piano on Google's [Chrome Music Lab](https://musiclab.chromeexperiments.com) (You must use a browser that supports WebMidi & USB support, e.g. Google Chrome), SonicPi or GarageBand (Mac & iPad only), to receive the MIDI signal. 
> **_Extra Note:_**  Sonic Pi users can use the following script to accept external Midi Input...
> <img src=".\assets\sonicPi_MidiInput.png" alt="sonicPi_MidiInput" style="zoom: 80%;"  />
>
> ```Ruby
> live_loop :midi_piano do
> note, velocity = sync "/midi:usb2.0-midi_0:1/note_on"
> synth :piano, note: note
> end
> ```

3. **Test Your Program**: Press **Button A** to send the "note on" signal, and check if the connected computer or instrument plays the sound. Press **Button B** to stop it.



### Step 5: Experiment with MIDI Notes

To make your project more interesting, try experimenting with:

- **Different Notes**: Change the value of the `note` to a different number (e.g., 64 for E4 or 67 for G4).
- **Timing**: Use the `sleep()` function to create rhythm patterns.

> **_Extra Note:_**  I have added `midi_channel_prog` in the above code for the curious ones that wish to change the midi channel of choice to another instrument variant. This functionality is highly dependent on the software application you are running.



### Step 6: Building a simple Drum Set - Midi Percussion

In **General MIDI**, **Channel 10** is reserved for percussion instruments. Unlike other instruments that play unique notes, percussion instruments on Channel 10 each have specific MIDI note values for different drum sounds (e.g., bass drum, snare drum, hi-hat). This setup allows you to create rhythms and sound effects by triggering various "notes" on this channel.

For this example, we’ll use **Button A** to play a **bass drum** and **Button B** to play a **snare drum**. Please refer to Appendix B for the Notes if you wish to use a different percussion instrument. For our case, we’ll use **35 (0x23)** for the **bass drum** and **38 (0x26)** for the **snare drum**.

```python
from microbit import *
import music

# Define MIDI channel and percussion commands
midi_channel = 10  # General MIDI percussion channel
note_on = 0x90     # MIDI "note on" command
note_off = 0x80    # MIDI "note off" command

# Define percussion notes for bass drum and snare drum
bass_drum = 35     # MIDI note for bass drum
snare_drum = 38    # MIDI note for snare drum

# Prevent micro:bit being stuck in MIDI (UART) Mode
if button_a.is_pressed():
    while True:
        display.show(Image.SKULL)
        # Do Nothing Here

# Initialize the UART Port for Midi Sending
uart.init(baudrate=31250, bits=8, parity=None, stop=1, tx=pin15)

# Flags to track button states
button_a_pressed = False
button_b_pressed = False

while True:
    # Check Button A for bass drum
    if button_a.is_pressed():
        if not button_a_pressed:
            # Send "note on" for bass drum when button is first pressed
            uart.write(bytes([note_on | (midi_channel - 1), bass_drum, 127]))
            display.show("B")
            button_a_pressed = True
    else:
        if button_a_pressed:
            # Send "note off" when button is released
            uart.write(bytes([note_off | (midi_channel - 1), bass_drum, 0]))
            display.clear()
            button_a_pressed = False

    # Check Button B for snare drum
    if button_b.is_pressed():		# To check: What will happen when changed to was_pressed() from is_pressed()
        if not button_b_pressed:
            # Send "note on" for snare drum when button is first pressed
            uart.write(bytes([note_on | (midi_channel - 1), snare_drum, 127]))
            display.show("S")
            button_b_pressed = True
    else:
        if button_b_pressed:
            # Send "note off" when button is released
            uart.write(bytes([note_off | (midi_channel - 1), snare_drum, 0]))
            display.clear()
            button_b_pressed = False

    sleep(100)

```

#### Further Code Explanation

- **Button Flags**: `button_a_pressed` and `button_b_pressed` are used to track the button states.
- **Note On** and **Note Off**:
  - When a button is first pressed (`is_pressed()` returns `True` and the flag is `False`), it sends a **Note On** message and sets the flag to `True`.
  - When the button is released (`is_pressed()` returns `False` and the flag is `True`), it sends a **Note Off** message and resets the flag to `False`.

Using the Flag & Button State approach ensures that each button press triggers a single MIDI note, with the note stopping as soon as the button is released, mimicking more closely to how a real instrument should behave.



### Step 7: A better input mechanism - Alternative Sensor input, the Piezo Element

A **knock sensor** is a sensor that detects vibrations or knocks. In the heart of the knock sensor module is the **piezo element**—the core component of many buzzers or any musical greeting cards you have received during the holiday season or your birthday. Applying an electric signal to a piezoelectric material changes its size, when housed in a suitable enclosure, the piezo element would warp, pushing air molecules, thus creating sound. But when Piezo elements are subjected to physical vibrations/compression, the reverse happens; it generates small electrical signals, which we can measure using our Micro:Bit to detect knocks/vibration. 

When tapped, the piezo element outputs a voltage spike that can be detected on one of the Micro:Bit's GPIO pins. We’ll use this setup to detect knock events and respond by showing an indication on the Micro:Bit LED Matrix display.

The following MicroPython code will read the piezo element’s output on pin `P1`. When a knock is detected, it briefly displays an indicator on the Micro:Bit’s LED display. A flag is used to track knock states to ensure we capture both press (knock) and release events effectively.

```python

from microbit import *

# Threshold for detecting a knock; adjust based on the sensitivity of your sensor
knock_threshold = 500

# Flag to track knock state
knock_detected = False

while True:
    # Read the analog signal from the piezo element
    knock_value = pin1.read_analog()

    # Detect a knock if the value exceeds the threshold
    if knock_value > knock_threshold:
        if not knock_detected:
            # Knock detected, display an indicator
            display.show(Image.HAPPY)
            knock_detected = True	#To Check: Not Sure if Flag is even needed!!!
    else:
        if knock_detected:
            # Knock has ended, clear display and reset flag
            display.clear()
            knock_detected = False

    # Small delay to prevent flickering
    sleep(10)
```

------

#### Further Code Explanation

- **Threshold Setting**: The `knock_threshold` variable defines the sensitivity. If the signal from the piezo element exceeds this value, a knock is registered. You might need to adjust this value based on the piezo element's sensitivity.
- **Flag Tracking**: `knock_detected` tracks whether a knock is currently detected. This ensures that a knock event only registers once until the piezo element signal returns below the threshold.
- Display Feedback: When a knock is detected, the Micro:Bit briefly displays ":)" on the screen. The display clears when the knock has ended.

#### Enhancing our Drum Set

Now that we have a much more accurate/realistic reproduction of a drum set, could we perhaps combine the last 2 Python codes together?

## Appendix A: MIDI Note Table

Below is a table of common MIDI notes with their hexadecimal values. 
These values can be used to play different musical notes on MIDI-compatible devices & software applications.

| Musical Note | Hex Value | Musical Note | Hex Value | Musical Note | Hex Value |
| ------------ | --------- | ------------ | --------- | ------------ | --------- |
| C(-1)        | 00        | C3           | 30        | C6           | 54        |
| C#(-1)       | 01        | C#3          | 31        | C#6          | 55        |
| D(-1)        | 02        | D3           | 32        | D6           | 56        |
| D#(-1)       | 03        | D#3          | 33        | D#6          | 57        |
| E(-1)        | 04        | E3           | 34        | E6           | 58        |
| F(-1)        | 05        | F3           | 35        | F6           | 59        |
| F#(-1)       | 06        | F#3          | 36        | F#6          | 5A        |
| G(-1)        | 07        | G3           | 37        | G6           | 5B        |
| G#(-1)       | 08        | G#3          | 38        | G#6          | 5C        |
| A(-1)        | 09        | A3           | 39        | A6           | 5D        |
| A#(-1)       | 0A        | A#3          | 3A        | A#6          | 5E        |
| B(-1)        | 0B        | B3           | 3B        | B6           | 5F        |
| C0           | 0C        | C4           | 3C        | **C7**       | **60**    |
| C#0          | 0D        | C#4          | 3D        | C#7          | 61        |
| D0           | 0E        | D4           | 3E        | D7           | 62        |
| D#0          | 0F        | D#4          | 3F        | D#7          | 63        |
| E0           | 10        | E4           | 40        | E7           | 64        |
| F0           | 11        | F4           | 41        | F7           | 65        |
| F#0          | 12        | F#4          | 42        | F#7          | 66        |
| G0           | 13        | G4           | 43        | G7           | 67        |
| G#0          | 14        | G#4          | 44        | G#7          | 68        |
| A0           | 15        | A4           | 45        | A7           | 69        |
| A#0          | 16        | A#4          | 46        | A#7          | 6A        |
| B0           | 17        | B4           | 47        | B7           | 6B        |
| C1           | 18        | C5           | 48        | C8           | 6C        |
| C#1          | 19        | C#5          | 49        | C#8          | 6D        |
| D1           | 1A        | D5           | 4A        | D8           | 6E        |
| D#1          | 1B        | D#5          | 4B        | D#8          | 6F        |
| E1           | 1C        | E5           | 4C        | E8           | 70        |
| F1           | 1D        | F5           | 4D        | F8           | 71        |
| F#1          | 1E        | F#5          | 4E        | F#8          | 72        |
| G1           | 1F        | G5           | 4F        | G8           | 73        |
| G#1          | 20        | G#5          | 50        | G#8          | 74        |
| A1           | 21        | A5           | 51        | A8           | 75        |
| A#1          | 22        | A#5          | 52        | A#8          | 76        |
| B1           | 23        | B5           | 53        | B8           | 77        |
| C2           | 24        | C6           | 54        | C9           | 78        |
| C#2          | 25        | C#6          | 55        | C#9          | 79        |
| D2           | 26        | D6           | 56        | D9           | 7A        |
| D#2          | 27        | D#6          | 57        | D#9          | 7B        |
| E2           | 28        | E6           | 58        | E9           | 7C        |
| F2           | 29        | F6           | 59        | F9           | 7D        |
| F#2          | 2A        | F#6          | 5A        | F#9          | 7E        |
| G2           | 2B        | G6           | 5B        | G9           | 7F        |
| G#2          | 2C        | G#6          | 5C        |              |           |
| A2           | 2D        | A6           | 5D        |              |           |
| A#2          | 2E        | A#6          | 5E        |              |           |
| B2           | 2F        | B6           | 5F        |              |           |


## Appendix B: MIDI Percussion Note Reference

Below is a quick reference of some common MIDI percussion notes on Channel 10:

| Percussion Instrument | MIDI Note Value | Hex Value |
| --------------------- | --------------- | --------- |
| Bass Drum 1           | 35              | 0x23      |
| Snare Drum            | 38              | 0x26      |
| Closed Hi-Hat         | 42              | 0x2A      |
| Open Hi-Hat           | 46              | 0x2E      |
| Crash Cymbal          | 49              | 0x31      |
| Ride Cymbal           | 51              | 0x33      |
