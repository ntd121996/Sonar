/*
###################################
#### Do Khoang Cach Sieu Am Sonar##
### writen by : NTD    ############
*/
#include<iostm8s003F3.h>
#include<intrinsics.h>
#define Trig            PC_ODR_ODR4
#define Echo            PC_IDR_IDR3
#define Led_xanh        PD_ODR_ODR4
float range=0;
unsigned int  dem=0;

void Init_all();
void truyen_song();
void delay_us(unsigned int time);
void delay_ms(unsigned int time);
void Init_all()
{
  /*Cau Hinh Clock*/
  CLK_ICKR_LSIEN                = 1;          
  CLK_ECKR_HSEEN                = 0;
  CLK_CKDIVR_HSIDIV             = 0x00;             
  CLK_SWCR_SWEN                 = 1;        
  CLK_SWR                       = 0xE1;
  __disable_interrupt();
  /*Cau Hinh Chan Trigger*/
  PC_DDR_DDR4                   =1;
  PC_CR1_C14                    =1;
  PC_ODR_ODR4                   =0;
  /* Cau Hinh Led Xanh   */
  PD_DDR_DDR4                   =1;
  PD_CR1_C14                    =1;
  PD_ODR_ODR4                   =1;//1 la tat
  /*Cau Hinh Chan Echo*/
  PC_DDR_DDR3                   =0; //input
  PC_CR1_C13                    =1; //1:push pull
  PC_CR2_C23                    =1; //cho phep ngat
 
  /*Cau Hinh Timmer */
  TIM2_PSCR                     = 4;              
  TIM2_ARRH                     = (12>>8);        
  TIM2_ARRL                     = (12&0xFF);
  TIM2_CR1_ARPE                 = 1;          
  TIM2_IER_UIE                  = 1;         
  TIM2_CR1_CEN                  = 0; 
  
  /*Cau Hinh Ngat Ngoai*/
  EXTI_CR1_PCIS                 =2;
    __enable_interrupt(); 
 
  
  
}
/*Ham Con*/
void delay_us(unsigned int time)
{
  while(time--)
  {
    asm("NOP");// ton 2 chu ki may.
    asm("NOP");
    asm("NOP");
    asm("NOP");
    asm("NOP");
    asm("NOP");
    asm("NOP");
    asm("NOP");
  }
}

//------------------------
void delay_ms(unsigned int time)
{
  while(time--)
    delay_us(1000);
}

//-------------------------
void truyen_song()
{
  Trig=0;
  delay_us(2);
  Trig=1;
  TIM2_CR1_CEN  = 1;
  delay_us(12);
}
//____MAIN
main()
{
  Init_all();
 
  while(1)
  { 
      truyen_song();
      delay_ms(50);
     
     
  }
  
}

/*Ham phuc vu ngat TIM2*/
#pragma vector = TIM2_OVR_UIF_vector
__interrupt void ngat_TIM(void)
{   
    if(dem>=2500) //qua 30ms reset lai ve 0
    { 
      dem=0;
      Trig=1;
    }
      Trig=0;
      dem++;
      TIM2_SR1_UIF =0; // xoa co ngat
}


/*ham phuc vu ngat ngoai*/
#pragma vector = EXTI2_vector 
__interrupt void ngatngoai(void)
{ 
    while(Echo==1);
    TIM2_CR1_CEN  = 0;
    range  =(0.036f*dem*0.000006f);
    dem    =0;
    Led_xanh=0;
   
  
}


/*_______________THE END____________________*/


