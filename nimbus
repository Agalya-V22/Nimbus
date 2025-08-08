// PIC16F877A Configuration Bit Settings
// 'C' source line config statements

// CONFIG
#pragma config FOSC = HS        // Oscillator Selection bits (HS oscillator)
#pragma config WDTE = OFF       // Watchdog Timer Enable bit (WDT disabled)
#pragma config PWRTE = OFF      // Power-up Timer Enable bit (PWRT disabled)
#pragma config BOREN = ON       // Brown-out Reset Enable bit (BOR enabled)
#pragma config LVP = OFF        // Low-Voltage (Single-Supply) In-Circuit Serial Programming Enable bit (RB3 is digital I/O, HV on MCLR must be used for programming)
#pragma config CPD = OFF        // Data EEPROM Memory Code Protection bit (Data EEPROM code protection off)
#pragma config WRT = OFF        // Flash Program Memory Write Enable bits (Write protection off; all program memory may be written to by EECON control)
#pragma config CP = OFF         // Flash Program Memory Code Protection bit (Code protection off)

// #pragma configure statements should precede project file includes.
// Use project enums instead of #define for ON and OFF.
#include <xc.h>
#include <stdio.h>          // For sprintf()

#define _XTAL_FREQ 20000000
#define MOTOR RB0           // Motor connected to RB0

// LCD connections (4-bit mode)
#define RS RD0
#define EN RD1
#define D4 RD2
#define D5 RD3
#define D6 RD4
#define D7 RD5



// === LCD Functions ===
void lcd_pulse() {
    EN = 1; __delay_ms(2); EN = 0; __delay_ms(2);
}

void lcd_command(unsigned char cmd) {
    // High nibble
    D4 = (cmd>>4)&1; D5 = (cmd>>5)&1; D6 = (cmd>>6)&1; D7 = (cmd>>7)&1;
    RS = 0; lcd_pulse();
    // Low nibble
    D4 = cmd&1; D5 = (cmd>>1)&1; D6 = (cmd>>2)&1; D7 = (cmd>>3)&1;
    RS = 0; lcd_pulse();
    __delay_ms(2);
}


void lcd_data(unsigned char data) {
    // High nibble
    D4 = (data>>4)&1; D5 = (data>>5)&1; D6 = (data>>6)&1; D7 = (data>>7)&1;
    RS = 1; lcd_pulse();
    // Low nibble
    D4 = data&1; D5 = (data>>1)&1; D6 = (data>>2)&1; D7 = (data>>3)&1;
    RS = 1; lcd_pulse();
    __delay_ms(2);
}

void lcd_init() {
    TRISD = 0x00;  // LCD PORTD as output
    __delay_ms(20);
    lcd_command(0x02);  // 4-bit mode
    lcd_command(0x28);  // 2-line, 5x7
    lcd_command(0x0C);  // Display on, cursor off
    lcd_command(0x06);  // Entry mode
    lcd_command(0x01);  // Clear
    
}

void lcd_set_cursor(unsigned char row, unsigned char col) {
    unsigned char pos;
    pos = (row==0)?0x80:0xC0;
    pos += col;
    lcd_command(pos);
}

void lcd_print(char *str) {
    while(*str) lcd_data(*str++);
}

// === ADC Function ===
unsigned int readADC(unsigned char channel) {
    ADCON0 = (channel << 3) | 0x81;  // Select channel + ADC ON
    __delay_us(50);
    GO_nDONE = 1;
    while(GO_nDONE);                 // Wait for conversion
    return ((ADRESH << 8) + ADRESL); // Return 10-bit value
}



// === Main Program ===
void main() {
    unsigned int tempADC, humADC;
    float tempC, humidity;
    char buffer[16];

    // I/O Configuration
    TRISA = 0xFF;  // RA0=Temp, RA1=Humidity
    
    TRISB = 0x00;  // RB0=Motor
    MOTOR = 0;

    // ADC Configuration
    ADCON1 = 0x80; // Right justified, Vref=Vdd

    // LCD Initialization
    lcd_init();
    lcd_set_cursor(0,0);
    
    lcd_print("Cloud Seeding");

    while(1) {
        // === Sensor Reading ===
        tempADC = readADC(0);   // LM35 on RA0
        humADC  = readADC(1);   // Humidity sensor on RA1

        // === ADC to Physical Values ===
        tempC = (tempADC * 5.0 / 1023.0) * 100.0;  // LM35: 10mV per °C
        humidity = (humADC * 100.0) / 1023.0;      // Approx 0-100%

        // === Display on LCD ===
        lcd_set_cursor(1,0);
        sprintf(buffer,"T:%2.1fC H:%2.0f%%", tempC, humidity);
        lcd_print(buffer);

        // === Motor Control ===
        if(tempC > 30 && humidity < 60) {
            MOTOR = 1; // Motor ON
        } 
        else if(tempC < 25 || humidity > 70) {
            MOTOR = 0; // Motor OFF
        }

        __delay_ms(500);
    }
}