# Assembly-langguge-X86-Arm-processor-ECE372--Project2
Full report ECE372 Project2 2018.docx and code "* .txt"
# Part1 SETUP I2C
## Initialization for I2C
To map the I2C, I have to know the pins available to the Beagle Bones Black P9 and P8 connector by changing the MUX that select the signal go to the pads.
 ![alt-text](https://github.com/Phasor2/Assembly-langguge-X86-Arm-processor-ECE372--Project2/blob/master/MUX.png)
The I2C that is needed for this project is I2C1_SDA_MUX3, and I2C1_SCL_MUX3. The default of the MUX when initialized it will begin in MODE0. Essentially in the manual ARM355x, the register name that use to map will be named in MODE0. So all we need to look up spi0_d1, spi0_cs0 change to mode 2.
After mapping and started the I2C clock at CM_PER_I2C1_CLKCTRL, next step is setting the module clock for I2C to communicate with LCD screen. First of all, is setting Prescaler value I2C_PSC offset 0xB0. Initially, value default for Prescaler is 48MHz, the value that we write to the register will be divider for the Prescaler. In this case, we want a 12 MHz for our I2C so 48/4=12 MHz. We will write 0x3 to register. 
Next setting data rate, we want 100kbps by writing value for SCLH and SCLL. Here we need some calculation
First finding period T:
T=1/F=1/100kps=10μs
	After having period, we can find period for tLow and tHigh
						tHigh=tLow=T/2=10μs/2=5μs
	Next we find internal rate clock
ICLK=1/12Mhz=83ns
	The formula that provided in the registers are 
{█(tLow=(SCLL+7)×ICLK@tHigh=(SCLH+5)×ICLK)┤
{█(SCLL=53.2≈53=0X35@SCLH=55.2≈55=0X37)┤
	So we write 0x35 for 0x4802A035(SCLL) and 0x37 for 0x4802A037(SCLH)	
Setting Own Address is the next step, I2C_OA offset 0xA8. For this register, we simply write 0x00.
Setting Slave Address I2C_SA offset 0x78. For this register, we write 0x3C.
Setting configuration for I2C_CON offset 0xA4, in this register there are many feature bits. But we only need bit 10 MST and bit 9 TRX, bit 1 STT, and bit 0 STP. For now, we write the value 0x8600 to the register. we don’t want to start the transmission yet.
Finally, We set the Count register I2C_CNT this register depend on how many bytes we want to write out to the I2C including instruction and ASCII. In this initialization, I just write to initialize the screen so write 0xC to the register.
The important part of the transmission is the XRDY bit in IRQ_RAW_STATUS register. Since the LCD does not talk back to the processor, so we just need to pay attention to this bit for now.
When setting the I2C_CON register, notice the XRDY is set to 1 indicate the acknowledge and ready to write from I2C. One other thing to look for is AERR Access error bit and BB bit bus busy, AERR bit is also let us know if there is something wrong with the I2C transmission. And BB is indicating if the bus is busy.
	
Now for the Interrupt part, there are 2 main interrupt signal will be generated from the UART. They are THR interrupts and MODEM status interrupts. Initialized these 2 will be for the part of sending character through RS-232C.
For part of the project, there is no requirement for FIFO. So we also need to disable FIFO.
