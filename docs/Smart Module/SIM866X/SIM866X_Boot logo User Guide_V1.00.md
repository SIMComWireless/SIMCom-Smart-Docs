# SIM866X_Boot logo User Guide

## **Version History**

| **versions**|**date**|**author**|**remark**|
| :------- | :-------- | :------- | :------- |
| 1.00     |2026.3.17| Liu Yi| The first version   |

# 1 Introduction

## 1.1 Purpose of this article

This article mainly describes the use of Uboot logo and kernel logo on SIM 866X module. This article will take the default display on the SIM8668 EVB (RK3568) board as an example to introduce the logo display configuration in detail from the software level.

## 1.2 References

[1] Rockchip_Developer_Guide_DRM_Display_Driver_CN.pdf

# 2 U-Boot drivers

## 2.1 Display drive

1. The display driver mainly provides two functions in U-Boot: boot logo display and charging interface display.
   On the Rockchip platform, the boot logo is generally divided into two stages: displaying the U-Boot logo and displaying the Kernel logo. (Photo omitted)

2. Two LOGO images are placed in the Linux kernel root directory (logo.bmp and logo_kernel.bmp) by default, and the Linux Kernel is compiled at the time.
   They are packaged into resource.img and then packaged into Boot.img.
   When U-Boot starts, these two files will be loaded into memory. The U-Boot LOGO will be displayed at the U-Boot stage, and the Kernel Logo will be in memory.
   The address is passed to the Linux kernel by U-Boot and is displayed during the drm driver initialization phase of the Linux Kernel.


## 2.2 Driver directory

drivers/video/drm/

## 2.3 Driver files

​		

| Driver    | File                                                         |
| --------- | ------------------------------------------------------------ |
| Core      | rockchip_display.c rockchip_crtc.c rockchip_connector.c rockchip_phy.c rockchip_panel.c rockchip_bridge.c |
| VOP       | rockchip_vop.c rockchip_vop_reg.c rockchip_vop2.c rockchip_vop2_reg.c |
| RGB       | rockchip_rgb.c inno_video_combo_phy.c                        |
| LVDS      | rockchip_lvds.c inno_video_combo_phy.c                       |
| MIPI-DSI  | drm_mipi_dsi.c dw_mipi_dsi2.c inno_mipi_phy.c inno_video_combo_phy.c samsung_mipi_dcphy.c |
| eDP       | rockchip_analogix_dp.c rockchip_analogix_dp_reg.c            |
| HDMI      | dw_hdmi.c rockchip_dw_hdmi.c rockchip-inno-hdmi-phy.c inno_hdmi.c dw_hdmi_qp.c rockchip_dw_hdmi_qp.c phy-rockchip-samsung-hdptx-hdmi.c |
| TVE /CVBS | rockchip_drm_tve.c                                           |
| DP        | dw-dp.c drm_dp_helper.c phy-rockchip-usbdp.c                 |

# 3. Interface Description

1. Show U-Boot logo

   ```c
   void rockchip_show_logo(void)
   ```

2. Display the specified bmp picture, currently mainly used for charging logo display

   ```c
   void rockchip_show_bmp(const char *bmp)
   ```

3. Transfer some variables determined in U-Boot to kernel by modifying dtb file, including kernel logo size, address, format, output scanning time
  Order and configuration of overscan

  ```c
  void rockchip_display_fixup(void *blob)
  ```

# 4. Application description

1. Open U-Boot logo
logo through the Linux kernel dts(U-Boot display module and Linux kernel multiplexing the same dtb) corresponding to the display interface route_xxx node
Control, xxx here can be dsi, edp, hdmi, lvds, please search route keyword in dts for details.
Take MIPI DSI0 as an example, find the route_dsi0 node in the corresponding board-level dts file, and set the status to "okay":

```c
diff --git a/arch/arm64/boot/dts/rockchip/rk3588-evb1-lp4.dtsi b/arch/arm64/boot/dts/rockchip/rk3588-evb1-lp4.dtsi
index 543d78d3f182..b634192d77df 100644
--- a/arch/arm64/boot/dts/rockchip/rk3588-evb1-lp4.dtsi
+++ b/arch/arm64/boot/dts/rockchip/rk3588-evb1-lp4.dtsi
@@ -655,7 +655,7 @@
 };
 
 &route_dsi0 {
-       status = "disabled";
+       status = "okay";
        connect = <&vp3_out_dsi0>;
 };
```

