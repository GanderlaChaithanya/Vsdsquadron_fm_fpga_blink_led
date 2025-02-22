# Vsdsquadron_fm_fpga_blink_led
### Introduction to FM_fpga board:
The VSDSquadron FPGA Mini (FM) is a compact and cost-effective development board tailored for FPGA prototyping and embedded system projects. At its core is the ICE40UP5K FPGA, known for its low power consumption and versatility, making it suitable for a range of applications from educational purposes to professional development. 
# My Project

![fpga_fm](images/.png)

This board offers a variety of input/output interfaces, including GPIO pins, which facilitate seamless integration with other hardware components. Its design emphasizes ease of use, providing developers with a straightforward platform to implement and test their digital designs efficiently 


### Understanding of Verilog code for led_blink:
#### Overview
This Verilog code is designed for an FPGA to control an RGB LED while also implementing a frequency counter. The key components of this design include

##### 1. Internal Oscillator (SB_HFOSC)
   
 - Generates a clock signal to drive the system.
##### 2. Frequency Counter (frequency_counter_i)
    
 - A 28-bit counter that increments on every clock cycle.
        
 - One bit (frequency_counter_i[5]) is output to testwire for debugging or frequency analysis.

   ##### 3. RGB LED Driver (SB_RGBA_DRV)
    
  - Controls a tri-color LED (Red, Green, and Blue).
        
 - Configured to turn ON only the blue LED.
##### 4. Parameter Settings (defparam)
    
Configures the current settings for each LED color
#### Code Walkthrough
##### Module Declaration

```verilog

          module top (  output wire led_red , 
          output wire led_blue ,
           output wire led_green , 
           input wire hw_clk, // Hardware Oscillator, not the internal oscillator output wire output wire testwire );

```

##### Explanation:

- top is the name of the module.
- LED Outputs (led_red, led_green, led_blue) control a tri-color LED.
- hw_clk is an input for an external oscillator,.
- testwire is an output connected to one bit of the frequency counter to observe its behavior.

#### Internal Signal Declaration 

```verilog

             wire int_osc ; 
             reg [27:0] frequency_counter_i;
```
 - int_osc is a wire that will hold the clock signal from the internal  oscillator.
 - frequency_counter_i is a 28-bit register used to count clock cycles.
#### Assigning Test Output

