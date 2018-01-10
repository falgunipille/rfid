#include "mylcd.h"
void ser_init(void);
void tx(unsigned char c);
unsigned char rx(void);
void tx_str(unsigned char *s);
 
void sms(unsigned char *msg);
void gsm_delay(void);

void uart0_init()
{

	//PINSEL0|=0x05;
	PINSEL0=(5<<0);
	U0LCR=0x83;
	
	U0DLL=97;
 U0DLM=0;
 U0LCR=0x03;
 U0TER=(1<<7);
}

void uart_putchar(char c)
{
	U0THR=c;
 while(!(U0LSR&(1<<5))==0);
}
void tx_string(unsigned char *s)
{
 while(*s) {
 uart_putchar(*s++);
 }
}
 
unsigned char rx()
{
	unsigned char a;
 while((U0LSR&(1<<0))==0);
	a=U0RBR;
 return a;
}


	

int main()
{
	
	unsigned int i;
	
 unsigned char id[12],orig[12]="4400078A02CB",flag;
 VPBDIV=0x00;
	
	PINSEL1=0x00;
PINSEL2=0x00;
IO1DIR=(0x07<<22);
IO0DIR=(0x0F<<10);
	uart0_init();
	
	 
	//PINSEL0&=(~(0xF));
	//PINSEL0|=(1<<2)|(1<<0);
	LCD_init();
	LCD_command(0x80);
	LCD_write_string("show ur card");
	LCD_command(0xc0);
	
	for(i=0; i<12; i++) {
		 id[i]=rx();
  //LCD_data(id[i]);
		 //delay(10);
 }

// while(1)
// {
 for(i=0; i<12; i++) {
		if(id[i]==orig[i])
		{			
  //LCD_write_string("t");
			flag=1;
		}
		else
		{
			 //LCD_write_string("f");
			flag=0;
			
		}
		 
 } 
// break;
 //}


if(flag==0)
{
LCD_write_string("f");
	  ser_init(); 
	sms("alert");

}
else
{
 LCD_write_string("t");

}
}



void sms(unsigned char *msg)
{
	tx_str("AT");
	tx(0x0d);
	gsm_delay();
 
	tx_str("AT+CMGF=1");
	tx(0x0d);
	gsm_delay();
 
	tx_str("AT+CMGS=");
	tx('"');
	tx_str("9970500120");
	//while(*num1)
	//	tx(*num1++);
	tx('"');
	tx(0x0d);
	gsm_delay();
 
	while(*msg)
		tx(*msg++);
	tx(0x1a);//ctrl+z
	gsm_delay();
}
 

void gsm_delay()
{
	unsigned long int gsm_del,ff;
	for(gsm_del=0;gsm_del<=500;gsm_del++)
		 for(ff=0;ff<25;ff++);
}
 

void ser_init()
{
	
	
	VPBDIV=0x02;      //PCLK = 30MHz
	PINSEL0=(5<<16);
	U1LCR=0x83;
	U1DLL=195;
	U1DLM=0;
	U1LCR=0x03;
	U1TER=(1<<7);
}


void tx(unsigned char c)
{
	U1THR=c;
	while((U1LSR&(1<<5))==0);
}
 
void tx_str(unsigned char *s)
{
	while(*s)
	{
		tx(*s++);
	}
}
 
unsigned char rx_gsm()
{
	while((U1LSR&(1<<0))==0);
	return U1RBR;
}


	


