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
