# SIM866X_Low Speed Peripheral (I2C & UART & SPI & PWM & GPIO) Development User Guide

## **Version History**

| **versions**|**date**|**author**|**remark**|
| :------- | :-------- | :------- | :------- |
| 1.00     |2026.3.17| Zhang bei| The first version  |

## 1 Introduction

Read this document for a quick overview of I2C UART SPI GPIO interface configuration and related debugging tips.

## 2 I2C configuration and debugging

SIM8666 module can provide up to 4 groups of I2C interfaces, I2C only supports master mode, the highest rate is 400Kbps, and supports 7-bit and 10-bit addressing modes.   。

**Note: I2C1, I2C2, I2C3 and I2C4 have pull-up resistors of 2.2KΩ inside the module; the IO of I2C1, I2C2 and I2C4 is 1.8V on average, and the IO level of I2C3 is 3.3V.**

###  2.1 I2C configuration

i2c DTS is configured as follows in the number of devices, and the path of the related device tree is RK356X_A11/kernel/arch/arm64/boot/dts/rockchip/rk3568.dtsi

```
i2c1: i2c@fe5a0000 {
	compatible = "rockchip,rk3399-i2c";
	reg = <0x0 0xfe5a0000 0x0 0x1000>;
	clocks = <&cru CLK_I2C1>, <&cru PCLK_I2C1>;
	clock-names = "i2c", "pclk";
	interrupts = <GIC_SPI 47 IRQ_TYPE_LEVEL_HIGH>;
	pinctrl-names = "default";
	pinctrl-0 = <&i2c1_xfer>;
	#address-cells = <1>;
	#size-cells = <0>;
	status = "disabled";
};
```


The file path of the device tree to open i2c is: RK356X_A11/sunsearch/project_sunsearch/SIM8666/kernel/arch/arm64/boot/dts/rockchip/simcom/simcom.dtsi

```
&i2c1 {
	status = "okay";
};

&i2c2{
	status = "okay";
	pinctrl-0 = <&i2c2m1_xfer>;
};

&i2c3{
	status = "okay";
};

// sensors Light & Proximity
&i2c4 {
	status = "okay";
	hym8563: hym8563@51 {
		compatible = "haoyu,hym8563";
		reg = <0x51>;
	};
	gc8034@37 {
		status = "disabled";
	};
};
```

I2C peripheral configurations can be added in simcom.dtsi. where status = "okay"; to enable the corresponding I2C bus or peripheral; status = "disabled"; to disable the corresponding I2C bus or peripheral.
Main driver files:

```
RK356X_A11/kernel/drivers/i2c/busses/i2c-rk3x.c
```

### 2.2 Configuring I2C peripherals

Take gt9xx touch chip as an example to configure I2C peripherals.
Configure I2C peripheral TP device resources in the device tree. The path of the device tree is RK356X_A11/kernel/arch/arm64/boot/dts/rockchip/simcom/tp_mipi.dtsi

```
&gt1x {
    status = "disabled";
};

&i2c1 {
	gt9xx: gt9xx@5d {
		status = "disabled";
		compatible = "goodix,gt9xx";
		reg = <0x5d>;
		pinctrl-names = "default";
		pinctrl-0 = <&tp_gpio>;
		touch-gpio = <&gpio0 RK_PB5 GPIO_ACTIVE_HIGH>;
		reset-gpio = <&gpio0 RK_PB6 GPIO_ACTIVE_HIGH>;
		touchscreen-size-x = <800>;
		touchscreen-size-y = <1280>;
		......
	};
};
```

Register and operate I2C devices in TP driver files, see gt9xx Touch Chip Driver Codes for details.

Driver file path: RK356X_A11/kernel/drivers/input/touchscreen/gt9xx/gt9xx.c

### 2.3 I2C Commissioning

Typically, I2C devices are controlled by kernel drivers. However, all devices on the bus can also be accessed from user mode through the "/dev/i2c-%d" interface, which is described and illustrated in the Documentation/i2c/dev-interface document below the kernel.
You can use I2C tool open source tools for I2C device debugging, you need to download it directly for cross-compilation.

### 2.4 References

Refer to debugging documentation Please refer to:
	RKDocs/common/I2C/Rockchip_Developer_Guide_I2C_CN.pdf

## 3 UART configuration and debugging

SIM8666 has 3 uarts available, of which uart2 is debug port output system debug log; uart5 is used by users;**uart9 is common with CIF_D6 CIF_D7, and only one uart9 and CIF interface can be enabled**.

