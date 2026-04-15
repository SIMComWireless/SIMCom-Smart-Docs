# SIM866X_Thermal Introduction and Debug User Guide

## **version history**

| **versions**|**date**|**author**|**remark**|
| :------- | :-------- | :------- | :------- |
| 1.00     |Zhang bei|2026.3.19| The first version |

## 1 Introduction

Read this document for quick Thermal configuration and debugging tips.

## 2 Thermal Configuration and Commissioning

Thermal for SIM8666 module is a framework model defined by kernel developers that supports controlling system temperature according to a specified governor to prevent overheating of the chip.  

Thermal framework is built by governor,core,cooling device and sensor driver.

Thermal governor:
Used to determine whether the cooling device needs to be reduced in frequency and to what extent. The Linux 4.4 kernel currently includes the following types of governors:

Power_allocator:
PID (proportional-integral-derivative) control is introduced to dynamically allocate power to each cooling device according to the current temperature, and convert power into frequency, so as to achieve the effect of limiting frequency according to temperature.

step_wise: 
cooling device steps down according to the current temperature. fair share: cooling devices with more frequency levels have priority in frequency reduction. userspace: unlimited frequency.

Thermal core: 
encapsulates and abstracts thermal governments and thermal drivers, and defines clear interfaces. Thermal sensor driver: sensor driver for temperature acquisition, such as tsadc.

Thermal cooling device: 
heat source or cooling device, such as CPU, GPU, DDR, etc.

###  2.1 Thermal Configuration

tsadc is used as a thermal sensor in temperature control to obtain temperature, which usually needs to be configured in the number of devices.

Tsadc is configured in the number of devices as follows, and the path of the related device tree is: RK356X_A11/kernel/arch/arm64/boot/dts/rockchip/rk3568.dtsi

```
tsadc: tsadc@fe710000 {
	compatible = "rockchip,rk3568-tsadc";
	reg = <0x0 0xfe710000 0x0 0x100>; /* Register base address and total length of register addresses */
	interrupts = <GIC_SPI 115 IRQ_TYPE_LEVEL_HIGH>; /* Interrupt number and interrupt triggering method */
	rockchip,grf = <&grf>;	/* Referencing the grf module, some platforms require it. */
	clocks = <&cru CLK_TSADC>, <&cru PCLK_TSADC>;	/* Work clock and configuration clock */
	clock-names = "tsadc", "apb_pclk";
	assigned-clocks = <&cru CLK_TSADC_TSEN>, <&cru CLK_TSADC>;  /* Working clock */
	assigned-clock-rates = <17000000>, <700000>;
	resets = <&cru SRST_TSADC>, <&cru SRST_P_TSADC>,	/* Reset signal */
		 <&cru SRST_TSADCPHY>;
	reset-names = "tsadc", "tsadc-apb", "tsadc-phy";
    /*
    * The "thermal sensor" identifier indicates that the tsadc can be used as a thermal sensor.
* It also specifies the number of parameters that need to be provided when referencing the tsadc node.
* If there is only one tsadc in the SoC, set it to 0. If there are more than one, set it to 1.
    */
	#thermal-sensor-cells = <1>;
	rockchip,hw-tshut-temp = <120000>;	/* Over-temperature restart threshold: 120 degrees Celsius */
	rockchip,hw-tshut-mode = <0>; /* Shutdown mode 0: CRU 1: GPIO Select whether to reset through CRU or GPIO */
	rockchip,hw-tshut-polarity = <0>; /* tshut polarity 0:LOW 1:HIGH Low-level reset or high-level reset */
	pinctrl-names = "gpio", "otpout";	/* The configuration of the TSADC output pins supports two modes: GPIO and OTPOUT. */
	pinctrl-0 = <&tsadc_gpio_func>;
	pinctrl-1 = <&tsadc_shutorg>;
	status = "disabled";
};
```


IO port configuration in the device tree is as follows, the relevant device tree path: RK356X_A11/kernel/arch/arm64/boot/dts/rockchip/rk3568-pinctrl.dtsi

```
pinctrl: pinctrl {
......
	tsadc {
	......
		/omit-if-no-ref/ /* Configure to over temperature protection mode */
		tsadc_shutorg: tsadc-shutorg {
			rockchip,pins =
				/* tsadc_shutorg */
				<0 RK_PA1 2 &pcfg_pull_none>;
		};
	};
......
	gpio-func { /* Confgure to GPIO mode */
		/omit-if-no-ref/
		tsadc_gpio_func: tsadc-gpio-func {
			rockchip,pins =
				<0 RK_PA1 RK_FUNC_GPIO &pcfg_pull_none>;
		};
	};
};
```

