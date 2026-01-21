# 25-26-SEM1-SKEE3223---SEC13-GRP-6
#include "stm32f4xx.h"

// 7-segment codes for 0~9
uint8_t seg_table[10] = {0x3F,0x06,0x5B,0x4F,0x66,0x6D,0x7D,0x07,0x7F,0x6F};

// Countdown variables
volatile uint8_t dice1=0, dice2=0;           // Desired digits
volatile uint8_t countdown1=0, countdown2=0; // Current countdown
volatile uint8_t running=0;

// Flags set by interrupts
volatile uint8_t pa0_flag=0;

// Last button state for edge detection in main loop
uint8_t last_pa0=1;

// ----------------- EXTI Handlers -----------------
void EXTI0_IRQHandler(void)  // PA0 Start/Pause
{
    if(EXTI->PR & (1<<0)) { EXTI->PR |= (1<<0); pa0_flag=1; }
}
