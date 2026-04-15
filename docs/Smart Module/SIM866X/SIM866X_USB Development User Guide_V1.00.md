# SIM866X_USB Development User Guide



## **version history**

| **versions**|**date**|**author**|**remark**|
| :------- | :-------- | :------- | :------- |
| 1.00     |2026.3.17| Ge Zhengyang| The first version  |



## 1 Introduction

Read this document to quickly learn about SIM866X Series USB interface configuration and related debugging tips.



## 2 SIM8666 USB Interface List

SIM8666 USB Interface Configuration Reference Table

| interface name| USB Type| USB interface       | VBUS enable foot       | function description                                                |
| -------- | ------- | ------------- | ----------------- | ------------------------------------------------------- |
| USB0     |  USB 2.0 | Type-C（OTG） | GPIO0_PA5         | ADB firmware download                                      |
| USB1     |  USB 3.0 | USB-A（HOST） |GPIO0_PA6 (shared)| connectivity peripherals                                                |
| USB2     |  USB 2.0 | USB-A（HOST） |GPIO0_PA6 (shared)| connectivity peripherals                                                |
| USB3     |  USB 2.0 | USB-A（HOST） | GPIO2_PC6         | Connect internal HUB, HUB expands 2-way TYPE-A interface and SIM7600 external 4G module|



## 3 SIM8666 USB-related device tree configuration

SIM8666 supports 4-channel USB, of which USB0 supports OTG function, and the rest of USB1~3 only supports Host mode. Only USB1 supports USB 3.0 functionality.

The file path of the related device tree is: RK356X_A11/sunsearch/project_sunsearch/SIM8668/kernel/arch/arm64/boot/dts/rockchip/simcom/simcom.dtsi

The SIM8666 EVB USB original device tree configuration is as follows:

~~~dts
/* USB0 TYPE-C vbus regulator */
&vcc5v0_otg {
	gpio = <&gpio0 RK_PA5 GPIO_ACTIVE_HIGH>;
	pinctrl-names = "default";
	pinctrl-0 = <&vcc5v0_otg_en>;
};

/* HUB vbus regulator */
&vcc5v0_host {
	gpio = <&gpio2 RK_PC6 GPIO_ACTIVE_HIGH>;
	pinctrl-names = "default";
	pinctrl-0 = <&vcc5v0_host_en>;
};

/ {
	/* USB2.0 HOST2 / USB3.0 HOST vbus regulator */
	vcc5v0_host1: vcc5v0-host1-regulator {
		compatible = "regulator-fixed";
		regulator-name = "vcc5v0_host1";
		regulator-boot-on;
		regulator-always-on;
		regulator-min-microvolt = <5000000>;
		regulator-max-microvolt = <5000000>;
		enable-active-high;
		gpio = <&gpio0 RK_PA6 GPIO_ACTIVE_HIGH>;
		vin-supply = <&vcc5v0_usb>;
		pinctrl-names = "default";
		pinctrl-0 = <&vcc5v0_host1_en>;
	};
};

/* USB0 TYPE-C Port USB2.0 */
&u2phy0_otg {
	vbus-supply = <&vcc5v0_otg>;
	status = "okay";
};

&usb2phy0 {
	status = "okay";
};

&usbdrd_dwc3 {
	dr_mode = "otg";
	status = "okay";
};

&usbdrd30 {
	status = "okay";
};

/* USB1 TYPE-A Port USB3.0 */
&combphy1_usq {
	status = "okay";
};

&u2phy0_host {
	phy-supply = <&vcc5v0_host1>;
	status = "okay";
};

&usb2phy0 {
	status = "okay";
};

&usbhost_dwc3 {
	status = "okay";
};

&usbhost30 {
	status = "okay";
};

/* USB2 TYPE-A Port USB2.0 */
&u2phy1_otg {
	phy-supply = <&vcc5v0_host1>;
	status = "okay";
};

&usb2phy1 {
	status = "okay";
};

&usb_host0_ehci {
	status = "okay";
};

&usb_host0_ohci {
	status = "okay";
};

/* USB3 HUB Port USB2.0 */
&u2phy1_host {
	phy-supply = <&vcc5v0_host>;
	status = "okay";
};

&usb2phy1 {
	status = "okay";
};

&usb_host1_ehci {
	status = "okay";
};

&usb_host1_ohci {
	status = "okay";
};
~~~





## 4 SIM8668 EVB USB Interface List

SIM8668 EVB USB Interface Configuration Reference Table

`Note: The USB0 interface of RK3568 supports USB 2.0 + SATA 3.0 (default configuration) or USB 3.0 mode.`

| interface name| USB Type| USB interface       | VBUS enable pin       | function description                                                |
| -------- | ------- | ------------- | ----------------- | ------------------------------------------------------- |
| USB0     |  USB 2.0 | Type-C（OTG） | GPIO0_PA5         | ADB firmware download                                      |
| USB1     |  USB 3.0 | USB-A（HOST） |GPIO0_PA6 (shared)| connectivity peripherals                                                |
| USB2     |  USB 2.0 | USB-A（HOST） |GPIO0_PA6 (shared)| connectivity peripherals                                                |
| USB3     |  USB 2.0 | USB-A（HOST） | GPIO2_PC6         | Connect internal HUB, HUB expands 2-way TYPE-A interface and SIM7600 external 4G module|