The configuration to enable Tsadc in DTSI is as follows, the relevant device tree path: RK356X_A11/kernel/arch/arm64/boot/dts/rockchip/rk3568-pinctrl.dtsi

```
&tsadc {
	status = "okay";
};
```

### 2.2 Thermal Drive Codes

Governor-related codes:

```
RK356X_A11/kernel/drivers/thermal/power_allocator.c 	/* Power Allocator Temperature Control Strategy*/
RK356X_A11/kernel/drivers/thermal/step_wise.c			/* Stepwise temperature control strategy */
RK356X_A11/kernel/drivers/thermal/fair_share.c 			/* fair share temperature control strategy */
RK356X_A11/kernel/drivers/thermal/user_space.c 			/* userspace temperature control strategy */
```

Cooling device related codes:

```
RK356X_A11/kernel/drivers/thermal/devfreq_cooling.c
RK356X_A11/kernel/drivers/thermal/cpu_cooling.c
```

Core related code:

```
RK356X_A11/kernel/drivers/thermal/thermal_core.c
```

Driver related code:

```
RK356X_A11/kernel/drivers/thermal/rockchip_thermal.c 	/* The tsadc driver for other platforms except RK3368 */
RK356X_A11/kernel/drivers/thermal/rk3368_thermal.c 		/* RK3368 platform TSADC driver */
```

### 2.3 Configuration methods

Usually, the Thermal function has been enabled, the specific configuration is as follows, the relevant configuration file path: RK356X_A11/kernel/arch/arm64/configs/rockchip_defconfig

```
CONFIG_THERMAL=y
CONFIG_THERMAL_WRITABLE_TRIPS=y
CONFIG_THERMAL_DEFAULT_GOV_POWER_ALLOCATOR=y
CONFIG_THERMAL_GOV_FAIR_SHARE=y
CONFIG_THERMAL_GOV_STEP_WISE=y
CONFIG_THERMAL_GOV_USER_SPACE=y
CONFIG_CPU_THERMAL=y
CONFIG_DEVFREQ_THERMAL=y
CONFIG_ROCKCHIP_THERMAL=y
CONFIG_RK3368_THERMAL=y
```

### 2.4 Power allocator policy configuration

Power allocator temperature control strategy introduces PID (proportional-integral-derivative) control, according to the current temperature, dynamically distribute power to each cooling device, when the temperature is low, the power that can be distributed is relatively large, that is, the frequency that can be operated is high, with the temperature rising, the power that can be distributed gradually decreases, and the frequency that can be operated gradually decreases, so as to achieve the frequency limit according to the temperature.  

#### 2.4.1 CPU配置

CPU is used as cooling device in temperature control, and the node needs to contain #cooling-cells and dynamic-power-coefficient attributes. Related device tree path: RK356X_A11/kernel/arch/arm64/boot/dts/rockchip/rk3568.dtsi

```
cpu_l0: cpu@0 {
	device_type = "cpu";
	compatible = "arm,cortex-a53", "arm,armv8";
	reg = <0x0 0x0>;
	enable-method = "psci";
	clocks = <&cru ARMCLKL>;
	next-level-cache = <&cluster0_l2>;
	operating-points-v2 = <&cluster0_opp>;
	sched-energy-costs = <&RK3368_CPU_COST_0 &RK3368_CLUSTER_COST_0>;
	/* cooling device，indicates that this device can be used as a cooling device */
	#cooling-cells = <2>; /* min followed by max */ 
	dynamic-power-coefficient = <149>;	/* Parameters used for calculating dynamic power consumption */
};
......
cpu_b0: cpu@100 {
	device_type = "cpu";
	compatible = "arm,cortex-a53", "arm,armv8";
	reg = <0x0 0x100>;
	enable-method = "psci";
	clocks = <&cru ARMCLKB>;
	next-level-cache = <&cluster1_l2>;
	operating-points-v2 = <&cluster1_opp>;
	sched-energy-costs = <&RK3368_CPU_COST_1 &RK3368_CLUSTER_COST_1>;
	/* cooling device indicates that this device can be used as a cooling device */
	#cooling-cells = <2>; /* min followed by max */
	dynamic-power-coefficient = <160>;	/* Parameters used for calculating dynamic power consumption */
};
```

