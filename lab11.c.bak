#include <targets\AT91SAM7.h>
#include <ctl_api.h>
#include <string.h> // Pracujemy na ³añcuchu
#include <pcf8833u8_lcd.h>

void czas (int ms){
  volatile int aa,bb;
  for(aa=0;aa<=ms;aa++){
    for(bb=0;bb<=3000;bb++){
      __asm__("NOP");
      }}}

// Funkcja pomocnicza do obliczania czêstotliwoœci MCK
/*unsigned long ctl_at91sam7_get_mck_frequency(unsigned long mainck_frequency) {
   unsigned long result = 0;
   switch (PMC_MCKR & 3) {
       case 0:
           result = 32768;
           break;
       case 1:
           result = mainck_frequency;
           break;
       case 2:
       case 3:
           result = mainck_frequency * (((CKGR_PLLR >> 16) & 0x7FF) + 1) / (CKGR_PLLR & 0xFF);
           break;
   }
   return result / (1 << ((PMC_MCKR >> 2) & 7));
}*/
// Funkcja wysy³aj¹ca jeden znak przez USART
void usart_send_char(char c) {
   while (!(US0_CSR & (1 << 1))); // Poczekaj na pusty rejestr nadawczy
   US0_THR = c; 
}
 
void send_string(char *str){
  while(*str){
    usart_send_char(*str++);
  }
}
volatile char PIN[4] = {'1', '2', '3', '4'};
volatile int TRYB_GRY = 0;

void wprowadzPIN() {
  LCDPutStr("Podaj PIN: ", 10, 15, 2, 0xFFF, 0x000);
  send_string("\nPodaj PIN: ");
  char znak[] = {'0', '0', '0', '0'};
  int a = 0;
  int licznik = 0;
  //char PIN[] = {'1', '2', '3', '4'};
  while(a < 4) {
    if (US0_CSR & 1 << 0){
          znak[a]= US0_RHR;
          
          usart_send_char(znak[a]);
          usart_send_char('\n');
          usart_send_char(PIN[a]);
          
          a = a + 1;  
       }
    
  }


  for (int i = 0; i < 4; i++) {
      if (PIN[i] == znak[i]) {
        licznik = licznik + 1;
      }
    }
    if (licznik == 4 ) {
      send_string("PIN OK");
      LCDPutStr("PIN OK ", 50, 15, 2, 0xFFF, 0x000);
    }

}

void wprowadzPINxRazy() {
  LCDPutStr("Podaj PIN: ", 10, 15, 2, 0xFFF, 0x000);
  send_string("\n\rPodaj PIN: ");
  char znak[] = {'0', '0', '0', '0'};
  
  int proby = 0;
  //char PIN[] = {'1', '2', '3', '4'};
  while (proby < 5) {
    int a = 0;
    int licznik = 0;
    while(a < 4) {
      if (US0_CSR & 1 << 0){
	znak[a]= US0_RHR;
          
	//usart_send_char(znak[a]);
	//usart_send_char('\n');
	//usart_send_char(PIN[a]);
          
	a = a + 1;  
      }
    
    }

    for (int i = 0; i < 4; i++) {
      if (PIN[i] == znak[i]) {
        licznik = licznik + 1;
      }
    }
    if (licznik == 4 ) {
      send_string("\n\rPIN OK");
      LCDPutStr("PIN OK ", 50, 15, 2, 0xFFF, 0x000);
      TRYB_GRY = 0;
      LCDClearScreen();
      return;
    } else {
      send_string("\n\rPIN nieprawidlowy\n");
      proby = proby + 1;
    }
  }
}

volatile int timer_dla_gry = 0;

TC0_ISR() {
  TC0_SR;
  timer_dla_gry = 1;
}

void resetPIN() {
    char newPIN[4] = {'0', '0', '0', '0'};
    int a = 0;

    send_string("\n\rWprowadz nowy PIN: ");
    LCDPutStr("Wprowadz nowy PIN", 10, 15, 2, 0xFFF, 0x000);

    while (a < 4) {
      if (US0_CSR & (1 << 0)) {
	newPIN[a] = US0_RHR;
	usart_send_char(newPIN[a]);
	a = a + 1;
      }
      
    }
    for (int i = 0; i < 4; i++) {
      PIN[i] = newPIN[i];
    }
    send_string("\n\rNowy PIN ustawiony.\n");
 }

volatile int check_isr = 0, isr_x = 110, isr_y =110;

