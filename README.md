# mwy
#include "LPC11XX.h"


static uint8_t count = 5;

void GPIOinit()
{
	LPC_GPIO2->DIR = 0xFFF;
	LPC_GPIO2->DATA = 0xFFF;
	LPC_GPIO3->DIR &= ~(1<<0);
	LPC_GPIO3->DIR &= ~(1<<1);
	LPC_GPIO3->IE |= (1<<0);
  LPC_GPIO3->IE |= (1<<1);
	LPC_GPIO3->IS &= ~(1<<0);
	LPC_GPIO3->IS &= ~(1<<1);
	LPC_GPIO3->IEV &= ~(1<<0);
	LPC_GPIO3->IEV &= ~(1<<1);
	NVIC_EnableIRQ(EINT3_IRQn);
}

void CT32B1_Init(uint32_t interval)
{
	LPC_SYSCON->SYSAHBCLKCTRL|=(1<<16);
	LPC_IOCON->R_PIO1_2 &=~(0x07);
	LPC_IOCON->R_PIO1_2 |=0x03;    //将1—2设置为CT32B1-MAT1输出

	LPC_SYSCON->SYSAHBCLKCTRL&=~(1<<16);  

  LPC_SYSCON->SYSAHBCLKCTRL|=(1<<10);  //打开定时器计数器1时钟
  LPC_TMR32B1->TCR = 0x02;          //复位TC 
	LPC_TMR32B1->PR =0;               //每个PCLK，TC+1
  LPC_TMR32B1->MCR =0x02<<9;         //当TC与MR3匹配时复位TC   
  LPC_TMR32B1->PWMC = 0x02;             //选择CT32B1-MAT1作为PWM输出
	LPC_TMR32B1->MR1 =interval * count / 10 ;          //CT32B1-MAT1输出脉冲占空比
	LPC_TMR32B1->MR3 =interval ;           //PWM周期
	LPC_TMR32B1->TCR = 0x01;               //启动定时器
}

void PIOINT3_IRQHandler()
{
	if((LPC_GPIO3->MIS & (1<<0)) == (1<<0))
	{		
			count ++;
		  if(count > 10)   count = 10;
		  LPC_TMR32B1->MR1 = SystemCoreClock/1000 * count / 10 ;
			LPC_GPIO3->IC = (1<<0);
	}	

	if((LPC_GPIO3->MIS & (1<<1)) == (1<<1))
	{
			count --;
		  if(count < 2)   count = 2;
		  LPC_TMR32B1->MR1 = SystemCoreClock/1000 * count / 10 ;
			LPC_GPIO3->IC = (1<<1);
		
	}
	
}


int main (){
  GPIOinit ();
  CT32B1_Init(SystemCoreClock/1000);
	while(1);
	



}

:( 2017/5/10 9:53:17
P3的0和1

:( 2017/5/10 9:53:48