2. Configure U-Boot logo to display in full screen
  Logo is displayed in the center by default. If full-screen display is required, modify the route node of the corresponding interface.
  For example, if you need to display the logo output on DSI0 in full screen, modify the logo,mode attribute of route_dis0 to "fullscreen".

  ```c
  --- a/arch/arm64/boot/dts/rockchip/rk3588s.dtsi
  +++ b/arch/arm64/boot/dts/rockchip/rk3588s.dtsi
  @@ -1065,7 +1065,7 @@
          status = "disabled";
          logo,uboot = "logo.bmp";
          logo,kernel = "logo_kernel.bmp";
  -       logo,mode = "center";
  +       logo,mode = "fullscreen";
          charge_logo,mode = "center";
          connect = <&vp3_out_dsi0>;
  ```

  

3. Logo image requirements
  The U-Boot logo and Linux Kernel logo have the same resolution, and the resolution must be even.
  Only supports 8bit, 16bit, 24bit, 32bit bmp pictures;
  U-Boot logo and Linux Kernel logo must be enabled at the same time, i.e. logo.bmp and logo_kernel.bmp must be provided at the same time, not only logo.bmp
  One of them.

4. Start log confirmation
  If the logo function is enabled properly, you will see logs similar to the following during the U-Boot phase:

  ```c
  Rockchip UBOOT DRM driver version: v1.0.1
  vp0 have layer nr:2[0 2 ], primary plane: 2
  vp1 have layer nr:2[1 3 ], primary plane: 3
  vp2 have layer nr:2[6 8 ], primary plane: 8
  vp3 have layer nr:2[7 9 ], primary plane: 9
  Using display timing dts
  dsi@fde20000: detailed mode clock 132000 kHz, flags[a]
    H: 1080 1095 1099 1129
    V: 1920 1935 1937 1952
  bus_format: 100e
  VOP update mode to: 1080x1920p0, type: MIPI0 for VP3
  VOP VP3 enable Esmart3[654x270->654x270@213x825] fmt[2] addr[0xedf04000]
  final DSI-Link bandwidth: 880000 Kbps x 4
  ......
  hdmi_select_link_config use tmds mode
  mode:1920x1080 bus_format:0x100a
  hdmi@fde80000: detailed mode clock 148500 kHz, flags[5]
    H: 1920 2008 2052 2200
    V: 1080 1084 1089 1125
  bus_format: 100a
  VOP update mode to: 1920x1080p0, type: HDMI0 for VP0
  ......
  VOP VP0 enable Esmart0[654x270->654x270@633x405] fmt[2] addr[0xedf04000]
  ......
  CLK: (uboot. arm: enter 1200000 KHz, init 1200000 KHz, kernel 0N/A)
  ```

  You can see that the logo display is enabled on both VP0 and VP3.
  VP0 has a display resolution of 1920 x 1080 and a HDMI interface.
  The VP3 has a display resolution of 1080 x 1920 and a display interface of MIPI DSI.
  The displayed logo size is 654 x 270.

# 5 Analyze the splash screen or problems that cannot be displayed during the transition from U-Boot logo to Linux Kernel logo

1. Confirm whether the DRM driver has been loaded normally.
During the loading process of the DRM driver, some resources may not be ready, causing the DRM driver to fail to bind. Something similar to the following log may occur:

  ```c
  [1.792387] dw-mipi-dsi2 fde20000.dsi: [drm:dw_mipi_dsi2_bind] *ERROR* Failed to find
  panel or bridge: -517
  ```

 This log indicates that the panel or bridge is not ready at this moment. The normal driver framework will restart the binding process after a certain period of time. However, if the following log appears later, it means that the DRM driver has been successfully loaded. In such cases, we don't need to pay too much attention to the previous failed binding logs.

  ```c
  [2.566831] rockchip-drm display-subsystem: [drm] fb0: rockchipdrmfb frame buffer
  device
  ```

  

  If the startup log keeps continuously displaying log messages indicating "bind failure", then you may need to check your DTS configuration. The most common issues encountered in the product are that the GPIO port is registered by other devices first, the compatible setting of the panel is incorrect, resulting in panel registration failure, and the backlight driver registration fails, etc. If the following logo failure-related information appears in the Linux kernel log, then you need to carefully analyze it by referring to the code:

  ```c
  rockchip-drm display-subsystem: failed to parse resources for logo display
  rockchip-drm display-subsystem: connector[HDMI-A-1] can't found any modes
  rockchip-drm display-subsystem: can't not find any logo display
  rockchip-drm display-subsystem: failed to show loader logo
  ```