JOY_ISR() {
  int push = PIOA_ISR;
  if ((push & 1 << 15) != 0) {
    if (TRYB_GRY == 2 && check_isr == 1) {
      //TRYB_GRY = 0;
      check_isr = 0;
    }
    
  }
}
int main() {
   ctl_global_interrupts_disable();
   // Konfiguracja PMC (w³¹czenie PIOA i USART0)
   PMC_PCER = (1 << 2) | (1 << 6) | (1 << 3) | (1 << 10) | (1 << 17) | (1 << 12); // W³¹czenie zegara dla PIOA i USART0
   // Konfiguracja pinów zwi¹zanych z USART0
   PIOA_PDR = (1 << 0) | (1 << 1); 
   PIOA_ASR = (1 << 0) | (1 << 1); // Ustawienie funkcji alternatywnej A dla USART0
   // Konfiguracja PWM
   PIOB_PDR = 1 << 20;
   PIOB_ASR = 1 << 20;
   PWM_CMR1 |= 1 << 3 | 1 << 9 | 1 << 10; // /256, channel polarity, 
   PWM_CPRD1 = 1023 * 3;
   PWM_ENA |= 1 << 1;
   PWM_CDTY1 =1022*3;
   // Konfiguracja ADC
   ADC_CR = 1 << 0;
   ADC_CHER = 1 << 6;
   ADC_MR = (23 << ADC_MR_PRESCAL_BIT) | (2 << ADC_MR_STARTUP_BIT) | (1 << ADC_MR_SHTIM_BIT);
   ADC_CR = 1 << 1;
   int trim_val = 0;

   //Konfiguracja TC0
   TC0_CCR = 1 << 1;
   TC0_IDR = 0xFF;
   TC0_SR;
   TC0_CMR |= 1 << 2 | 1 << 14;
   TC0_RC = 46812; // 1 s = 46812, 0.5s = 23406, 0.33s = 15506
   ctl_set_isr(12, 1, CTL_ISR_TRIGGER_FIXED, TC0_ISR, 0);
   TC0_IER = 1 << 4;
   ctl_unmask_isr(12);
   TC0_CCR |= 1 << 0 | 1 << 2;
   // Resetowanie i ustawienie trybu USART0
   
   US0_CR = (1 << 2) | (1 << 3) | (1 << 5) | (1 << 7); // Reset odbiornika i nadajnika, wy³¹czenie obu
   US0_MR = (3 << 6) | (4 << 9); // Tryb normalny, d³ugoœæ znaku 8-bit, brak parzystoœci
   // Ustawienie baud rate
   unsigned long baudrate = 9600;
   unsigned long masterClock = ctl_at91sam7_get_mck_frequency(OSCILLATOR_CLOCK_FREQUENCY);
   US0_BRGR = (masterClock / baudrate) / 16;
   // W³¹czenie nadajnika i odbiornika
   US0_CR = (1 << 4) | (1 << 6); // W³¹czenie nadajnika i odbiornika
   PIOA_PER = 1 << 7 | 1 << 8 | 1 << 9 | 1 << 14 | 1 << 15;
   PIOA_ODR |= 1 << 7 | 1 << 8 | 1 << 9 | 1 << 14 | 1 << 15;
   ctl_set_isr(2, 1, CTL_ISR_TRIGGER_FIXED, JOY_ISR, 0);
   PIOA_IER |= 1 << 15;
   PIOA_IMR |= 1 << 15;
   ctl_unmask_isr(2);
   //int TRYB_GRY = 0;
   int KURSOR_X = 70, KURSOR_Y, KURSOR_X_OLD = 0;
   int SNAKE_X = 70, SNAKE_Y = 70;
   int SNAKE_VEL_X = 0, SNAKE_VEL_Y = 0;
   int PUNKT_X = 50, PUNKT_Y = 50, LICZBA_PUNKTOW = 0;
   char SCORE[10];
   InitLCD();
   Backlight(1);
   LCDClearScreen();
   int test_gra_y = 0;
   //LCDSetRect(40, 10, 115, 115, 0, 0xFFF);
   wprowadzPINxRazy();
   ctl_global_interrupts_enable();
   while (1) {

       if (TRYB_GRY == 0) { 
          SNAKE_X = 70;
          SNAKE_Y = 70;
          LICZBA_PUNKTOW = 0;
          SNAKE_VEL_X = 0;
          SNAKE_VEL_Y = 0;
          //LCDClearScreen();
          LCDSetCircle(KURSOR_X_OLD, 15, 5, 0x000);
          LCDPutStr("SZYBKOSC GRY", 20, 25, 2, 0xFFF, 0x000);
          LCDPutStr("GRAJ ", 40, 25, 2, 0xFFF, 0x000);
          LCDPutStr("RESET PINU", 60, 25, 2, 0xFFF, 0x000);
          LCDSetCircle(KURSOR_X, 15, 5, 0xFFF);
          if ((PIOA_PDSR & 1 << 9) == 0) { //up
             KURSOR_X_OLD = KURSOR_X;
             KURSOR_X = KURSOR_X - 20;
             while((PIOA_PDSR & 1 << 9) == 0) {}
           } else if ((PIOA_PDSR & 1 << 8) == 0) { // down
             KURSOR_X_OLD = KURSOR_X;
             KURSOR_X = KURSOR_X + 20;
             while((PIOA_PDSR & 1 << 8) == 0) {}
           } else if ((PIOA_PDSR & 1 << 7) == 0) { // left
             while((PIOA_PDSR & 1 << 7) == 0) {}
           } else if ((PIOA_PDSR & 1 << 14) == 0) { // riht
             while((PIOA_PDSR & 1 << 14) == 0) {}
           } else if ((PIOA_PDSR & 1 << 15) == 0) { // push
             if (TRYB_GRY == 0 & KURSOR_X >10 & KURSOR_X < 35) {
              TRYB_GRY = 1;
              LCDClearScreen();
             } else if (TRYB_GRY == 0 & KURSOR_X >35 & KURSOR_X < 55) {
                TRYB_GRY = 2;
                LCDClearScreen();
             } else if (TRYB_GRY == 0 & KURSOR_X >55 & KURSOR_X < 75) {
              TRYB_GRY = 3;
              LCDClearScreen();
             } 

             while((PIOA_PDSR & 1 << 15) == 0) {}
           }
           
       } else if (TRYB_GRY == 1) {
         //LCDClearScreen();
         
	 LCDPutStr("SZYBKOSC GRY", 20, 25, 2, 0xFFF, 0x000);
         LCDSetCircle(KURSOR_X_OLD, 15, 5, 0x000);
	 LCDSetCircle(KURSOR_X, 15, 5, 0xFFF);
	 LCDPutStr("1s", 40, 25, 2, 0xFFF, 0x000);
	 LCDPutStr("0.5s", 60, 25, 2, 0xFFF, 0x000);
	 LCDPutStr("0.3s", 80, 25, 2, 0xFFF, 0x000);
	 
	 if ((PIOA_PDSR & 1 << 9) == 0) { //up
	   KURSOR_X_OLD = KURSOR_X;
           KURSOR_X = KURSOR_X - 20;
	   while((PIOA_PDSR & 1 << 9) == 0) {}
	 } else if ((PIOA_PDSR & 1 << 8) == 0) { // down
	   KURSOR_X_OLD = KURSOR_X;
           KURSOR_X = KURSOR_X + 20;
	   while((PIOA_PDSR & 1 << 8) == 0) {}
	 } else if ((PIOA_PDSR & 1 << 7) == 0) { // left
	   while((PIOA_PDSR & 1 << 7) == 0) {}
	 } else if ((PIOA_PDSR & 1 << 14) == 0) { // riht
	   while((PIOA_PDSR & 1 << 14) == 0) {}
	 } else if ((PIOA_PDSR & 1 << 15) == 0) { // push
	   if (TRYB_GRY == 1 && KURSOR_X > 35 && KURSOR_X < 55) {
	     TC0_RC = 46812;
             LCDClearScreen();
	     KURSOR_X = 70;
	     TRYB_GRY = 0;
	   } else if (TRYB_GRY == 1 && KURSOR_X >55 && KURSOR_X < 75) {
	     TC0_RC = 23406;
             LCDClearScreen();
	     KURSOR_X = 70;
	     TRYB_GRY = 0;
	   } else if (TRYB_GRY == 1 && KURSOR_X >75 && KURSOR_X < 95) {
	     TC0_RC = 15506;
             LCDClearScreen();
	     KURSOR_X = 70;
	     TRYB_GRY = 0;
	   } 
	   
	   while((PIOA_PDSR & 1 << 15) == 0) {}
	 }
       } else if (TRYB_GRY == 2) {
          if (timer_dla_gry == 1) {
            /* test
            LCDPutStr("1", test_gra_y, 25, 2, 0xFFF, 0x000);
            test_gra_y = test_gra_y + 5;
            timer_dla_gry = 0;
            */
           //CDClearScreen();
	   //LCDSetRect(8, 8, 120, 120, 0, 0xFFF);
	   
	   LCDSetCircle(SNAKE_X, SNAKE_Y, 4, 0x000);
	   LCDSetRect(30, 8, 120, 120, 0x000, 0xFFF);
	   if ((PIOA_PDSR & 1 << 9) == 0) { //up
	     SNAKE_VEL_X = -1;
	     SNAKE_VEL_Y = 0;
             check_isr = 1;
	     while((PIOA_PDSR & 1 << 9) == 0) {}
	   } else if ((PIOA_PDSR & 1 << 8) == 0) { // down
	     SNAKE_VEL_X = 1;
	     SNAKE_VEL_Y = 0;
             check_isr = 1;
	     while((PIOA_PDSR & 1 << 8) == 0) {}
	   } else if ((PIOA_PDSR & 1 << 7) == 0) { // left
	     SNAKE_VEL_Y = -1;
	     SNAKE_VEL_X = 0;
             check_isr = 1;
	     while((PIOA_PDSR & 1 << 7) == 0) {}
	   } else if ((PIOA_PDSR & 1 << 14) == 0) { // riht
	     SNAKE_VEL_Y = 1;
	     SNAKE_VEL_X = 0;
             check_isr = 1;
	     while((PIOA_PDSR & 1 << 14) == 0) {}
	   } else if ((PIOA_PDSR & 1 << 15) == 0) { // push
	     
	     while((PIOA_PDSR & 1 << 15) == 0) {}
	   } 
	   
	   SNAKE_X = SNAKE_X + SNAKE_VEL_X;
	   SNAKE_Y = SNAKE_Y + SNAKE_VEL_Y;
	   LCDSetCircle(SNAKE_X, SNAKE_Y, 4, 0xFFF);
	   if ((PUNKT_X >= SNAKE_X - 5 && PUNKT_X <= SNAKE_X + 5) && (PUNKT_Y >= SNAKE_Y - 5 && PUNKT_Y <= SNAKE_Y + 5)) {
	     LCDSetCircle(PUNKT_X, PUNKT_Y, 4, 0x000);
             PUNKT_X = rand() % 86 + 35;
	     PUNKT_Y = rand() % 95 + 20;
	     LICZBA_PUNKTOW = LICZBA_PUNKTOW + 1;
	   }
	   
	   sprintf(SCORE, "PUNKTY: %d", LICZBA_PUNKTOW);
	   LCDPutStr(SCORE, 8, 25, 2, 0xFFF, 0x000);

	   LCDSetCircle(PUNKT_X, PUNKT_Y, 4, 0x0F0);
	   if (SNAKE_X - 5 <= 30 || SNAKE_X + 5 >= 120) {
	     TRYB_GRY = 0;
	     KURSOR_X = 70;
	     SNAKE_X = 70;
	     SNAKE_Y = 70;
	     LICZBA_PUNKTOW = 0;
	     SNAKE_VEL_X = 0;
	     SNAKE_VEL_Y = 0;
             send_string("\n\r");
             send_string(SCORE);
             LCDClearScreen();
	     
	     
	   }
	   if (SNAKE_Y - 5 <= 8 || SNAKE_Y + 5 >= 120) {
	     TRYB_GRY = 0;
	     KURSOR_X = 70;
	     SNAKE_X = 70;
	     SNAKE_Y = 70;
	     LICZBA_PUNKTOW = 0;
	     SNAKE_VEL_X = 0;
	     SNAKE_VEL_Y = 0;
             send_string("\n\r");
             send_string(SCORE);
             LCDClearScreen();
	   }
	   
	   timer_dla_gry = 0;
          
          }
       } else if (TRYB_GRY == 3) {
          LCDClearScreen();
          resetPIN();
          wprowadzPINxRazy();
       }

       /*
       if ((PIOA_PDSR & 1 << 9) == 0) { //up
         KURSOR_X = KURSOR_X - 20;
         while((PIOA_PDSR & 1 << 9) == 0) {}
       } else if ((PIOA_PDSR & 1 << 8) == 0) { // down
         KURSOR_X = KURSOR_X + 20;
         while((PIOA_PDSR & 1 << 8) == 0) {}
       } else if ((PIOA_PDSR & 1 << 7) == 0) { // left
         while((PIOA_PDSR & 1 << 7) == 0) {}
       } else if ((PIOA_PDSR & 1 << 14) == 0) { // riht
         while((PIOA_PDSR & 1 << 14) == 0) {}
       } else if ((PIOA_PDSR & 1 << 15) == 0) { // push
         if (TRYB_GRY == 0 & KURSOR_X >10 & KURSOR_X < 35) {
          TRYB_GRY = 1;
         }
        
         while((PIOA_PDSR & 1 << 15) == 0) {}
       } 
       */
       ADC_CR = 1 << 1;
       while((ADC_SR & ADC_SR_EOC6) == 0) {}
       trim_val = ADC_CDR6;
       if (trim_val > 1023) trim_val = 1023;
       PWM_CDTY1 =trim_val*3;

   }
   return 0;
}