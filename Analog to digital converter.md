# Analog-to-digital-converter
Build an 8051 based data acquisition system to interface analog to digital converter to read potentiometer based voltage divider and display the value on LCD display

# Interfacing ADC0808 with 8051 Microcontroller
ADC is the Analog to Digital converter, which converts analog data into digital format; usually it is used to convert analog voltage into digital format. Analog signal has infinite no of values like a sine wave or our speech, ADC converts them into particular levels or states, which can be measured in numbers as a physical quantity. Instead of continuous conversion, ADC converts data periodically, which is usually known as sampling rate. Telephone modem is one of the examples of ADC, which is used for internet, it converts analog data into digital data, so that computer can understand, because computer can only understand Digital data. The major advantage, of using ADC is that, we noise can be efficiently eliminated from the original signal and digital signal can travel more efficiently than analog one. That’s the reason that digital audio is very clear, while listening.

## ADC0808
ADC0808/0809 is a monolithic CMOS device and microprocessor compatible control logic and has 28 pin which gives 8-bit value in output and 8- channel ADC input pins (IN0-IN7). Its resolution is 8 so it can encode the analog data into one of the 256 levels (28).  This device has three channel address line namely: ADDA, ADDB and ADDC for selecting channel. Below is the Pin Diagram for ADC0808

![image](https://user-images.githubusercontent.com/56119169/191889652-d097a29d-f458-4e55-a44b-27eef437f749.png)

 ADC0808 requires a clock pulse for conversion. We can provide it by using oscillator or by using microcontroller. In this project we have applied frequency by using microcontroller.
 
 # Circuit Diagram:
 ![image](https://user-images.githubusercontent.com/56119169/191889711-ffd2122e-00aa-419d-9e72-8182535b08fa.png)

# Working:
In this project we have interfaced three channels of ADC0808. And for demonstration we have used three variable resistors. When we power the circuit then  microcontroller initialize the LCD by using appropriate command, gives clock to ADC chip,  selects ADC channel by using address line and send start conversion signal to ADC. After this ADC first reads selected ADC channel input and gives its converted output to microcontroller. Then microcontroller shows its value at Ch1 position in LCD. And then microcontroller changes ADC channel by using address line. And then ADC reads selected channel and send output to microcontroller. And show on LCD as name Ch2. And like wise for other channels.

![image](https://user-images.githubusercontent.com/56119169/191889738-1367552d-1789-47c0-8cd3-ecf900a5db56.png)

# Code:
```
#include<reg51.h>
#include<stdio.h>
sbit ale=P3^3;  
sbit oe=P3^6; 
sbit sc=P3^4; 
sbit eoc=P3^5;  
sbit clk=P3^7;  
sbit ADDA=P3^0;  //Address pins for selecting input channels.
sbit ADDB=P3^1;
sbit ADDC=P3^2;
#define lcdport P2  //lcd 
sbit rs=P2^0;
sbit rw=P2^2;
sbit en=P2^1;
#define input_port P1  //ADC
int result[3],number;
void timer0() interrupt 1  // Function to generate clock of frequency 500KHZ using Timer 0 interrupt.
{
clk=~clk;
}
void delay(unsigned int count)  
{
int i,j;
for(i=0;i<count;i++)
  for(j=0;j<100;j++);
}
void daten()
{
    rs=1;
    rw=0;
    en=1;
    delay(1);
    en=0;
}
void lcd_data(unsigned char ch)
{
    lcdport=ch & 0xF0;
    daten();
    lcdport=ch<<4 & 0xF0;
    daten();
}
void cmden(void)
{
    rs=0;
    en=1;
    delay(1);
    en=0;
}
void lcdcmd(unsigned char ch)
{
    lcdport=ch & 0xf0;
    cmden();
    lcdport=ch<<4 & 0xF0;
    cmden();
}
lcdprint(unsigned char *str)  //Function to send string data to LCD.
{
    while(*str)
    {
        lcd_data(*str);
        str++;
    }
}
void lcd_ini()  //Function to inisialize the LCD
{
    lcdcmd(0x02);
    lcdcmd(0x28);
    lcdcmd(0x0e);
    lcdcmd(0x01);
}
void show()
{ 
sprintf(result,"%d",number);
   lcdprint(result);
   lcdprint("  ");
}
void read_adc()
{
   number=0;
   ale=1;
   sc=1;
   delay(1);
   ale=0;
   sc=0;
   while(eoc==1);
   while(eoc==0);
   oe=1;
   number=input_port;
   delay(1);
   oe=0;
}
void adc(int i)  //Function to drive ADC
{
switch(i)
  {
   case 0:
    ADDC=0;  // Selecting input channel IN0 using address lines
    ADDB=0;
    ADDA=0;
    lcdcmd(0xc0);
    read_adc();
    show();
    break;
   case 1:
    ADDC=0;  // Selecting input channel IN1 using address lines
    ADDB=0;
    ADDA=1;
    lcdcmd(0xc6);
    read_adc();
    show();
   break;
   case 2:
    ADDC=0;  // Selecting input channel IN2 using address lines
    ADDB=1;
    ADDA=0;
    lcdcmd(0xcc);
    read_adc();
    show();
    break;
  }
}
void main()
{
 int i=0;
 eoc=1;
 ale=0;
 oe=0;
 sc=0;
 TMOD=0x02;
 TH0=0xFD;
lcd_ini();
lcdprint(" ADC 0808/0809  ");
lcdcmd(192);
lcdprint("  Interfacing   ");
delay(500);
lcdcmd(1);
lcdprint("Converting");
lcdcmd(192);
lcdprint("System Ready...   ");
delay(500);
lcdcmd(1);
lcdprint("Ch1   Ch2   Ch3 ");
 IE=0x82;
 TR0=1;
while(1)
{
   for(i=0;i<3;i++)
   {
     adc(i);
     number=0;
   }
}
}
 

```
# Circuit Designing:
![image](https://user-images.githubusercontent.com/56119169/191889990-b94bfe4c-9526-4968-8b94-a093d461bba9.png)

![image](https://user-images.githubusercontent.com/56119169/191890034-006739d6-7435-4532-9fea-0acb7c7f197c.png)

## ADC  Display output:
![image](https://user-images.githubusercontent.com/56119169/191890080-99161a3a-1ad1-437b-8d8b-8016d19af303.png)
