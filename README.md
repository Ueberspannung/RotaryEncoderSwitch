# RotaryEncoderSwitch

##### Table of contents
- [description](#description)  
- [issues](#issues)  
- [HW considerations](#HW-considerations)  
    - [AVR](#AVR)  
    - [SAM](#SAM-D21)  
- [interface](#Interface) 
    - [button_init   ](#button_init   )     
    - [button_process](#button_process)  
    - [button_setLong](#button_setLong)  
    - [button_turns  ](#button_turns  )  
    - [button_down   ](#button_down   )  
    - [button_pressed](#button_pressed)  
    - [button_held   ](#button_held   )  


## description
KY-040 Rotary Encoder Switch driver  
this is a simple C code for using a rotary encoder switch such as the KY-040  

In the KY-040 form the encoder comes with integrated pull-ups.
This is not really necessary but nice to have.
One could use the internal pull-ups of the microcontroller   

## Issues
The SW is based on the Arduino Framework and designed to run on both AVR (e.g. Arduino Uno) and SAM devices (e.g. Arduino Zero).  
The switch produces a Gray Code signal on the clock and data pins which should be easily be decoded even without a HW unit.  
That's the theory which caused me quite a headache up to the point a use a scope to have a look at the signals.
Most KY-040 encoders are of quite poor quality. They seem to be implemented with wiper contancts. 
The wiper seems to loose connection when the knob is turned.
This leads to an almost undecodable noise. The counter jumps back and forth uncontrollable. 
But there has been an easy solution:  

Just add a small capacitor (~10nF) between the clock / data lines and ground.  

Every time the wiper contacts the base the capacitor is instantly discharged. 
During interruptions the capacitor is charged through the pull up resistor and 
instantly discharged when the conection is restored. When the capacitor is chosen 
correctly the voltage never rises to a high level until the wiper reaches the end of the contact.
The max. speed of the encoder is determined by the pull up resitor value and the capacitance.
I tried even 100 nF. This resulted in only very fast movements to be discarded.   

## HW considerations

### AVR
on Arduino Uno and similar platforms intterrupts are grouped by ports. 
Each port pin can generate an interrupt but there is only one interrupt vector for one port.
PB - PE are assigned to  PCINT0 - 3 vector.  
The arduino framework supports only one handler per interrupt vector.   
If all switch lines are connected to one port only one handler can be used to handle all signals.
If the switch lines are distributed ove several ports the SAM D21 approach can be used


### SAM D21
Interrupts are controlled through multiplexer A, Interrupt lines 0-15 Are assigned to PA/PB 0-15 und 16-32. 
So there is the possibility to assign 4 different pins to one interrupt line. This is all done by the Arduino Framework.
Although all externel interrupts share one interrupt vector 
it is possible to attach one handler to each of the external interrupt lines. 
The nasty work is done by the framework.

This allows to use one interrupt handler for each of the KY-040 lines

## Interface
The button lib uses only 6 function to controll the rotary encoder switch
One can request
- the turns since last call
- if the button is currently pressed
- if only a short press was detected
- if the button has been held down for a defined time (long press)

```
void button_init(void);
void button_process(void);
void button_setLong(uint16_t ms);
int8_t button_turns(void);
bool button_down(void);
bool button_pressed(void);
bool button_held(void);
```
The button lib das not use timer intterupts. Therefore the decision for debouncing has to be made in a task function.


### button_init
init has to be calle first.  
Pin configuration and initilisation is done. 
Time for long press ist set to 500 ms

### button_process
needs to be called regulary (~10 ms interval)  
needed for debouncing of the button   
if a capacitor is added to the switch line of the rotary encoder switch 
this function can be omitted and the pin change can be set valid instantly

### button_setLong
set the desired period for a long press.  

### button_turns
returns the count of the rotary encoder since last call.  
The direction is indicated by the signal

### button_down
reflects the actual state of the button

### button_pressed
indicates that the button has been pressed for less than the long time

### button_held
indicates that the button is active for more than the long press time