1. The DDR frequency change causes screen flickering or display misalignment.
  1) Try to disable the DDR frequency adjustment node in the dts file to ensure that no DDR frequency adjustment is performed during the kernel stage. The modification method is as follows:

  ```c
  &dmc {
  	status = “disabled”;
  };
  ```

  2) Try to adjust the DCLK frequency when shutting down the DDR frequency conversion. The modification method is as follows:

  ```c
  &dmc {
  	vop-dclk-mode = <1>;
  };
  ```

3. The change in the clk tree has caused the clk tree configuration in some platforms' U-Boot to be independent of the clk tree configuration in the kernel. If the clk strategies of the two drivers are inconsistent, there may be a screen flicker problem when the Linux kernel's clk is reinitialized. As an example with RK3399, the following method can be used to confirm:
1) Add "while(1)" at the beginning of the rk3399_clk_init() function in the kernel/drivers/clk/rockchip/clk-rk3399.c file. Confirm if there will be a screen flicker.
2) Add "while(1)" at the end of the rk3399_clk_init() function in the kernel/drivers/clk/rockchip/clk-rk3399.c file to check if there will be a screen flicker.
3) If there is no screen flicker in step (1) but there is screen flicker in step (2), it can be basically confirmed that the screen flicker issue is caused by the change in the clock tree. You can directly contact the corresponding platform cru person or submit it to redmine and explain to transfer it to the person in charge of the PLL.

4. The clock being turned off may have led to some process-related issues or software bugs. There might be some necessary clocks that were not enabled during the driver registration process, causing these clocks to be automatically shut down by the framework after the kernel execution is completed, resulting in display abnormalities. You can try the following modification to test without turning off the clock by default:

  In the dts file, locate the "chosen" node. Add "clk_ignore_unused" at the end of the "bootargs" line, such as "bootargs = "xxxx clk_ignore_unused"."

  ```c
  --- a/arch/arm64/boot/dts/rockchip/rk3399-linux.dtsi
  +++ b/arch/arm64/boot/dts/rockchip/rk3399-linux.dtsi
  @@ -47,7 +47,7 @@
   	compatible = "rockchip,linux", "rockchip,rk3399";
   
   	chosen {
  -		bootargs = "earlycon=uart8250,mmio32,0xff1a0000";
  +		bootargs = "earlycon=uart8250,mmio32,0xff1a0000 clk_ignore_unused";
   	};
  ```

5. The sizes of the U-Boot logo image and the kernel logo image are not consistent.
On the Rockchip platform, there is a requirement that the resolution sizes of the U-Boot logo and the kernel logo should be the same. If the U-Boot logo displays normally but the kernel stage shows abnormalities, it can be confirmed whether the resolution of logo.bmp and logo_kernel.bmp in the kernel directory is consistent.

  ```c
  $file logo.bmp logo_kernel.bmp
  logo.bmp:      PC bitmap, Windows 3.x format, 654 x 258 x 8
  logo_kernel.bmp: PC bitmap, Windows 3.x format, 654 x 258 x 8
  ```

6. During the kernel initialization process, some power GPIOs were reinitialized. There are many possible causes for this issue. In short, during the porting process of the new project, it is necessary to promptly confirm whether the display-related GPIO power sources have conflicts with other modules; if the DTS is correct, keywords related to the serial port can be searched in the kernel code, and the location of each module loading can be traversed using binary search to gradually identify the point causing the screen flickering problem.

7. VOP Priority Configuration Issue
  If the VOP priority is not set to the highest level, it is possible that during the kernel loading stage, the bus will be occupied by other IPs, resulting in a screen flickering issue. This problem is usually fixed before the SDK is released.

8. Test the related power supply and signals
  If the above steps do not solve the splash screen issue, please use an oscilloscope to capture the waveforms of signals such as CLK, DATA and power from U-Boot to the kernel stage, and submit the results to redmine.
