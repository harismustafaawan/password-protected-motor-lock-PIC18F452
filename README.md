# Password-Protected Motor Lock System (PIC18F452)

An embedded systems project implementing a password-protected access control system using the PIC18F452 microcontroller, simulated in Proteus.

## Features
- 4-digit password verification via UART
- 16x2 LCD display for user interaction and status messages
- Timeout detection using Timer0 if input is delayed
- Lockout after 3 failed attempts
- 7-segment display shows failed attempt count
- Motor activation for 60 seconds on correct password entry
- Green/Red LED indicators for access status

## Concepts Used
- Embedded C Programming
- UART Communication
- LCD Interfacing
- Timer/Counter Configuration
- Digital I/O Control
- Proteus Circuit Simulation


## Code

#include "p18f452.h"
#pragma config OSC = HS
#pragma config WDT = OFF
#pragma config LVP = OFF
#pragma config BOR = OFF

#define LCD_RS LATBbits.LATB4
#define LCD_EN LATBbits.LATB5
#define LCD_DATA LATD
#define GREEN_LED LATBbits.LATB0
#define RED_LED LATBbits.LATB1
#define MOTOR LATBbits.LATB2

void delay_ms(unsigned int ms);
void delay_us(unsigned int us);
void LCD_Init(void);
void LCD_Cmd(unsigned char cmd);
void LCD_Char(unsigned char data);
void LCD_String(const char *str);
void UART_Init(void);
char UART_Read(void);
void UART_Write(char data);
void Timer0_Init(void);
void Timer0_Start(void);
void Timer0_Stop(void);
unsigned char Timer0_Timeout(void);
unsigned char my_strcmp(const char* s1, const char* s2);
void display_attempt(unsigned char count);
void LCD_ClearAndSetCursor(unsigned char row, unsigned char col);

const char password[5] = {'1', '2', '3', '4', '\0'};
char input[5];
unsigned char attempt = 0;

void main(void) {
    unsigned char i;
    unsigned char timeout_flag;
    TRISC = 0x80;
    TRISB = 0x00;
    TRISD = 0x00;
    LATB = 0x00;
    LATD = 0x00;
    LATC = 0x00;
    UART_Init();
    LCD_Init();
    Timer0_Init();
    LCD_ClearAndSetCursor(1, 1);
    LCD_String("Ready");
    delay_ms(2000);
    while (1) {
        LCD_Cmd(0x01);
        LCD_String("Enter Pass:");
        LCD_Cmd(0xC0);
        attempt = 0;
        while (1) {
            timeout_flag = 0;
            Timer0_Start();
            for (i = 0; i < 4; i++) {
                while (!PIR1bits.RCIF) {
                    if (Timer0_Timeout()) {
                        timeout_flag = 1;
                        break;
                    }
                }
                if (timeout_flag) break;
                input[i] = UART_Read();
                UART_Write(input[i]);
                LCD_Char('*');
            }
            Timer0_Stop();
            if (timeout_flag) {
                LCD_Cmd(0x01);
                LCD_String("Timeout");
                delay_ms(2000);
                break;
            }
            input[4] = '\0';
            if (my_strcmp(input, password)) {
                LCD_Cmd(0x01);
                LCD_String("Access OK");
                RED_LED = 0;
                for (i = 0; i < 3; i++) {
                    GREEN_LED = 1; delay_ms(300);
                    GREEN_LED = 0; delay_ms(300);
                }
                MOTOR = 1;
                delay_ms(60000);
                MOTOR = 0;
                break;
            } else {
                attempt++;
                LCD_Cmd(0x01);
                LCD_String("Denied");
                RED_LED = 1;
                display_attempt(attempt);
                delay_ms(1000);
                RED_LED = 0;
                if (attempt >= 3) {
                    LCD_Cmd(0x01);
                    LCD_String("Locked");
                    delay_ms(5000);
                    break;
                }
                LCD_Cmd(0x01);
                LCD_String("Enter Pass:");
                LCD_Cmd(0xC0);
            }
        }
    }
}

void delay_ms(unsigned int ms) {
    unsigned int i, j;
    for (i = 0; i < ms; i++) {
        for (j = 0; j < 333; j++) {
            Nop();
        }
    }
}

void delay_us(unsigned int us) {
    while(us--) {
        Nop(); Nop(); Nop(); Nop();
    }
}

void LCD_Init(void) {
    delay_ms(50);
    LCD_Cmd(0x38);
    delay_ms(5);
    LCD_Cmd(0x38);
    delay_ms(1);
    LCD_Cmd(0x38);
    delay_ms(1);
    LCD_Cmd(0x0C);
    delay_ms(2);
    LCD_Cmd(0x01);
    delay_ms(5);
    LCD_Cmd(0x06);
    delay_ms(2);
}

void LCD_ClearAndSetCursor(unsigned char row, unsigned char col) {
    LCD_Cmd(0x01);
    delay_ms(2);
    if(row == 1)
        LCD_Cmd(0x80 + (col - 1));
    else if(row == 2)
        LCD_Cmd(0xC0 + (col - 1));
}

void LCD_Cmd(unsigned char cmd) {
    LCD_RS = 0;
    LCD_DATA = cmd;
    LCD_EN = 1;
    delay_us(10);
    LCD_EN = 0;
    delay_us(100);
}

void LCD_Char(unsigned char data) {
    LCD_RS = 1;
    LCD_DATA = data;
    LCD_EN = 1;
    delay_us(10);
    LCD_EN = 0;
    delay_us(100);
}

void LCD_String(const char *str) {
    while (*str) {
        LCD_Char(*str++);
    }
}

void UART_Init(void) {
    SPBRG = 129;
    TXSTAbits.BRGH = 1;
    RCSTAbits.SPEN = 1;
    TXSTAbits.TXEN = 1;
    RCSTAbits.CREN = 1;
}

char UART_Read(void) {
    while (!PIR1bits.RCIF);
    return RCREG;
}

void UART_Write(char data) {
    while (!PIR1bits.TXIF);
    TXREG = data;
}

void Timer0_Init(void) {
    T0CON = 0x87;
}

void Timer0_Start(void) {
    TMR0H = 0x0B;
    TMR0L = 0xDC;
    INTCONbits.TMR0IF = 0;
    T0CONbits.TMR0ON = 1;
}

void Timer0_Stop(void) {
    T0CONbits.TMR0ON = 0;
    INTCONbits.TMR0IF = 0;
}

unsigned char Timer0_Timeout(void) {
    if (INTCONbits.TMR0IF) {
        INTCONbits.TMR0IF = 0;
        T0CONbits.TMR0ON = 0;
        return 1;
    }
    return 0;
}

unsigned char my_strcmp(const char* s1, const char* s2) {
    while (*s1 && *s2) {
        if (*s1 != *s2) return 0;
        s1++; s2++;
    }
    return (*s1 == '\0' && *s2 == '\0');
}

void display_attempt(unsigned char count) {
    const unsigned char seg_digits[10] = {
        0x3F, 0x06, 0x5B, 0x4F, 0x66,
        0x6D, 0x7D, 0x07, 0x7F, 0x6F
    };
    if (count < 10) {
        LATC = seg_digits[count] & 0x3F;
        LATBbits.LATB6 = (seg_digits[count] & 0x40) ? 1 : 0;
    } else {
        LATC = 0x00;
        LATBbits.LATB6 = 0;
    }
}