uart functions are based on the 16550A serial port standard, and the complete module supports the following functions:
	Support 5, 6, 7, 8 bits of data.
	Support 1, 1.5, 2 bits stop bit. Odd parity and even parity are supported, mark parity and space parity are not supported.
	It supports receive FIFO and transmit FIFO, usually 32 bytes or 64 bytes.
	Support up to 4M baud rate, the actual support of baud rate requires chip clock frequency division strategy. Support interrupt transfer mode and DMA transfer mode.
	Support hardware automatic flow control, RTS+CTS.

Note: The UART functions supported in the actual chip shall be described in the UART chapter in the chip manual, and some UART functions will be appropriately tailored.

### 3.1 UART configuration

uart DTS is configured in the number of devices as follows, and the path of the related device tree is RK356X_A11/kernel/arch/arm64/boot/dts/rockchip/rk3568.dtsi

```
uart5: serial@fe690000 {
	compatible = "rockchip,rk3568-uart", "snps,dw-apb-uart";
	reg = <0x0 0xfe690000 0x0 0x100>;
	interrupts = <GIC_SPI 121 IRQ_TYPE_LEVEL_HIGH>;
	clocks = <&cru SCLK_UART5>, <&cru PCLK_UART5>;
	clock-names = "baudclk", "apb_pclk";
	reg-shift = <2>;
	reg-io-width = <4>;
	dmas = <&dmac0 10>, <&dmac0 11>;
	pinctrl-names = "default";
	pinctrl-0 = <&uart5m0_xfer>;
	status = "disabled";
};
```

The file path of the device tree to open uart is: RK356X_A11/sunsea/project_sunsea/SIM8666/kernel/arch/arm64/boot/dts/rockchip/simcom/simcom.dtsi

```
&uart5{
	status = "okay";/* Enable uart5 */
	pinctrl-0 = <&uart5m1_xfer>;/*Config uart5 pin */
};

/* Conflicts with the PDM1_SDI2 pins of Audio */
&uart7{
	status = "disabled";/* Disable uart7 */
	pinctrl-0 = <&uart7m2_xfer>;
};

&uart9{
	status = "okay";/* Enable uart9 */
	pinctrl-0 = <&uart9m2_xfer>;
};
```

Main driver files:

```
RK356X_A11/kernel/drivers/tty/serial/8250/8250_core.c 	# 8250 UART driver core
RK356X_A11/kernel/drivers/tty/serial/8250/8250_dw.c 	# Synopsis DesignWare 8250 UART driver
RK356X_A11/kernel/drivers/tty/serial/8250/8250_dma.c 	# 8250 UART DMA driver
RK356X_A11/kernel/drivers/tty/serial/8250/8250_port.c 	# 8250 UART operation API
RK356X_A11/kernel/drivers/tty/serial/8250/8250_early.c 	# 8250串口early console driver
```

### 3.2 Common serial port operation commands

The stty tool is commonly used to configure and debug serial ports. Examples are as follows:

```
Display the information of a certain serial port parameter: stty -F /dev/ttyS1 -a
Set the information of a certain serial port parameter: stty -F /dev/ttyS1 speed 115200 cs7 -parenb -cstopb -echo (7-bit data, no parity check, 1 stop bit, no echo)
Set the serial port parameters: stty -F /dev/ttyS1 ispeed 115200 ospeed 115200 cs8;
Send data through the serial port: echo "abcdefg" > /dev/ttyS1;
Display the received data of the serial port: cat /dev/ttyS1;
The main functions of several key options of the stty command are as follows:
Option parenb enables the terminal to perform parity check, while -parenb disables the check;
Options cs5, cs6, cs7, and cs8 respectively set the character size to 5, 6, 7, and 8 bits;
Options 300, 600, 1200, 2400, 4800, 9600, and 19200 set the baud rate;
cstopb and -cstopb respectively set two or one stop bits;
tabs makes the system use tab characters instead of a sequence of spaces, thus reducing the output volume. The option -tabs only uses spaces. This option should be used when the terminal cannot correctly handle tab characters (tab).
```

After uart is enabled, you can burn boot and run it. Generally, you will see device files such as ttyS0 and ttyS1 under/dev/, which correspond to uart. You can use the following method to check the pairing of ttyS0 S1 and uart1 uart2 in dts: udevadm info /dev/ttyS0 command to check the serial port information. Pay attention to the output address information, which corresponds to the address defined by uart in dtsi. For example, the ttyS0 address below corresponds to fe690000 of uart5.

