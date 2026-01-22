# 25-26-SEM1-SKEE3223---SEC13-GRP-6
#include "stm32f4xx.h"

// 7-segment codes for 0~9
uint8_t seg_table[10] = {0x3F,0x06,0x5B,0x4F,0x66,0x6D,0x7D,0x07,0x7F,0x6F};

// Countdown variables
volatile uint8_t dice1=0, dice2=0;           // Desired digits
volatile uint8_t countdown1=0, countdown2=0; // Current countdown
volatile uint8_t running=0;

// Flags set by interrupts
volatile uint8_t pa0_flag=0, pa1_flag=0, pa9_flag=0, pa10_flag=0;

// Last button state for edge detection in main loop
uint8_t last_pa0=1, last_pa1=1, last_pa9=1, last_pa10=1;

// ----------------- Subroutine: simple delay -----------------
void delay_ms(uint32_t ms)
{
SysTick->LOAD = 16000-1; // 1ms at 16MHz
SysTick->VAL = 0;
SysTick->CTRL = 5;
for(uint32_t i=0;i<ms;i++)
    while(!(SysTick->CTRL & (1<<16)));
SysTick->CTRL = 0;
}

// ----------------- EXTI Handlers -----------------
void EXTI0_IRQHandler(void)  // PA0 Start/Pause
{
    if(EXTI->PR & (1<<0)) { EXTI->PR |= (1<<0); pa0_flag=1; }
}
void EXTI1_IRQHandler(void) // PA1 Reset
{
    if(EXTI->PR & (1<<1)) { EXTI->PR |= (1<<1); pa1_flag=1; }
}
void EXTI9_5_IRQHandler(void) // PA9 Tens
{
    if(EXTI->PR & (1<<9)){EXT->PR |=(1<<9); pa9_flag=1;}
}
void EXTI15_10_IRQHandler(void) //PA10 Units
{
    if(EXTI->PR & (1<<10) {EXTI ->PR |=(1<<10); pa10_flag=1; }


// ----------------- Main Function -----------------
intmain(void)
{
    // Enable clocks
   RCC->AHB1ENR |= (1<<0)|(1<<2); // GPIOA + GPIOC
   RCC->APB2ENR |= (1<<14); // SYSCFG
   
   // Configure PA0, PA1, PA9, PA10 as input pull-up
   GPIOA->MODER &= ~((3<<0*2)|(3<<1*2)|(3<<9*2)|(3<<10*2));
   GPIOA->PUPDR &= ~((3<<0*2)|(3<<1*2)|(3<<9*2)|(3<<10*2));
   GPIOA->PUPDR |= ((1<<0*2)|(1<<1*2)|(1<<9*2)|(1<<10*2));
   
   // Configure PC0–PC15 as output
   GPIOC->MODER = 0x55555555;
   GPIOC->ODR = 0x0000;
   
   // Configure EXTI
   SYSCFG->EXTICR[0] &= ~((0xF<<0)|(0xF<<4)); // EXTI0->PA0, EXTI1->PA1
   SYSCFG->EXTICR[2] &= ~((0xF<<4)|(0xF<<8)); // EXTI9->PA9, EXTI10->PA10
   
   EXTI->IMR |= (1<<0)|(1<<1)|(1<<9)|(1<<10); // Unmask EXTI
   EXTI->FTSR |= (1<<0)|(1<<1)|(1<<9)|(1<<10); // Falling edge
   
   NVIC_SetPriority(EXTI0_IRQn, 2);
   NVIC_SetPriority(EXTI1_IRQn, 2);
   NVIC_SetPriority(EXTI9_5_IRQn, 2);
   NVIC_SetPriority(EXTI15_10_IRQn, 2);
   NVIC_EnableIRQ(EXTI0_IRQn);
   NVIC_EnableIRQ(EXTI1_IRQn);
   NVIC_EnableIRQ(EXTI9_5_IRQn);
   NVIC_EnableIRQ(EXTI15_10_IRQn);
   
   // Initialize display
   countdown1 = 0;
   countdown2 = 0;
   GPIOC->ODR =(seg_table[countdown2]<<8) | seg_table[countdown1];
   
   uint32_t tick_ms_counter = 0;

while(1)
{
    // ---------- Main loop handles button edge detection ----------
    uint8_t pa0 = (GPIOA->IDR & (1<<0)) ? 1:0;
    uint8_t pa1 = (GPIOA->IDR & (1<<1)) ? 1:0;

    // PA0 Start/Pause
    if(pa0_flag) {
        pa0_flag=0;
        if(last_pa0==1 && pa0==0) { running=!running; countdown1=dice1; countdown2=dice2; delay_ms(20); }
     }
    // PA1 Reset
    if(pa1_flag) {
        pa1_flag=0;
        if(last_pa1==1 && pa1==0) { running=0; dice1=dice2=0; countdown1=dice1; countdown2=dice2; delay_ms(20); }
    }
    // PA9 Tens
   if(pa9_flag){
       pa9_flag=0;
       if(last_pa9==1&&pa9==0){dice2++; if(dice2>9)dice2=0; if(!running)countdown2=dice2; delay_ms(20);}
       }

       last_pa0 = pa0;
       last_pa1 = pa1;
       last_pa9 = pa9;