If you need to modify USB type, please refer to **"3.1.2 RK3568 OTG Configuration to USB3.0"** section of RKDocs/common/usb/Rockchip_RK356X_User_Guide_USB_CN.pdf



## 5 SIM8668 USB-related device tree configuration

SIM8666 supports 4-way USB, of which USB0 supports OTG function, and the rest of USB1~3 only supports Host mode. By default, only USB1 supports USB 3.0 functionality. For USB0 support for USB 3.0, please refer to the RK official documentation.

The file path of the related device tree is: RK356X_A11/sunsearch/project_sunsearch/SIM8666/kernel/arch/arm64/boot/dts/rockchip/simcom/simcom.dtsi

The SIM8668 EVB USB original device tree configuration is as follows:

~~~
/* USB0 TYPE-C vbus regulator */
&vcc5v0_otg {
	gpio = <&gpio0 RK_PA5 GPIO_ACTIVE_HIGH>;
	pinctrl-names = "default";
	pinctrl-0 = <&vcc5v0_otg_en>;
};

/* HUB vbus regulator */
&vcc5v0_host {
	gpio = <&gpio2 RK_PC6 GPIO_ACTIVE_HIGH>;
	pinctrl-names = "default";
	pinctrl-0 = <&vcc5v0_host_en>;
};

/ {
	/* USB2.0 HOST2 / USB3.0 HOST vbus regulator */
	vcc5v0_host1: vcc5v0-host1-regulator {
		compatible = "regulator-fixed";
		regulator-name = "vcc5v0_host1";
		regulator-boot-on;
		regulator-always-on;
		regulator-min-microvolt = <5000000>;
		regulator-max-microvolt = <5000000>;
		enable-active-high;
		gpio = <&gpio0 RK_PA6 GPIO_ACTIVE_HIGH>;
		vin-supply = <&vcc5v0_usb>;
		pinctrl-names = "default";
		pinctrl-0 = <&vcc5v0_host1_en>;
	};
};

/* USB0 TYPE-C Port USB2.0 */
&u2phy0_otg {
	vbus-supply = <&vcc5v0_otg>;
	status = "okay";
};

&usb2phy0 {
	status = "okay";
};

&usbdrd_dwc3 {
	dr_mode = "otg";
	status = "okay";
};

&usbdrd30 {
	status = "okay";
};

/* USB1 TYPE-A Port USB3.0 */
&combphy1_usq {
	status = "okay";
};

&u2phy0_host {
	phy-supply = <&vcc5v0_host1>;
	status = "okay";
};

&usb2phy0 {
	status = "okay";
};

&usbhost_dwc3 {
	status = "okay";
};

&usbhost30 {
	status = "okay";
};

/* USB2 TYPE-A Port USB2.0 */
&u2phy1_otg {
	phy-supply = <&vcc5v0_host1>;
	status = "okay";
};

&usb2phy1 {
	status = "okay";
};

&usb_host0_ehci {
	status = "okay";
};

&usb_host0_ohci {
	status = "okay";
};

/* USB3 HUB Port USB2.0 */
&u2phy1_host {
	phy-supply = <&vcc5v0_host>;
	status = "okay";
};

&usb2phy1 {
	status = "okay";
};

&usb_host1_ehci {
	status = "okay";
};

&usb_host1_ohci {
	status = "okay";
};
~~~



## 6 Other commissioning instructions



### 6.1 Forced Switching OTG Mode

USB working mode description: In addition to USB0 support OTG, USB1/2/3 only support Host mode can not be forced to switch USB working mode.

The RK 356x SDK supports software methods to force USB0 OTG to switch to Host mode or device mode, regardless of the USB hardware circuit's OTG ID level or Type-C interface.

The reference instructions are as follows:

~~~
#1.Force host mode
echo host > /sys/devices/platform/fe8a0000.usb2-phy/otg_mode

#2.Force peripheral mode
echo peripheral > /sys/devices/platform/fe8a0000.usb2-phy/otg_mode

#3.Force otg mode
echo otg > /sys/devices/platform/fe8a0000.usb2-phy/otg_mode
~~~



If USB0 needs to be forced to work in a certain mode after the device is powered on, it is recommended to keep the default configuration of the device tree and then add the working mode switch command in system/core/rootdir/init.rc.

The reference instructions are as follows:

~~~
on init
	write /sys/devices/platform/fe8a0000.usb2-phy/otg_mode "host"
~~~



### 6.2 Query USB Device

Use`lsusb` or`cat /sys/kernel/debug/usb/devices` command to query the USB peripherals identified by the current device



### 6.3 USB Host Eye Test

RK documentation Rockchip_Developer_Guide_USB_SQ_Test_CN.pdf mentions using the`io -4 0xff580440 0x8000` command to package is not available in the current SDK.

io programs need to access/dev/mem devices, but Android versions do not allow the creation of/dev/mem devices for security reasons. Forcing this device driver to open will cause compilation errors.

Therefore, it is recommended to use ehset driver in combination with EHSET TEST Fixture for testing. Before testing, add`CONFIG_USB_EHSET_TEST_FIXTURE=y`to kernel defconfig, and then connect the port under test to the package board to trigger the eye diagram test signal.

For eye diagram test, please contact FAE, SIMCOM provides test calibration service.



### 6.4 USB Gadget driver development

Refer to RK documentation: RKDocs/common/usb/Rockchip_Developer_Guide_USB_CN.pdf