```
udevadm info /dev/ttyS0
P: /devices/platform/fe690000.serial/tty/ttyS0
N: ttyS0
L: 0
S: serial/by-path/platform-fe690000.serial
E: DEVPATH=/devices/platform/fe690000.serial/tty/ttyS0
E: DEVNAME=/dev/ttyS0
E: MAJOR=4
E: MINOR=64
E: SUBSYSTEM=tty
E: USEC_INITIALIZED=4566989
E: ID_PATH=platform-fe690000.serial
E: ID_PATH_TAG=platform-fe690000_serial
E: DEVLINKS=/dev/serial/by-path/platform-fe690000.serial
E: TAGS=:systemd
E: CURRENT_TAGS=:systemd:
```



### 3.3 Configuring UART2 as a normal serial port

Configure uart2 as a normal serial port by modifying it as follows
1. Close CONSOLE macro 

```
RK356X_A11/kernel/arch/arm64/configs/rockchip_defconfig
-CONFIG_SERIAL_8250_CONSOLE=y
+CONFIG_SERIAL_8250_CONSOLE is not set 
# Disable CONFIG_SERIAL_8250_CONSOLE
```

Close fiq-debugger node 

```
RK356X_A11/kernel/arch/arm64/boot/dts/rockchip/rk3568-android.dtsi 
	fiq-debugger {
		compatible = "rockchip,fiq-debugger";
		......
		pinctrl-0 = <&uart2m0_xfer>;
	-	status = "okay";
	+	status = "disabled";
	};
```

3. Open uart2

```
RK356X_A11/sunsea/project_sunsea/SIM8666/kernel/arch/arm64/boot/dts/rockchip/simcom/simcom.dtsi
+&uart2 {
+       status = "okay";
+};
```

Recompile the kernel and burn it to take effect. When it takes effect, the/dev/ttyS2 node will be generated.

### 3.4 References

Refer to debugging documentation Please refer to:
	RKDocs/common/UART/Rockchip_Developer_Guide_UART_CN.pdf

​	RKDocs/common/UART/Rockchip_Developer_Guide_UART_FAQ_CN.pdf

## 4 SPI configuration and debugging

The SIM8666 module offers up to three SPI interfaces. The SIM8666 SPI interface only supports master mode, defaults to Motorola SPI protocol, supports 8-bit and 16-bit, clock frequency up to 50MHz.
SPI and other functions multiplex pin, please refer to UART/SPI/I2C/I2S interface multiplexing function table, according to the actual situation to turn off/enable the corresponding function.

### 4.1 SPI Configuration

SPI DTS is configured in the number of devices as follows, and the path of the related device tree is: RK356X_A11/kernel/arch/arm64/boot/dts/rockchip/rk3568.dtsi

```
spi0: spi@fe610000 {
	compatible = "rockchip,rk3066-spi";
	reg = <0x0 0xfe610000 0x0 0x1000>;
	interrupts = <GIC_SPI 103 IRQ_TYPE_LEVEL_HIGH>;
	#address-cells = <1>;
	#size-cells = <0>;
	clocks = <&cru CLK_SPI0>, <&cru PCLK_SPI0>;
	clock-names = "spiclk", "apb_pclk";
	dmas = <&dmac0 20>, <&dmac0 21>;
	dma-names = "tx", "rx";
	pinctrl-names = "default", "high_speed";
	pinctrl-0 = <&spi0m0_cs0 &spi0m0_cs1 &spi0m0_pins>;
	pinctrl-1 = <&spi0m0_cs0 &spi0m0_cs1 &spi0m0_pins_hs>;/* Config spi0 pin */
	status = "disabled";/* Disable spi0 */
};
```

Main driver files:

```
RK356X_A11/kernel/drivers/spi/spi.c 			SPI driver framework
RK356X_A11/kernel/drivers/spi/spi-rockchip.c 	Implementation of each interface of the SPI protocol
RK356X_A11/kernel/drivers/spi/spidev.c 			Create a SPI device node for user-level use.
RK356X_A11/kernel/drivers/spi/spi-rockchip-test.c The SPI test driver needs to be manually added to the Makefile for compilation.
RK356X_A11/kernel/Documentation/spi/spidev_test.c User-mode SPI testing tool
```

### 4.2 Enabling SPI

SPI is enabled in dts and the relevant device tree file path: RK356X_A11/sunsearch/project_sunsearch/SIM8666/kernel/arch/arm64/boot/dts/rockchip/simcom/simcom.dtsi

```
&SPI0 {
	status = "okay";
};
```

Recompile the kernel to generate boot.img, burn boot.img to the device, and query pwm-related information through shell commands after the device is started.

```
# Cat SPI bus
ls /sys/bus/spi/devices/
# Check the information about SPI in the startup log.
dmesg | grep -i spi
```

If you see initialization messages like rockchip-spi or spi_master, the driver is loaded.

### 4.3 References

