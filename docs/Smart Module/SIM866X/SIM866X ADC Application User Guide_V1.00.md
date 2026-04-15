# SIM866X ADC Application User Guide

## **Version History**

| **versions**|**date**    |**author**|**remark**         |
| :----- | :-------- | :----- | :------------- |
| 1.00   |2026.3.7| Wu Wenxiang| The first version  |

## 1 Introduction

Read this document for a quick overview of ADC configuration and debugging tips.
## 2 SARADC pin

SARADC has 8 SARCADC_VIN(0~7) in total,If there is a demand, the corresponding channel can be configured.


## 3 Device tree configuration

The file path of the related device tree is: RK356X_A11/sunsearch/project_sunsearch/SIM8666/kernel/arch/arm64/boot/dts/rockchip/simcom/simcom.dtsi

```c
/* For example, in the EVB, channels 1, 4, 5, 6 and 7 are not in use and can be utilized for other functions. */
/{
...
	simcom_adc: simcom_adc {
		compatible = "simcom,simcom_adc";
		io-channels = <&saradc 0x1>,
						<&saradc 0x4>,
						<&saradc 0x5>,
						<&saradc 0x6>,
						<&saradc 0x7>;
		io-channel-names = "saradc1","saradc4","saradc5","saradc6","saradc7";
	};
};
```

One thing to note is the configuration of saradc reference voltage. Look at the hardware design. The reference voltage of SIM8666 is 1.8V. Use EVB default configuration.
vref-supply = <&vccadc_ref>;
The reference voltage corresponding to the aradc value needs to be set according to the specific hardware environment. The maximum is 1.8V, and the corresponding saradc value is 1024. The voltage and adc value are linear.

## 4 Makefile configuration
Kernel macro configuration path: RK356X_A11/sunsea/project_sunsea/SIM8666/ProjectConfig.mk
Just add the following macro:
CONFIG_SIMCOM_ADC=y

## 5 Driver configuration
Kernel driver path: kernel/drivers/iio/adc/simcom_adc.c
Drive explanation:
	1. Depending on the "iio" framework, you need to initialize struct iio_dev structure. For details, please see rockchip_saradc_probe function when
	indio_dev in, and finally call iio_device_register(indio_dev) to register indio_dev , waiting for the "input" framework to use.

	2. Take "adc-key" as an example, you need to initialize struct input_polled_dev. For details, see
	adc_keys_probe function in drivers/input/keyboard/adc-keys.c, call
	input_register_polled_device(poll_dev); Register poll_dev in the "input" frame.

	3. When using getevent test, assuming adc-key is event0, getevent -s /dev/input/event0 will
	adc_keys_poll -> iio_read_channel_processed -> iio_channel_read ->
	> chan->indio_dev ->info->read_raw(rockchip_saradc_read_raw) ->iio_convert_raw_to_processed_unlocked
	rockchip_saradc_read_raw is an important function, analyze it one by one:
	1. writel_relaxed(8, info->regs + SARADC_DLY_PU_SOC); Sets the interval between power up and start of sampling to 8 sclk cycles.
	2. writel(SARADC_CTRL_POWER_CTRL | (chan->channel & SARADC_CTRL_CHN_MASK) | SARADC_CTRL_IRQ_ENABLE,info->regs + SARADC_CTRL); 
		a) "power up saradc" 
		b) Set sampling channel
		c) Enable interrupt to start sampling
	3. wait_for_completion_timeout(&info->completion, SARADC_TIMEOUT) Wait for saradc to complete sampling and generate an interrupt.

	4. *val = info->last_val; Store sampled data in val.
	5. Finally, iio_convert_raw_to_processed_unlocked is called to convert the sampled data to the corresponding voltage value.

	Interrupt handling process: rockchip_saradc_isr function:
	1. info->last_val = readl_relaxed(info->regs + SARADC_DATA); Save data for use in step 4 above.
	2. writel_relaxed(0, info-> rees + SARADC_CTRL); clear interrupt, and "power down saradc", close saradc.

	A complete sampling process is rockchip_saradc_read_raw configure saradc, turn saradc on, start sampling, wait for interrupt, clear interrupt in interrupt function, turn saradc off.

The requested URL/iio/adc/simcom_adc.c was not found on this server.
	saradc_vin1	-->  proc/saradc1
	saradc_vin4	-->  proc/saradc4
	saradc_vin5	-->  proc/saradc5
	saradc_vin6	-->  proc/saradc6
	saradc_vin7	-->  proc/saradc7

## 6 SARADC Debug
Results read:
	cat proc/saradcx //Read the adc voltage value of the corresponding channel, the value range is 0~1800 (unit is mv)


Refer to RKDocs/common/SARADC/Rockchip_Developer_Guide_Linux_SARADC_CN.pdf for debugging documentation