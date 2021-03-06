# CoolMeal
People counting Project based on i-cube-lrwan stack (STM32L072/82 - LoRaWAN EU868 ) and STSW-IMG010 stack (ToF VL53L1X)


This project is the embedded part who allows you to get a pathtrack did by a people for more information go to https://www.st.com/en/embedded-software/stsw-img010.html

## Hardware:
Based on grasshopper lorawan development board powered by STM32L082 ==> https://www.tindie.com/products/tleracorp/grasshopper-loralorawan-development-board/
**********************************************************************************************************************************
To be compatible with B-L072Z-LRWAN1 (Discovery-Board) you have to change line 115 & 116 to the right mapping of TCXO.

#define RADIO_TCXO_VCC_PORT                       GPIOA
#define RADIO_TCXO_VCC_PIN                        GPIO_PIN_12
**********************************************************************************************************************************

VL53L1X (satel) ==> https://www.st.com/en/ecosystems/x-nucleo-53l1a1.html

## Software:
i-cube-lrwan V1.1.5 for Keil MDK-ARM (with Time of Flight API)

Follows this requirement to increase the code size limitation compilation from 32KB to 256 KB.
https://www2.keil.com/stmicroelectronics-stm32/mdk


Mapping grasshoper:

                                               -------------------
                                               |                 |
                    ----------                 |                 |
                    |        |    INTERRUPT    |                 |
                    |        |---------------->| D5(PB2)         |--> LoRaWAN RF (EU868)
                    |  ToF   |    XSHUTDOWN    |                 |
                    |        |<----------------|AREF             |
                    |        |      3V3        |                 |
                    |        |<----------------|                 |    VIN
                    |        |      GND        |                 |<--------
                    |        |<----------------|                 |    GND
                    |        |      I2C_SCL    |                 |<--------
                    |        |<--------------->|PB8              |
                    |        |      I2C_SDA    |                 |         
                    |        |<--------------->|PB9              |
                    ----------                 |                 |
                                               |  Grassehopper   |
                                               -------------------
                                                                
                                               

Go to \CoolMeal\i-cube-lrwan_V1.1.5\STM32CubeExpansion_LRWAN_V1.1.5\Projects\Multi\Applications\LoRa\End_Node_V2.1\MDK-ARM\B-L072Z-LRWAN1\lora.uvprojx

 Add your Device EUI and lorawan connection key (api key & app key) in commissioning.h file.
 Compile your project.
 Plugin STM32L0 board and toggle the RESET button while holding down the BOOT button then from STM32CubeProgrammer load the hexfile.
 
 ## How does it works:
 
 ### Pathrack Management 
 The spads of Tof are divided in two Region of Interest (Front zone and Back zone) it measures alternately each Region, from that we can determine a pathtrack and increment or decrement a counter then grasshopper will send a lorawan frame (payload) thanks to a timer event.
 
![image](https://drive.google.com/uc?export=view&id=1S3OefQd81Le0X-aaqf3QF1A209wH5QDk)

* In the first case the Person cross respectively the **Front Zone**, the **Interzone** then the **Back Zone**. 
* In the other case the Person cross respectively the **Back Zone**, the **Interzone** then The **Front Zone**.

The Pathtrack Buffer will be filled regarding the direction of the Person : 

**[0,2,3,1,0]** or **[0,1,3,2,0]**
 
 ### Sending Lora Frame Management

To respect the Lora time on air limitation, The Maximum Frame sent by the Sensor per Hour is 720 with a SF7. 

The Code implements 2 Timer : 

* TXTimerEvent : This one is a counter of 5s that allow to keep the interval between 2 frame above 5s duration
* LedTimerEvent : This is a counter of 10 minutes that allow to send a frame if no event occured for 10 minutes in order to inform the LoraServer that he is still Alive.

The Device is sending frame only if The people Counter is not 0. It means that if someone cross all the Zones and no event occurred for more than 5s and less than 10min, A frame will be instantly sent to the LoraServer
But in the case of no one crossed all the zone since 10 minutes, The Device will sent an alive frame Anyway to inform that he is still working. 

 It's compatible with the Things Networks (https://www.thethingsnetwork.org/) or Chirpstack server (https://www.chirpstack.io/) or others lorawan networks. 