Refer to debugging documentation Please refer to:
	RKDocs/common/SPI/Rockchip_Developer_Guide_Linux_SPI_CN.pdf

## **5 PWM configuration and debugging**

The SIM8666 module offers up to 16 PWM interfaces. SIM8666 PWM interface and other functions multiplex pin, please refer to UART/SPI/I2C/I2S interface multiplexing function table, according to the actual situation to turn off/enable the corresponding function.

### 5.1 PWM Configuration

PWM DTS is configured in the number of devices as follows, and the path of the related device tree is: RK356X_A11/kernel/arch/arm64/boot/dts/rockchip/rk3568.dtsi

```
pwm4: pwm@fe6e0000 {
	compatible = "rockchip,rk3568-pwm", "rockchip,rk3328-pwm";
	reg = <0x0 0xfe6e0000 0x0 0x10>;
	#pwm-cells = <3>;
	pinctrl-names = "active";
	pinctrl-0 = <&pwm4_pins>;
	clocks = <&cru CLK_PWM1>, <&cru PCLK_PWM1>;
	clock-names = "pwm", "pclk";
	status = "disabled";
};
```

Main driver files:

```
RK356X_A11/kernel/drivers/pwm/pwm-rockchip.c
```

### 5.2 PWM debugging

Enable PWM in dts, the relevant device tree file path: RK356X_A11/sunsearch/project_sunsearch/SIM8666/kernel/arch/arm64/boot/dts/rockchip/simcom/simcom.dtsi

```
&pwm4 {
	status = "okay";
};
```

Recompile the kernel to generate boot.img, burn boot.img to the device, and query pwm-related information through shell commands after the device is started.

```
cd /sys/class/pwm/
ls
pwmchip0  pwmchip1
cd pwmchip0
/sys/devices/platform/fe6e0000.pwm/pwm/pwmchip0
```

It can be seen that there are two pwm, the number starts from 0, pwm0 corresponds to the lowest number of all enabled pwm channels, which can also be confirmed by the peripheral address, for example, the peripheral address of pwm4 is fe 6e0000.

PWM provides the interface of user layer. Under/sys/class/pwm/node, after PWM driver is successfully loaded, pwmchip0 directory will be generated under/sys/class/pwm/directory; writing 0 to export file means turning on pwm timer 0, which will generate pwm0 directory. Conversely, writing 0 to unexport will turn off pwm timer, and pwm0 directory will be deleted. There are several files in this directory:
	enable: Write 1 to enable pwm, write 0 to turn pwm off;
	polarity: normal or inverted two parameters to select, indicating that the output pin level flip;
	duty_cycle: in normal mode, indicates the duration of high level in one cycle (unit: nanosecond); in reversed mode, indicates the duration of low level in one cycle (unit: nanosecond);
	period: indicates the period of pwm wave (unit: nanosecond);
The following is an example of pwmchip0, set pwm0 output frequency 100K, duty cycle 50%, polarity is positive polarity:

```
cd /sys/class/pwm/pwmchip0/
echo 0 > export
cd pwm0
echo 10000 > period
echo 5000 > duty_cycle
echo normal > polarity
echo 1 > enable
```

### 5.3 References

Refer to debugging documentation Please refer to:
	RKDocs/common/PWM/Rockchip_Developer_Guide_Linux_PWM_CN.pdf		

​	RKDocs/common/PWM/Rockchip_Developer_Guide_PWM_IR_CN.pdf

## 6 GPIO configuration and debugging

GPIO (General-Purpose Input/Output) is a general-purpose pin that can be dynamically configured and controlled during software operation. All GPIOs are in input mode after power-on, which can be set to pull-up or pull-down by software, and can also be set to interrupt pins. The drive strength is programmable.

### 6.1 GPIO Number Calculation

SIM8666 has 5 GPIO banks: GPIO0~GPIO4, each group is divided by A0~ A7, B0~B7, C0~ C7, D0~D7 as the number, the following formula is commonly used to calculate the pins:

```
GPIO pin calculation formula: pin = bank * 32 + number
GPIO group number calculation formula: number = group * 8 + X
```

Here is how to calculate GPIO4_A6 pin:

bank = 4;      //GPIO4_A6 => 4, bank ∈ [0,4]

group = 0;      //GPIO4_A6 => 3, group ∈ {(A=0), (B=1), (C=2), (D=3)}

X =6;       //GPIO4_A6 => 6, X ∈ [0,7]

number = group * 8 + X = 0 * 8 + 6 = 6

pin = bank*32 + number= 4 * 32 + 6 = 134

### 6.2 Controlling I/O Using GPIO sysfs Interface

