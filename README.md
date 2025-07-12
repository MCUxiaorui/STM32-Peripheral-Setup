# STM32-Peripheral-Setup
Just to make my life simple. I want to avoid re-spending time on setting up peripherals

## Table of Contents
- [RTC](#RTC)
  - [Private typedef](#Private-typedef)
  - [Encapulating function](#Encapulating-function)
- [ADC](#adc-wdma-and-continuous-mode)
  - [ADC Configuration](#adc-configuration)
  - [Start ADC w/DMA](#Starting-ADC-with-DMA)
  - [IRQ Configuration](#DMA-Interrupt-Handler-(if-needed)-in-it.c)
- [TIMER](#Timer-interrupts)
- [USB](#USB)
  - [HID Gamepad](#HID-Gamepad)
  - [Virtual COM Port (VCP)](#Virtual-COM-Port-(VCP))
- [License](#License)
- [Author](#author)

## GPIO_C13 seems to have some weird problem after initialize RTC (F103 Series)

## RTC
### Private typedef
```c
RTC_TimeTypeDef sTime = {0};
RTC_DateTypeDef sDate = {0};

RTC_TimeTypeDef currentTime = {0};
RTC_DateTypeDef currentDate = {0};
```
### Encapulating function
```c
void RTC_Update(uint8_t hour, uint8_t minute, uint8_t second, uint8_t weekDay, uint8_t date, uint8_t month, uint8_t year)
{

    sTime.Hours = hour;
    sTime.Minutes = minute;
    sTime.Seconds = second;
    sTime.DayLightSaving = RTC_DAYLIGHTSAVING_NONE;
    sTime.StoreOperation = RTC_STOREOPERATION_RESET;
    HAL_RTC_SetTime(&hrtc, &sTime, RTC_FORMAT_BIN);

    sDate.WeekDay = weekDay;
    sDate.Date = date;
    sDate.Month = month;
    sDate.Year = year;
    HAL_RTC_SetDate(&hrtc, &sDate, RTC_FORMAT_BIN);
}
```

## ADC w/DMA and Continuous Mode 

### ADC Configuration
- **Select the desired ADC instance** (e.g., `ADC1`).
- **Enable Continuous Conversion Mode**.
- **Enable DMA Peripheral to Memory** (`Half Word`).
- **Set Data Alignment** (e.g., `Right Alignment`).
- **Choose the Sampling Time** according to the requirement.
- **Configure the Scan Mode** if using multiple channels.

### Starting ADC with DMA
```c
HAL_ADC_Start_DMA(ADC_HandleTypeDef* hadc, uint32_t* pData, uint32_t Length);
```

### DMA Interrupt Handler (if needed) in `it.c`
```c
void DMA1_Channel1_IRQHandler(void)
{
  /* USER CODE BEGIN DMA1_Channel1_IRQn 0 */

  /* USER CODE END DMA1_Channel1_IRQn 0 */
  HAL_DMA_IRQHandler(&hdma_adc1);
  /* USER CODE BEGIN DMA1_Channel1_IRQn 1 */

  /* USER CODE END DMA1_Channel1_IRQn 1 */
}
```

## Timer interrupts
### Settings
- **Clock Source** (e.g., `Internal Clock`)
- **Prescaler** (`As required`)
- **Counter Period** (`As required`)
- **NVIC Enable interrupts** (e.g., `Update Interrupt`)
### Start timer after peripheral configuration
```c
  if (HAL_TIM_Base_Start_IT(TIM_HandleTypeDef *htim) != HAL_OK)
  {
    /* Starting Error */
    Error_Handler();
  }
```
### Then implement IRQHandler function
`For example`
```c
void TIM1_UP_IRQHandler(void)
{
  /* USER CODE BEGIN TIM1_UP_IRQn 0 */
  HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
  /* USER CODE END TIM1_UP_IRQn 0 */
  HAL_TIM_IRQHandler(&htim1);
  /* USER CODE BEGIN TIM1_UP_IRQn 1 */

  /* USER CODE END TIM1_UP_IRQn 1 */
}
```

## USB ``Heavy on Flash Usage``
### HID Gamepad
```c
usb_buffer.Xaxis = (uint16_t)(usb_buffer.Xaxis & 0x0FFF);
USBD_HID_SendReport(&hUsbDeviceFS, (uint8_t *) &usb_buffer, 20);
```
### Virtual COM Port (VCP)
- **Setup Buffer first**
```c
char TxBuf[64];
char RxBuf[64];
```
- **Use following function directly**
```c
CDC_Transmit_FS(uint8_t* Buf, uint16_t Len);
CDC_Receive_FS(uint8_t* Buf, uint32_t *Len);
```
## Interesting relationship of Vtemp vs Vref From ADC (``No correlation``)
![image](https://github.com/user-attachments/assets/aebc4160-8b11-4cf0-a8c9-44c2f947a732)

---
### License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

### Author
Created by [Xiaorui Liu](https://github.com/MCUxiaorui)