```verilog
 assign testwire = frequency_counter_i[5] 
          
```
- The 6th bit ([5]) of the counter is assigned to testwire.
- Since the counter increments every clock cycle, this bit will toggle at a lower frequency than the original clock.
- This allows external debugging by observing the toggling signal.

 #### Frequency Counter Logic 
 ```verilog
always @(posedge int_osc)
 begin
 frequency_counter_i <= frequency_counter_i + 1'b1;
 end  
          
```
-  Triggered on the rising edge of int_osc (internal clock signal).
- frequency_counter_i increments by 1 on every clock cycle.
- This means the counter acts as a frequency divider, where each bit toggles at a different frequency.
- Bit [0] toggles at full clock frequency.
- Bit [1] toggles at half the clock frequency.
- Bit [5] toggles at Clock Frequency/2^6 frquency, meaning a significantly lower frequency.
#### Internal Oscillator Instantiation 
 ```verilog
SB_HFOSC #(.CLKHF_DIV ("0b10")) u_SB_HFOSC ( .CLKHFPU(1'b1), .CLKHFEN(1'b1), .CLKHF(int_osc)); 
          
```
- SB_HFOSC is a built-in Lattice FPGA primitive that generates an internal    clock.
- CLKHF_DIV ("0b10"): Sets the clock division factor.
- "0b10" means divide the default oscillator frequency by 4.
- If the default clock is 48 MHz, the output int_osc will be 12 MHz.
- CLKHFPU(1'b1): Powers up the oscillator.
- CLKHFEN(1'b1): Enables the oscillator output.
- CLKHF(int_osc): Assigns the generated clock to int_osc, which is used in the counter.
  
 #### RGB LED Driver Instantiation 
  ```verilog
SB_RGBA_DRV RGB_DRIVER (
 .RGBLEDEN(1'b1 ), 
.RGB0PWM (1'b0), // red 
.RGB1PWM (1'b0), // green 
.RGB2PWM (1'b1), // blue
 .CURREN (1'b1 ), 
.RGB0 (led_red ), //Actual Hardware connection 
.RGB1 (led_green ), 
.RGB2 (led_blue ) 
); 
          
```
- SB_RGBA_DRV is a primitive used to drive RGB LEDs on Lattice FPGAs.
- RGBLEDEN(1'b1): Enables the LED driver.
- RGB0PWM (1'b0), RGB1PWM (1'b0), RGB2PWM (1'b1):Red LED (RGB0) and Green LED (RGB1) are OFF.Blue LED (RGB2) is ON.
- CURREN(1'b1): Enables current to the LED outputs.
#### Current Configuration for LEDs 
```verilog
defparam RGB_DRIVER.RGB0_CURRENT = "0b000001"; 
defparam RGB_DRIVER.RGB1_CURRENT = "0b000001";
 defparam RGB_DRIVER.RGB2_CURRENT = "0b000001";
```
-  Configures the current for each LED color.
-  Each LED is set to "0b000001", which is a low current level.
-  This reduces power consumption and prevents damage to the LEDs.

```verilog
endmodule
```
This marks the end of the Verilog module. 
### Key Functionalities of the Code
1. Generates a clock using an internal high-frequency oscillator (SB_HFOSC).
2. Implements a 28-bit counter that increments on every clock cycle.
3. Outputs a test signal (testwire), which is one bit of the counter.
4. Uses the FPGA's built-in RGB LED driver (SB_RGBA_DRV).
5. Configures the LED to keep only the Blue LED ON while keeping Red and Green OFF.
6. Sets low current levels to prevent excessive power usage.

### PCF File in FPGA Design
A PCF (Physical Constraints File) is used in FPGA design to specify pin assignments for the FPGA. It tells the synthesis and place-and-route tools which FPGA pins are connected to specific signals in the design.

##### PCF File in Lattice FPGA
In Lattice FPGAs, a .pcf file is used to define the pin mapping of the FPGA I/O to external hardware.This is especially used in open-source FPGA tools like Yosys and nextpnr for iCE40 FPGAs.
```verilog
                set_io led_red 39 # Assigns LED red signal to pin 39 
                set_io led_blue 40 # Assigns LED blue signal to pin 40 
                set_io led_green 41 # Assigns LED green signal to pin 41 
                set_io hw_clk 20 # Assigns hardware clock input to pin 20 
                set_io testwire 17 # Assigns testwire output to pin 17
```


• Each set_io command maps a Verilog signal to a physical FPGA pin. 
• Verilog module defines led_red, led_blue, led_green, hw_clk, and testwire as ports.
• The PCF file ensures these signals are correctly mapped to the physical FPGA pins

####  Integrating with the VSDSquadron FPGA Mini Board:
The VSDSquadron FM board - Features and specifications: 

•  FPGA: – Powered by the Lattice UltraPlus ICE40UP5K FPGA – Offers 5.3K LUTs, 1Mb SPRAM, 120Kb DPRAM, and 8 multipliers for versatile design capabilities 

•  Connectivity: – Equipped with an FTDI FT232H USB to SPI device for seamless communication – All FTDI pins are accessible through test points for easy debugging and customization

•   General Purpose I/O (GPIO): – All 32 FPGA GPIOs brought out for easy prototyping and interfacing

•   Memory: – Integrated 4MB SPI flash for data storage and configuration

•   LED Indicators: – RGB LED included for status indication or user-defined functionality

•   Form Factor: – Compact design with all pins accessible, perfect for fast prototyping and embedded applications


The VSDSquadron FPGA Mini (FM) board is an affordable, compact tool for prototyping and embedded system development. With powerful ICE40UP5K FPGA, onboard programming, versatile GPIO access, SPI flash, and integrated power regulation, it enables efficient design, testing, and deployment, making it ideal for developers, hobbyists, and educators exploring FPGA applications.






 