#### 2.4.2 GPU Configuration

GPU is used as cooling device in temperature control, and the node needs to contain #cooling-cells attribute and power_model child node.  Related device tree path: RK356X_A11/kernel/arch/arm64/boot/dts/rockchip/rk3568.dtsi

```
gpu: rogue-g6110@ffa30000 {
	compatible = "arm,rogue-G6110", "arm,rk3368-gpu";
	reg = <0x0 0xffa30000 0x0 0x10000>;
	clocks =
		<&cru SCLK_GPU_CORE>,
		<&cru ACLK_GPU_MEM>,
		<&cru ACLK_GPU_CFG>;
	clock-names =
		"sclk_gpu_core",
		"aclk_gpu_mem",
		"aclk_gpu_cfg";
	interrupts = <GIC_SPI 8 IRQ_TYPE_LEVEL_HIGH>;
	interrupt-names = "rogue-g6110-irq";
	power-domains = <&power RK3368_PD_GPU_1>;
	operating-points-v2 = <&gpu_opp_table>;
	/* cooling device indicates that this device can be used as a cooling device */
	#cooling-cells = <2>; /* min followed by max */
	gpu_power_model: power_model {
		compatible = "arm,mali-simple-power-model";
		voltage = <900>;
		frequency = <500>;
		static-power = <300>; 			/* Parameters used for calculating static power consumption */
		dynamic-power = <396>; 			/* Parameters used for calculating static power consumption */
		ts = <32000 4700 (-80) 2>; 		/* Parameters used for calculating static power consumption */
		thermal-zone = "gpu-thermal";	/* Obtain the temperature from gpu-thermal and use it to calculate the static power consumption. */
	};
};
```

#### 2.4.3 Thermal Zone Configuration 

Termal zone node is mainly used to configure parameters related to temperature control policy and generate corresponding user mode interface.  Related device tree path: RK356X_A11/kernel/arch/arm64/boot/dts/rockchip/rk3568.dtsi

