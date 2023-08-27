---
layout: post
author: jack
category: [linux-and-more,hardware]
---

From quite some time, I have been thinking about building the 8-bit computer based on SAP-1 architecture that is demoed on Ben Eater's YouTube channel. But I never gave any further thoughts and always left that idea behind just because I thought it is a bit complex and involved a lot of components. However, I recently I saw a video as part of that series where an EEPROM programmer is build using Arduino and a shiftregister. It barely has more than 3 components to build something that interesting. I think that is the only video with the longest duration in the entire series. So, based on the number of components used in the process, I think its doeable and started planning to do it during my vacation because at some point I will be bored and getting engaged in this task can be pretty fun. 

So I started collecting all the components that are required - an EEPROM (28C16 2K x 8 from Excel Microelectronics), 8-bit shiftregisters (74HS595 from ST Microelectronics), breadboard, jumper wires and an Arduino Uno board (I tried with nano version but maybe because of not making it sit on the breadboard correctly, it didn't quite do the job). As you can see, it involved all the regular components though I had to run between shops to find the EEPROM itself. I planned to just implement what is said on the video. To a person like me who barely worked on any ICs and did any Arduino projects, replicating what is done on the video is a big achievement. So, I thought that would be a good starting point.
<br>
<b>Note:</b> When I started building it, I had a few issues with the LEDs itself. So I thought to print the output directly to the serial monitor instead of observing it using LEDs. Hence I didn't go for another round of shopping.
<br><br>
<b>Testing and configuring Arduino for the 74595 shiftregisters - </b>
<br>
To begin with, I first tested the shiftregister itself. Pin 11 is the serial data pin, pin 12 is the clock pin for the register itself and pin 14 is the clock of the latch inside the 595 IC. Unlike 74164 shiftregister IC, the 74595 IC has an output storage register. The main difference that I observed is the way the data is released via the parallel output pins. For the 74595, there is an option to store the output and not release it immediately hence the latch clock. And there is no such thing for 74164 serial-in parallel-out 8-bit shiftregister. 
First we define the pins and then we declare the pinModes. There is a very comfortable function to out the serial data in the Arduino library which is called shiftOut(). So once the data is serially sent from the Arduino to the inputs of the shift register, it is then stored in that 8-bit output register. Only if the latch clock is high, this data is sent out. Hence we create a pulse using digitalWrite() function. 
<br>
Below code should send serial data to the shiftregister and print back the output of the IC via serial monitor. I think this is the best way to work if LEDs are not present unlike that is shown in the video.

<!-- <p style="background-color: rgb(164, 168, 168); margin-right: 20%;"> -->
```c++

#define SHIFT_DATA 2 // pin 11 in 595 
#define SHIFT_CLK 3 // pin 12 in 595
#define SHIFT_LATCH 4 // pin 14 in 595

void setup() {
    pinMode(SHIFT_DATA, OUTPUT);
    pinMode(SHIFT_CLK, OUTPUT);
    pinMode(SHIFT_LATCH, OUTPUT);

    shiftOut(SHIFT_DATA, SHIFT_CLK, MSBFIRST, #data);
    digitalWrite(SHIFT_LATCH, LOW);
    digitalWrite(SHIFT_LATCH, HIGH);
    digitalWrite(SHIFT_LATCH, LOW);
    
    Serial.begin(9600);

    # outputs of 74595 are connected to Arduino digital pins from 5 to 12.
    for (pin = 5; pin <= 12; pin += 1){
        pinMode(pin, OUTPUT);
        Serial.println(digitalWrite(pin));
    }
    
```

<b>Confiuguring two shiftregisters to store 11 bits for the address info- </b>
<br>
The real reason behind using shiftregisters is because of the number of pins available to set address and access I/O ports in EEPROM. Arduino only has 13 digital pins out of which 0 and 1 can't be used because of the serial monitor feature we are using to get the data from EEPROM. Hence we need to use the next 3 pins are for shiftregisters and next 8 pins are for reading the data. We use the last digital pin #13 for EEPROM's write enable signal. This is important because we need to enable/disable when we are writing/reading the data respectively. There is also this outputEnable pin for EEPROM which lets us see the data that is inside it. Also since now we are using 11 bits of data, we need to connect pin 9 (serial output) of the first shiftregister to the pin 11 (serial input) of the second shiftregister. This way we will be able to store 11 bits along with the outputEnable bit in the two shiftregisters. 
We implement the same idea inside the code. 

```c++
void setAddress(int address, bool outputEnable){

    shiftOut(SHIFT_DATA, SHIFT_CLK, MSBFIRST, (address >> 8) | (outputEnable ? 0x00 : 0x80) );
    shiftOut(SHIFT_DATA, SHIFT_CLK, MSBFIRST, address);
    
    digitalWrite(SHIFT_LATCH, LOW);
    digitalWrite(SHIFT_LATCH, HIGH);
    digitalWrite(SHIFT_LATCH, LOW);
    
    }
```


<b>Reading data from EEPROM - </b>
<br>

Now since we have configured our shift registers to send data for 11 address lines and 1 outputEnable, we can actually read data that is inside the EEPROM. Before we write code to do that, we first need to hook up our EEPROM in the setup. Pins 1 to 8 are the first 8 address lines that will be coming from the first shiftregister and pin 23, 22 and 19 are the last three address bits coming from the second register. Also the outputEnable i.e., pin 20 is coming from the last output bit (pin 7) of the second shiftregister. Pins 9 to 11 and 13 to 17 are connected to the digital pins of the Arduino along with the writeEnable (pin 21) to the last digital pin (pin 13 of the Arduino). Remaining connections are obvious from the data sheet. Now given all connections are set, we write the code to read the data from EEPROM.
<br>
Below code reads the EEPROM content in the address location #2 - 

```c++

#define SHIFT_DATA 2 // pin 11 in 595 
#define SHIFT_CLK 3 // pin 12 in 595
#define SHIFT_LATCH 4 // pin 14 in 595 
#define EEPROM_DO 5  
#define EEPROM_D7 12 
#define WRITE_ENABLE 13 

void setAddress(int address, bool outputEnable){ 

    shiftOut(SHIFT_DATA, SHIFT_CLK, MSBFIRST, (address >> 8) | (outputEnable ? 0x00 : 0x80) ); 
    shiftOut(SHIFT_DATA, SHIFT_CLK, MSBFIRST, address);
    
    digitalWrite(SHIFT_LATCH, LOW);
    digitalWrite(SHIFT_LATCH, HIGH);
    digitalWrite(SHIFT_LATCH, LOW); 

} 

byte readEEPROM(int address){ 

    for (int pin = 5; pin <= 12; pin += 1){ 
    pinMode(pin, INPUT); 
    } 
    setAddress(address, true); 
    byte data = 0; 
    for (int pin = 12; pin >= 5; pin -= 1) { 
    data = (data << 1) + digitalRead(pin); 
    } 
    return data; 
} 

void setup() {
    // put your setup code here, to run once:
    
    pinMode(SHIFT_DATA, OUTPUT);
    pinMode(SHIFT_CLK, OUTPUT); 
    pinMode(SHIFT_LATCH, OUTPUT); 
    
    digitalWrite(13, HIGH);
    pinMode(13, OUTPUT);
    
    Serial.begin(9600);
    
    // Read individual address 
    Serial.println(readEEPROM(2));

```

<b>Writing data to EEPROM - </b>


Writing the data is very much similar to that of reading the data but only thing that needs to be done in addition is to enable the writeEnable (connected to pin#13 of Arduino) and set the pinmodes of Arduino. The delay something that needs to be picked from the datasheet. The value mentioned is 1 us but I gave 10us to increase the window size but this is not necessary. Ben Eater in his video troublshoots this part for quite some time. Below is the code to do that - 

```c++

void writeEEPROM(int address, byte data){ 
    for (int pin = 5; pin <= 12; pin += 1) { 
        pinMode(pin, OUTPUT);
        }
        setAddress(address, false);
        for (int pin = 5; pin <= 12; pin += 1) {
        digitalWrite(pin, data & 1);
        data = data >> 1;
        }
        digitalWrite(13, LOW);
        delayMicroseconds(10);
        digitalWrite(13, HIGH);
        delay(10);
    
    }
```

    
</p>            

<b>Complete code to read and write data to EEPROM is available on GitHub - <a href="https://github.com/saicharan0112/eeprom_programmer/blob/main/eeprom_shiftreg_testing.ino">link to my GitHub repository</a> </b>


Here is the image of my final setup where between all those jumper wires forest lies those three ICs. 
<img src="/assets/images/eeprom_programmer.jpg" alt="eeprom_programmer_final_setup" style="width: 800px;">


<b>Closing thoughts - </b>

It was really exciting to build something like this. In the end, I had an EEPROM with the 7-segment display code available. I can reuse this process to build a lookup table to store values for operations and also store opcodes for something like the 8-bit computer. Given I never used EEPROMs and shiftregisters in this manner, it was really educational with all those troublshoots in the process. Especially learning the art of handling jumper wires and neatly wiring them.