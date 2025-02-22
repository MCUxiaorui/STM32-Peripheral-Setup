# STM32-Peripheral-Setup
Just to make my life simple. I want to avoid re-spending time on setting up peripherals

## Table of Contents
- [RTC setup time function](#RTC)
- [STM32 ADC Configuration](#stm32-adc-configuration)
  - [ADC Configuration](#adc-configuration)
  - [DMA Configuration](#dma-configuration)
- [Code Implementation](#code-implementation)
  - [ADC and DMA Initialization](#adc-and-dma-initialization)
  - [DMA Initialization](#dma-initialization)
  - [Starting ADC with DMA](#starting-adc-with-dma)
  - [DMA Interrupt Handler (if needed)](#dma-interrupt-handler-if-needed)
- [Verification](#verification)
- [License](#License)
- [Author](#author)

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
- **Enable ADC peripheral** in **CubeIDE**.
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

### DMA Interrupt Handler (if needed) in it.c
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

## USB-HID Gamepad
### placeholder
```c
usb_buffer.Xaxis = (uint16_t)(usb_buffer.Xaxis & 0x0FFF);
USBD_HID_SendReport(&hUsbDeviceFS, (uint8_t *) &usb_buffer, 20);
```
---
### License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

### Author
Created by [Xiaorui Liu](https://github.com/MCUxiaorui)