```
thermal_zones: thermal-zones {
	/* One node corresponds to one thermal zone and contains parameters related to the temperature control strategy. */
	soc_thermal: soc-thermal {
		/* When the temperature is higher than the value specified by "trip-point-0", the temperature is retrieved every 20 milliseconds. */
		polling-delay-passive = <200>; /* milliseconds */
		/* When the temperature is lower than the value specified by "trip-point-0", the temperature is retrieved every 1000 milliseconds. */
		polling-delay = <200>; /* milliseconds */
		/* When the temperature equals the value specified by "trip-point-1", the energy allocated by the system to the cooling device */
		sustainable-power = <600>; /* milliwatts */
		/* The current thermal zone obtains the temperature through tsadc0. */
		thermal-sensors = <&tsadc 0>;
		/* The "trips" feature includes various temperature thresholds and different temperature control strategies, and the configurations may not be the same. */
		trips {
			/*
			* 温控阀值，超过该值温控策略开始工作，但不一定马上限制频率，
			* power小到一定程度才开始限制频率
			*/
			threshold: trip-point-0 {
				/* 超过70摄氏度，温控策略开始工作，并且70摄氏度也是tsadc触发中断的一个阀值 */
				temperature = <70000>; /* millicelsius */
				/* 温度低于temperature-hysteresis时触发中断，当前未实现，但是框架要求必须填 */
				hysteresis = <2000>; /* millicelsius */
				/* 表示超过该温度值时，使用polling-delay-passive */
				type = "passive";
			};
			/* 温控目标温度，期望通过降频使得芯片不超过该值 */
			target: trip-point-1 {
				/* 期望通过降频使得芯片不超过85摄氏度，并且85摄氏度也是tsadc触发中断的一个阀值 */
				temperature = <80000>; /* millicelsius */
				/* 温度低于temperature-hysteresis时触发中断，当前未实现，但是框架要求必须填 */
				hysteresis = <2000>; /* millicelsius */
				type = "passive"; /* 表示超过该温度值时，使用polling-delay-passive */
			};
			/* 过温保护阀值，如果降频后温度仍然上升，那么超过该值后，让系统重启 */
			soc_crit: soc-crit {
				/* 超过115摄氏度重启，并且115摄氏度也是tsadc触发中断的一个阀值 */
				temperature = <115000>; /* millicelsius */
				/* 温度低于temperature-hysteresis时触发中断，当前未实现，但是框架要求必须填 */
				hysteresis = <2000>; /* millicelsius */
				type = "critical"; /* 表示超过该温度值时，重启 */
			};
		};
		/* cooling device配置节点，每个子节点代表一个cooling device */
		cooling-maps {
			map0 {
				/*
				* 表示在target trip下，该cooling device才起作用，
				* 对于power allocater策略必须填target
				*/
				trip = <&target>;
				/* A53做为cooloing device， THERMAL_NO_LIMIT不起作用，但必须填 */
				cooling-device =
				<&cpu_l0 THERMAL_NO_LIMIT THERMAL_NO_LIMIT>;
				contribution = <1024>; /* 计算功耗时乘以4096/1024倍，用于调整降频顺序和尺度 */
			};
			map1 {
				/*
				* It indicates that this cooling device only comes into effect under the "target trip" condition.
				* For the power allocator strategy, the "target" must be filled in.
				*/
				trip = <&target>;
				/* As a cooling device, A53 does not function properly when using THERMAL_NO_LIMIT, but it must be filled in. */
				cooling-device =
				<&cpu_b0 THERMAL_NO_LIMIT THERMAL_NO_LIMIT>;
				contribution = <1024>; /* When calculating power consumption, multiply by 4096/1024 to adjust the frequency reduction sequence and scale. */
			};
			map2 {
				/*
				* It indicates that this cooling device only comes into effect under the "target trip" condition.
				* For the power allocator strategy, the "target" must be filled in.
				*/
				trip = <&target>;
				/* As a cooling device, the GPU does not respond to THERMAL_NO_LIMIT, but it must be filled in. */
				cooling-device =
				<&gpu THERMAL_NO_LIMIT THERMAL_NO_LIMIT>;
				contribution = <1024>; /* When calculating power consumption, multiply by 4096/1024 to adjust the frequency reduction sequence and scale. */
			};
		};
	};
	/* One node corresponds to one thermal zone and contains parameters related to the temperature control strategy. The current thermal zone is only used to obtain the temperature. */
	gpu_thermal: gpu-thermal {
		/* It only takes effect when the temperature control strategy configuration is included. The framework requires that it must be filled in. */
		polling-delay-passive = <200>; /* milliseconds */
		/* Obtain the temperature every 200 milliseconds. */
		polling-delay = <200>; /* milliseconds */
		/* Currently, the thermal zone obtains the temperature through tsadc1. */
		thermal-sensors = <&tsadc 1>;
	};
};
```

### 2.5 Temperature control parameter adjustment 

Temperature control parameters are related to the chip and generally do not need to be modified.
Some parameters can be adjusted according to the actual situation of the product. For specific adjustment methods, please refer to the chapter "Temperature Control Parameter Adjustment" in Rockchip_Development_Guide_Thermal_CN.pdf, which will not be described here.

#### 2.5.1 Getting the current temperature

You can directly view the temp node under the user mode interface thermal_zone0 or thermal_zone1 directory.  
Enter the following command in the serial port:

```
# Obtain the CPU temperature
cat /sys/class/thermal/thermal_zone0/temp
# Obtain the GPU temperature
cat /sys/class/thermal/thermal_zone1/temp
```

#### 2.5.2 Turn off temperature control

First, switch the temperature control policy to user_space, that is, change the policy node in the user mode interface to user_space; or set the mode to disabled state; then, remove the frequency limit, that is, set the cur_state of all cdev in the user mode interface to 0.  

Enter the following command on the serial port:

```
# Switch the strategy to user_space
echo user_space > /sys/class/thermal/thermal_zone0/policy
# Or set the mode to the "disabled" state
echo disabled > /sys/class/thermal/thermal_zone0/mode
# Then, remove the frequency restrictions.
# How many cdevs are there exactly? Please modify as per the actual situation.
echo 0 > /sys/class/thermal/thermal_zone0/cdev0/cur_state
echo 0 > /sys/class/thermal/thermal_zone0/cdev1/cur_state
echo 0 > /sys/class/thermal/thermal_zone0/cdev2/cur_state
```

## 3 References

Refer to debugging documentation Please refer to:
	RKDocs/common/Thermal/Rockchip_Developer_Guide_Thermal_CN.pdf

