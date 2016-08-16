Date: 2012-07-15 14:00
Title: Using the Sparkfun LCD shield with the MSP430 - Part 1
Category: MSP430

After declaring myself a microcontroller hipster, and deciding the Arduino is
too mainstream (That and I dislike the software, there's no debugging, and it
produces rather bloated code...), I decided to pull out my TI Launchpad I got 
back when they were first announced, and trying to do some stuff with that.

I was at a local electronics store the other day, and I noticed they had
Sparkfun's [Color LCD Shield](http://www.sparkfun.com/tutorials/286). Granted,
it's for the Arduino, but I figured I could use it anyhow just by plugging in
wires.

After many attempts at getting it working, I borrowed my girlfriend's Arduino,
and started fiddling with the provided code samples. After a bit of experimentation,
I discovered that it was a Phillips controlled LCD, and not an Epson like I thought
at first (Three days of trying to get that working...). Once I had that sorted, 
it was rather easy going.

Since there is no good reference I can find for using the controller with an MSP430,
I've decided to post the relevent bits here. Keep in mind, this is for the Phillips
version. It shouldn't be hard to modify for the Epson interfaces. It's based on
[James Lynch's documentation](http://www.sparkfun.com/tutorial/Nokia%206100%20LCD%20Display%20Driver.pdf),
so consult that if you wish to adapt this code for the Epson LCDs. One thing I noticed
using the clear LCD code from there, was that the last row on the LCD would still be garbage
after filling, so instead of `(131*131)` I use `(131*132)`, and it clears all the rows.

    :::C
    #define LCD_P_SLEEPOUT	0x11
    #define LCD_P_INVON		0x21
    #define LCD_P_COLMOD	0x3A
    #define LCD_P_MADCTL	0x36
    #define LCD_P_SETCON	0x25
    #define	LCD_P_DISPON	0x29
    #define LCD_P_RAMWR		0x2C
    #define LCD_P_CASET		0x2A
    #define LCD_P_PASET		0x2B

    #define		LCD_OUT		P1OUT
    #define		LCD_DIR		P1DIR
    #define		LCD_RES		BIT3
    #define		LCD_CS		BIT4
    #define		LCD_CK		BIT5
    #define		LCD_MOSI	BIT6

    #define		PIXELCOUNT	(131*132)/2
    
    void WriteMessage(unsigned int message){
        unsigned char width=9;
        LCD_OUT &= ~LCD_CS;

        while(width--){
            LCD_OUT &= ~LCD_CK;
            if(message & 0x8000){
                LCD_OUT |= LCD_MOSI;
            } else {
                LCD_OUT &= ~LCD_MOSI;
            }
            LCD_OUT |= LCD_CK;
            message <<= 1;
        }
        LCD_OUT |= LCD_CS;
    }
    
    void LcdWrite(unsigned char command, const unsigned char* data, int length){
        unsigned int out=0;
        unsigned char i;
        out = command << 7;
        WriteMessage(out);

        for(i=0;i<length;i++){
            out = (1<<15) + (data[i] << 7);
            WriteMessage(out);
        }
    }

    void LcdInit(){
        LCD_OUT &= ~LCD_RES;
        __delay_cycles(1000);
        LCD_OUT |= LCD_RES;
        __delay_cycles(1000);

        static const unsigned char colmodData[] = {0x03};
        static const unsigned char madctlData[] = {0xC8};
        static const unsigned char setconData[] = {0x30};
        LcdWrite(LCD_P_SLEEPOUT,0,0);
        LcdWrite(LCD_P_COLMOD,colmodData,1);
        LcdWrite(LCD_P_MADCTL,madctlData,1);
        LcdWrite(LCD_P_SETCON,setconData,1);
        __delay_cycles(1000);
        LcdWrite(LCD_P_DISPON,0,0);
    }

    void LcdClear(unsigned int color){
        unsigned int i;
        static const unsigned char pasetParams[] = {0,131};
        static const unsigned char casetParams[] = {0,131};
        LcdWrite(LCD_P_PASET, pasetParams,2);
        LcdWrite(LCD_P_CASET, casetParams,2);

        WriteMessage(LCD_P_RAMWR << 7);

        unsigned int color1 = ((((color)>>4)&0xFF) << 7) + ((unsigned int)1 << 15);
        unsigned int color2 = ((((((color)&0x0F)<<4) | ((color)>>8))) << 7) +((unsigned int)1 << 15);
        unsigned int color3 = (((color) & 0xFF) << 7) + ((unsigned int)1<<15);
        for(i=PIXELCOUNT; i>0;i--){
            WriteMessage(color1);
            WriteMessage(color2);
            WriteMessage(color3);
        }
    }
    
A few notes about this code:

 1. I've tested this at 1, 12 and 16MHz, and have had no issue. I assume that all 
    frequencies in between will work.
 2. Bitbanging is used, since the 430 I'm testing with has a USCI and not a USI. 
    SPI with USCI only appears to support 8 and 16 bit transfers.
 3. Clearing the screen, even at 16MHz, takes a disappointing amount of time
    (200ms or so, judging by the LED I have that toggles every time it clears).
    I need to fiddle with the bitbanging code some.
 4. For some reason, colors appear to be in BGR ordering instead of RGB. Not a major
    issue, but still kinda strange.

I'm going to work on making this code faster, and get some graphics routines in, and make another post when I do