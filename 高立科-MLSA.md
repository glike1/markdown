#include "sys.h"  
#include "delay.h"  
#include "usart.h"  
#include "led.h"  
#include "key.h"  
#include "dht11.h"  
#include "pcf8574.h"  
#include "oled.h"  
#include "rtc.h"  
#include "tpad.h"  
#include "timer.h"         
#include "24cxx.h"  
#include "w25qxx.h"   
int main(void)  
{  
        u8 t=0;                   
    u8 temperature;         
    u8 humidity;  
    u8 temMAX=37;         
    u8 humMAX=69;  
    u8 key=0;  
    u8 y=27;             //这里设置箭头初始坐标，第一次进入设置模式就在调温度  
    u8 SIZE =40;  
    u32 FLASH_SIZE;  
    u8 mod=1;            //mod用来控制温变湿不变或者湿变温不变  
    HAL_Init();                     //初始化HAL库     
    Stm32_Clock_Init(360,25,2,8);   //设置时钟,180Mhz  
    delay_init(180);                //初始化延时函数  
    uart_init(115200);              //初始化USART    
    LED_Init();                     //初始化LED   
    KEY_Init();                     //初始化按键  
    PCF8574_Init();                 //初始化PCF8574  
        OLED_Init();                        //初始化OLED  
        AT24CXX_Init();                   //初始化IIC   
        W25QXX_Init();                    //W25QXX初始化  
        TPAD_Init(2);                     //初始化触摸按键,以90/8=25Mhz频率计数  
        RTC_Init();   //初始化RTC   
        RTC_TimeTypeDef RTC_TimeStruct;  
    RTC_DateTypeDef RTC_DateStruct;  
        u8 tbuf[40];  
        u8 dbuf[40];    
    PCF8574_ReadBit(BEEP_IO);       //由于DHT11和PCF8574的中断引脚共用一个IO，  
                                    //所以在初始化DHT11之前要先读取一次PCF8574的任意一个IO，  
                                    //使其释放掉中断引脚所占用的IO(PB12引脚),否则初始化DS18B20会出问题                                   
    while(1)  
    {                 
        if(t%10==0)//每100ms读取一次  
        {     
                PCF8574_ReadBit(BEEP_IO);   //读取一次PCF8574的任意一个IO，使其释放掉PB12引脚，  
                                    //否则读取DHT11可能会出问题   
        DHT11_Read_Data(&temperature,&humidity);        //读取温湿度值       
                    if(humidity>humMAX||temperature>temMAX)  
                    {     
                    if(humidity>humMAX)  
                    {  
                            OLED_Clear();  
                            OLED_ShowGBK4(0,0,0,16,1);       //湿度大于阈值警告  
                            OLED_ShowGBK4(15,0,1,16,1);  
                            OLED_ShowGBK4(30,0,2,16,1);  
                            OLED_ShowGBK4(45,0,3,16,1);  
                            OLED_ShowGBK4(60,0,4,16,1);  
                            OLED_ShowGBK4(75,0,5,16,1);  
                            OLED_ShowGBK4(90,0,6,16,1);  
                            OLED_ShowChar(68,45,'>',16,1);  
                            OLED_ShowNum(78,45,humMAX,2,16);              
                            OLED_ShowChar(98,45,'%',16,1);  
                            OLED_Refresh_Gram();        //更新显示到OLED   
                    }  
                    if(temperature>temMAX)  
                    {  
                            OLED_Clear();  
                            OLED_ShowGBK4(0,0,7,16,1);       //温度大于阈值警告  
                            OLED_ShowGBK4(15,0,8,16,1);  
                            OLED_ShowGBK4(30,0,9,16,1);  
                            OLED_ShowGBK4(45,0,10,16,1);  
                            OLED_ShowGBK4(60,0,11,16,1);  
                            OLED_ShowGBK4(75,0,12,16,1);  
                            OLED_ShowGBK4(90,0,13,16,1);  
                            OLED_ShowChar(76,27,'>',16,1);  
                            OLED_ShowNum(90,27,temMAX,2,16);              
                        OLED_ShowGBK5(105,27,0,16,1);  
                            OLED_Refresh_Gram();        //更新显示到OLED   
                    }         
    
            if(humidity>humMAX&&temperature>temMAX)  
                    {  
                            OLED_Clear();  
                            OLED_ShowGBK4(0,0,14,16,1);      //温度，湿度都大于阈值警告     
                            OLED_ShowGBK4(15,0,15,16,1);  
                            OLED_ShowGBK4(30,0,16,16,1);  
                            OLED_ShowGBK4(45,0,17,16,1);  
                            OLED_ShowGBK4(60,0,18,16,1);  
                            OLED_ShowGBK4(75,0,19,16,1);  
                            OLED_ShowGBK4(90,0,20,16,1);  
                        OLED_ShowGBK4(105,0,21,16,1);  
                            OLED_ShowGBK4(113,0,22,16,1);  
                            OLED_ShowChar(76,27,'>',16,1);  
                            OLED_ShowNum(90,27,temMAX,2,16);          
                        OLED_ShowGBK5(105,27,0,16,1);  
                            OLED_ShowChar(68,45,'>',16,1);  
                            OLED_ShowNum(78,45,humMAX,2,16);                  
                            OLED_ShowChar(98,45,'%',16,1);  
                            OLED_Refresh_Gram();        //更新显示到OLED   
                    }    
                }  
        
                else{                                         //显示日期和时间  
                OLED_Clear();  
                HAL_RTC_GetTime(&RTC_Handler,&RTC_TimeStruct,RTC_FORMAT_BIN);  
                sprintf((char*)tbuf,"%02d:%02d:%02d",RTC_TimeStruct.Hours,RTC_TimeStruct.Minutes,RTC_TimeStruct.Seconds);   
            HAL_RTC_GetDate(&RTC_Handler,&RTC_DateStruct,RTC_FORMAT_BIN);  
                sprintf((char*)dbuf,"20%02d-%02d-%02d",RTC_DateStruct.Year,RTC_DateStruct.Month,RTC_DateStruct.Date);   
                OLED_ShowString(75,0,tbuf,12);    
                OLED_ShowString(0,0,dbuf,12);  
                OLED_Refresh_Gram();        //更新显示到OLED   
                }  
                OLED_ShowGBK2(0,27,0,16,1);  
                OLED_ShowGBK2(15,27,1,16,1);  
                OLED_ShowGBK2(30,27,2,16,1);  
                OLED_ShowGBK1(0,45,0,16,1);  
                OLED_ShowGBK1(15,45,1,16,1);  
                OLED_ShowGBK1(30,45,2,16,1);  
                OLED_ShowNum(40,27,temperature,2,16);       //显示温度    
                OLED_ShowGBK5(58,27,0,16,1);  
            OLED_ShowNum(40,45,humidity,2,16);          //显示湿度    
                OLED_ShowChar(58,45,'%',16,1);  
            OLED_Refresh_Gram();        //更新显示到OLED  
                    
            }  
                        //tpad试验  
        if(TPAD_Scan(0))    //成功捕获到了一次上升沿(此函数执行时间至少15ms)  
        {  
                OLED_Clear();  
        while(1){  
                OLED_ShowGBK2(0,27,0,16,1);  
                OLED_ShowGBK2(15,27,1,16,1);  
                OLED_ShowGBK2(30,27,2,16,1);  
                OLED_ShowGBK1(0,45,0,16,1);  
                OLED_ShowGBK1(15,45,1,16,1);  
                OLED_ShowGBK1(30,45,2,16,1);  
                OLED_ShowNum(40,27,temMAX,2,16);        //显示温度阈值  
                OLED_ShowGBK5(58,27,0,16,1);  
                OLED_ShowNum(40,45,humMAX,2,16);            //显示湿度阈值  
                OLED_ShowChar(58,45,'%',16,1);  
                OLED_ShowGBK3(0,0,0,16,1);  
                OLED_ShowGBK3(15,0,1,16,1);  
                OLED_ShowGBK3(30,0,2,16,1);  
                OLED_ShowGBK3(45,0,3,16,1);  
                OLED_ShowGBK3(60,0,4,16,1);  
                OLED_ShowGBK3(75,0,5,16,1);  
                OLED_ShowGBK3(90,0,6,16,1);  
                OLED_ShowGBK3(105,0,7,16,1);  
                OLED_ShowGBK3(120,0,8,16,1);  
                OLED_ShowGBK6(80,y,0,16,1);     // 显示箭头  
        key=KEY_Scan(0); //得到键值  
        if(key)  
        {  
                    switch(key)  
            {  
                case WKUP_PRES:        //按上键调温度阈值  
                                {   
                                    mod=1;  
                                    y=27;  
                                }  
                break;  
            
                case KEY1_PRES:         //按下键调湿度阈值  
                {    
                                    mod=0;  
                                    y=45;  
                            }   
                                break;  
                                case KEY2_PRES:         //按左键调数值减小  
                {    
                                    if(mod){temMAX--; AT24CXX_WriteOneByte(0,temMAX);}   //IIC读写  
                                    else{humMAX--; AT24CXX_WriteOneByte(16,humMAX);}  
                            }   
                                break;  
                case KEY0_PRES:         //按右键调数值增大  
                {          
                                    if(mod){temMAX++; SPI5_ReadWriteByte(temMAX);}      //SPI读写  
                                    else{humMAX++; SPI5_ReadWriteByte(humMAX);}  
                            }   
                                break;  
            }  
                        OLED_Clear();  
            }else delay_ms(10);  
                    
                OLED_Refresh_Gram();        //更新显示到OLED   
        if(TPAD_Scan(0)) break;    //这边再捕捉一次上升沿然后退出设置界面  
                    }  
        }  
                
        delay_ms(10);              
                //tpad试验  
            
            
        delay_ms(10);  
        t++;  
        if(t==20)  
        {  
            t=0;  
            LED0=!LED0;  
        }  
    }                     
}  

其中，分别使用IIC和SPI读写数据：
                                    if(mod){temMAX--; AT24CXX_WriteOneByte(0,temMAX);}   //IIC读写  
                                    else{humMAX--; AT24CXX_WriteOneByte(16,humMAX);}  
                            }   
                                break;  
                case KEY0_PRES:         //按右键调数值增大  
                {          
                                    if(mod){temMAX++; SPI5_ReadWriteByte(temMAX);}      //SPI读写  
                                    else{humMAX++; SPI5_ReadWriteByte(humMAX);}  
                            }   
                                break;  