GPIO4_A6 may be occupied by other functions, the following are examples only. When the pin is not multiplexed by other peripherals, we can export the pin for use by exporting

```
:/ # ls /sys/class/gpio/
export     gpiochip128  gpiochip32   gpiochip64  unexport
gpiochip0  gpiochip255  gpiochip500  gpiochip96
:/ # echo 134 > /sys/class/gpio/export
:/ # ls /sys/class/gpio/
export   gpiochip0    gpiochip255  gpiochip500  gpiochip96
gpio134  gpiochip128  gpiochip32   gpiochip64   unexport
:/ # ls /sys/class/gpio/gpio134
active_low  device  direction  edge  power  subsystem  uevent  value
:/ # cat /sys/class/gpio/gpio134/direction
in
:/ # cat /sys/class/gpio/gpio134/value
0
```

### 6.3 Configure and use GPIO and IRQ in peripherals

Taking reset and irq GPIO of TP as examples, combined with TP driver, this paper introduces the configuration and use of GPIO driver.

Configure GPIO resources in TP peripheral information of device tree. Path of device tree: RK356X_A11/kernel/arch/arm64/boot/dts/rockchip/simcom/tp_mipi.dtsi

```
&gt1x {
    status = "disabled";
};

&i2c1 {
	gt9xx: gt9xx@5d {
		status = "disabled";
		compatible = "goodix,gt9xx";
		reg = <0x5d>;
		pinctrl-names = "default";
		pinctrl-0 = <&tp_gpio>; /* 配置pin */
		touch-gpio = <&gpio0 RK_PB5 GPIO_ACTIVE_HIGH>;
		reset-gpio = <&gpio0 RK_PB6 GPIO_ACTIVE_HIGH>;/* bank 引脚编号 高有效 */
		touchscreen-size-x = <800>;
		touchscreen-size-y = <1280>;
		......
	};
};

&pinctrl {
	tp {
		tp_gpio: tp-gpio {
			rockchip,pins =
			/* bank 引脚编号 功能 电气属性-上拉 */
				<0 RK_PB6 RK_FUNC_GPIO &pcfg_pull_up>,
				<0 RK_PB5 RK_FUNC_GPIO &pcfg_pull_up>;
		};
	};
};
```

In the TP driver file, get GPIO through Linux API, apply for and configure GPIO, driver file path: RK356X_A11/kernel/drivers/input/touchscreen/gt9xx/gt9xx.c

GPIO/IRQ related APIs are used as follows:

```
......
	/* Read the GPIO configuration numbers and flags of the touch-gpio and reset-gpio from the device tree. */
    ts->irq_pin = of_get_named_gpio_flags(np, "touch-gpio", 0, (enum of_gpio_flags *)(&ts->irq_flags));
    ts->rst_pin = of_get_named_gpio_flags(np, "reset-gpio", 0, &rst_flags);
......
	/* Apply for occupying this GPIO */
    ret = gpio_request(ts->rst_pin, "GTP_RST_PORT");
    ret = gpio_request(ts->irq_pin, "GTP_INT_IRQ");
......
	/* Set input mode */
	gpio_direction_input(ts->rst_pin);
	/* Set the output mode and output a low level. */
	gpio_direction_output(ts->rst_pin, 0); 
......
	/*  The GPIO pin values are converted to the corresponding IRQ values. */
	ts->irq=gpio_to_irq(ts->irq_pin); 
	/*  The registration thread was interrupted. The upper part was set to NULL, and the lower part was set to goodix_ts_irq_handler with IRQF_ONESHOT. After the interrupt handling was completed, the interrupt line remained in a disabled state until the thread processing function was finished. Interrupt name */
	ret = devm_request_threaded_irq(&(ts->client->dev), ts->irq, NULL, 
            goodix_ts_irq_handler, ts->irq_flags | IRQF_ONESHOT /*irq_table[ts->int_trigger_type]*/, ts->client->name, ts);
......
	/* Enable IRQ */
	enable_irq(ts->client->irq);
	/*  Disable IRQ */
	disable_irq(ts->client->irq);
......
	/*  Read GPIO value */
	gpio_get_value(ts->rst_pin);
......
	/*  Release GPIO */
	gpio_free(ts->rst_pin);
```

### 6.4 References

Refer to debugging documentation Please refer to:
	RKDocs/common/IO-Domain/Rockchip_Developer_Guide_Linux_IO_DOMAIN_CN.pdf

​	RKDocs/common/IO-Domain/Rockchip_PX30_Introduction_IO_Power_Domains_Configuration.pdf
​	RKDocs/common/IOMMU/Rockchip_Developer_Guide_Linux_IOMMU_CN.pdf

