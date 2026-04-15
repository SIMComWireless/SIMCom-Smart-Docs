# SIM866X_PCIE_SATA_RGMII Development User Guide



## **Version History**

| **versions**|**date**|**author**|**remark**|
| :------- | :-------- | :------- | :------- |
| 1.00     |2026.3.17|Ge Zhengyang| The first version|



## 1 Introduction

With this document you can quickly understand PCIE, SATA, RMII interface configuration and related debugging skills.



## 2 SIM866X PCIE interface

`Note: The PCIE20 and STAT2 interfaces of RK356X are shared. SIMCOM defaults to using this interface as PCIE RC.`

SIM8666 PCIE Interface Reference Configuration Table

| interface name| PCIE Type| Number of parallels (Lan)| RST Feet     | function description            |
| -------- | -------- | --------------- | --------- | ------------------- |
| PCIE20   |  PCIE 2.0 | 1               |  GPIO1_PB2 |RTL8111H Gigabit Ethernet|

The SIM866X module supports only one PCI E2.0 x1 bus at 5Gbps.



The file path of the related device tree is: RK356X_A11/sunsearch/project_sunsearch/SIM8666/kernel/arch/arm64/boot/dts/rockchip/simcom/simcom.dtsi

The PCIE original device tree configuration for the SIM8666 EVB is as follows:

~~~
/* PCIE2.0 x 1Lan */
&combphy2_psq {
	status = "okay";
};

&vcc3v3_pcie {
	vin-supply = <&dc_12v>;
	/delete-property/ enable-active-high;
	/delete-property/ gpio;
	/delete-property/ startup-delay-us;
};

&pcie2x1 {
	reset-gpios = <&gpio1 RK_PB2 GPIO_ACTIVE_HIGH>;
	vpcie3v3-supply = <&vcc3v3_pcie>;
	status = "okay";
};
~~~



SIM8668 EVB Reference Configuration Table

| interface name| PCIE Type| Number of parallels (Lan)| RST Feet     | function description            |
| -------- | -------- | --------------- | --------- | ------------------- |
| PCIE20   |  PCIE 2.0 | 1               |  GPIO1_PB2 |RTL8111H Gigabit Ethernet|
| PCIE30   |  PCIE 3.0 |2(default) or 1+1|  GPIO2_PD6 |M. 2 interface             |

SIM8668 supports one PCIE2.0 x1, bus rate 5Gbps; one PCIE3.0 x2, total rate 20Gbps.



The file path of the related device tree is: RK356X_A11/sunsearch/project_sunsearch/SIM8668/kernel/arch/arm64/boot/dts/rockchip/simcom/simcom.dtsi

The PCIE original device tree configuration for the SIM8668 EVB is as follows:

~~~
/* PCIE3.0 x 2Lan */
&pcie30phy {
	status = "okay";
};

&pcie3x2 {
	reset-gpios = <&gpio2 RK_PD6 GPIO_ACTIVE_HIGH>;
	vpcie3v3-supply = <&vcc3v3_pcie>;
	status = "okay";
};

/* PCIE2.0 x 1Lan */
&combphy2_psq {
	status = "okay";
};

&pcie2x1 {
	reset-gpios = <&gpio1 RK_PB2 GPIO_ACTIVE_HIGH>;
	vpcie3v3-supply = <&vcc3v3_pcie2>;
	status = "okay";
};

&vcc3v3_pcie2 {
	status = "okay";
};
~~~



### 2.1 PCIE debugging skills

Query PCIE Device using`lspci` command

For other PCIE debugging documentation, please refer to RKDocs/common/PCIe/Rockchip_Development_Guide_PCIe_CN.pdf



## 3 SIM866X EVB RMII interface

`Note: Due to the interface reusability relationship of the RK platform, the SIM866X series platform is by default only capable of supporting one RGMII interface.`

SIM866X EVB Reference Configuration Table

| interface name| type| support type              | RST Feet     | function description            |
| -------- | ---- | --------------------- | --------- | ------------------- |
| RGMII1   |  GMAC |10/100/1000 Mbps Ethernet port|  GPIO4_PC2 |RTL8211F Gigabit Ethernet|

SIM866X RMII device tree related configuration

Related equipment tree file path in:

RK356X_A11/sunsea/project_sunsea/SIM8666/kernel/arch/arm64/boot/dts/rockchip/simcom/simcom.dtsi

~~~
/* Ethnet RGMII PHT configurations */
&gmac1 {
	snps,reset-active-low;
	phy-supply = <&gmac_3v3_reg>;
	snps,reset-gpio = <&gpio4 RK_PC2 GPIO_ACTIVE_LOW>;
	pinctrl-0 = <&gmac1m0_rst
				&gmac1m0_miim
				&gmac1m0_tx_bus2_level3
				&gmac1m0_rx_bus2
				&gmac1m0_rgmii_bus_level3
				&gmac1m0_rgmii_clk_level2>;

	tx_delay = <0x41>;
	rx_delay = <0x2e>;
	status = "okay";
};
~~~



RK356X_A11/sunsea/project_sunsea/SIM8668/kernel/arch/arm64/boot/dts/rockchip/simcom/simcom.dtsi

~~~
/* Ethnet RGMII PHY configurations */
&gmac0 {
	/* not used */
	status = "disabled";
};

&gmac1 {
	snps,reset-active-low;
	phy-supply = <&gmac_3v3_reg>;
	snps,reset-gpio = <&gpio4 RK_PC2 GPIO_ACTIVE_LOW>;
	pinctrl-0 = <&gmac1m0_rst
				&gmac1m0_miim
				&gmac1m0_tx_bus2_level3
				&gmac1m0_rx_bus2
				&gmac1m0_rgmii_clk_level2
				&gmac1m0_rgmii_bus_level3>;

	tx_delay = <0x41>;
	rx_delay = <0x2e>;
	status = "okay";
};
~~~



### 3.1 RMII debugging techniques

Query Ethernet devices using the`ifconfig -a` command

Refer to RKDocs/common/GMAC/Rockchip_Developer_Guide_Ethernet_CN.pdf for additional RMII debugging documentation.



## 4 SIM8668 EVB SATA interface

`Note: Due to the interface reuse relationship of the RK platform, only the SIM8668 platform is by default supported to have 1 SATA interface. The SIM8666 platform does not support SATA.`

SIM8668 EVB Reference Configuration Table

| interface name| type    | rate| 12V & 5V power supply enable| function description     |
| -------- | ------- | ----- | -------------- | ------------ |
| SATA0    |  STAT3.0 | 6Gbps | GPIO3_PA0      | SATA hard disk interface|

For additional SATA debug documentation, please refer to RKDocs/common/NVM/Rockchip_Development_Guide_SATA_CN.pdf