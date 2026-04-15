# SIM866X_PANEL Migration User Guide

## **Version History**

| **versions**|**date**|**author**|**remark**|
| :------- | :-------- | :------- | :------- |
| 1.00     |2026.3.16|Liu Yi| The first version   |

## 1 Introduction

## 1.1 Purpose of this article

This article mainly describes the display driver guide for HDMI, EDP, MIPI, LVDS and other interfaces on the SIM 866X module based on the linux platform. It is divided into two parts: Uboot and kernel. This article will take the default display on SIM8668 EVB (RK3568) board as an example, and introduce the display software function configuration in detail from the software level.

## 1.2 References

【1】Rockchip_Developer_Guide_DRM_Panel_Porting_CN.pdf

【2】Rockchip_Developer_Guide_HDMI_CN.pdf

【3】Rockchip_Developer_Guide_MIPI_DSI2_CN.pdf

【4】Rockchip_Developer_Guide_LVDS_CN.pdf

【5】Rockchip_Developer_Guide_eDP_CN.pdf

## 1.3 Terminology and abbreviations

VOP：Video Output 

# 2 Information needed for debugging screens

## 2.1 Vendor Supplied Parts

1. Screen specifications, which need to be provided by the screen factory;

2. If the screen supports reading edid, this item is ignored. Screen related width, height, hsync, hfp, hbp, vsync, vfp,

   vbp, fps, lane_umber, pixel format, reset timing requirements, de-active, hsync-active, vsynv-active and other parameters;

3. Schematic diagram related to screen and module connection;

## 2.2 Code catalogue

The following is an example of evb screen to describe the location of the relevant configuration in the code:

RK356X_A11/sunsea/project_sunsea/SIM8668/u-boot/drivers/video/drm/

├── rockchip_display.c

 

RK356X_A11/sunsea/project_sunsea/SIM8668/u-boot/common/

├── edid.c

 

RK356X_A11/sunsea/project_sunsea/SIM8668/kernel/arch/arm64/boot/dts/rockchip/simcom/

└── lcd.dtsi

└── hdmi.dtsi

└── simcom.dtsi

└── mipi_open.dtsi

## 2.3 VOP configuration

The RK3568 chip is equipped with a VOP controller and has three ports for output, supporting HDMI 2.0 TX/MPI DSI.

TX1/LVDS TX0/LVDS TX1/eDP TX/RGB/BT1120/BT656 video interface output.

# 3 HDMI configuration

The configuration of RK platform hdmi in uboot stage is also obtained through dtsi, and no additional modifications are required.

The path to the dtsi configuration file for the kernel section is as follows:

RK356X_A11/sunsea/project_sunsea/SIM8668/kernel/arch/arm64/boot/dts/rockchip/simcom/hdmi.dtsi

For attributes and attribute descriptions, refer to document:

RK356X_A11/kernel/Documentation/devicetree/bindings/display/rockchip/dw_hdmi-rockchip.txt

## 3.1 Enable HDMI

Code Path:

RK356X_A11/sunsea/project_sunsea/SIM8668/kernel/arch/arm64/boot/dts/rockchip/simcom/hdmi.dtsi

```c
&hdmi {
	status = "okay";
};
```

## 3.2 Binding VOP

- In each platform of RK, the image data output by various display interfaces (HDMI, MIPI, etc.) comes from VOP;

- As can be seen from the VOP configuration diagram in Section 2.3, HDMI can be tied to VP0 or VP1. It is suggested that binding VP0 can support 4K output;

- When the display device node in dts is open, the ports corresponding to VP0 and VP1 of the display interface will be open, so you need to close the port corresponding to the VOP that is not needed;

  ```c
  
  &hdmi_in_vp0 {
      status = "okay";
  };
  &hdmi_in_vpl {
      status = "disabled";
  };
  ```

## 3.3 Turn on the logo

- If the u-boot logo is not turned on, the kernel stage cannot display the boot logo, and you can only wait until the system boots to see the image displayed by the application. Turn on route_hdmi enable in dts to support u-boot logo display;
- On this platform (supporting three VOPs), it is necessary to note that the VOP specified by connect in the code configuration below must be consistent with the VOP bound to HDMI, otherwise problems such as color screen may occur;

```c
&route_hdmi {
	status = "okay";
	connect = <&vp0_out_hdmi>;
};
```

## 3.4 vop bound PLL

- RK3568 HDMI bound VP dclk must be mounted on HPLL;

```c
&vop {
	assigned-clocks = <&cru DCLK_VOP0>, <&cru DCLK_VOP1>, <&cru DCLK_VOP2>;
	assigned-clock-parents = <&pmucru PLL_HPLL>, <&cru PLL_VPLL>, <&cru PLL_VPLL>;
	status = "okay";
};
```

## 3.5 HDMI DEBUG

The following is a brief introduction to several analysis points that need to be noted when the screen is not lit:

1. Confirm current connection status:

```c
root@rk3568-yocto:/# cat /sys/class/drm/card0-HDMI-A-1/status
connected
root@rk3568-yocto:/#
```

If status is disconnected, it may be a monitor connection or software configuration problem.

First, make sure that both ends of the HDMI cable (development board and display) are firmly inserted, confirm that the display itself is working properly, and has been switched to the appropriate input source. At the same time, ensure that the development board has sufficient power supply. Unstable power supply may also affect HDMI output.

2. Check available modes:

Execute the cat /sys/class/drm/card0-HDMI-A-1/modes command. If this file exists and several resolutions are listed inside, it usually means that the driver is basically normal, but no display is currently detected. If the file is missing or empty, it may indicate a deeper problem at the driver or hardware level.

3. View kernel logs:

Run dmesg| grep -i hdmi, check the HDMI logs during kernel startup and operation, especially if there are failed to get EDID information or error messages related to the physical layer (PHY).

4. The logo is not displayed during startup:

Confirm that the HDMI 5V path is normal, that is to say, the power supply of HDMI 5V is normal. Corresponding power supply can be configured in uboot;

You can view the corresponding hdmi log in the boot log. If it is successful, there will be the following log information:

```c
hdmi@fe0a0000: detailed mode clock 148500 kHz, flags[5]
H: 1920 2008 2052 2200 V:1080 1084 1089 1125 bus_format: 2025
VOP update mode to: 1920x1080p60, type: HDMI0 for VP0
VP0 set crtc clock to 148500KHz
VOP VP0 enable Smart1[654x270->654x270@633x405] fmt[0] addr[0x7df00000]
CEA mode used vic=16
final pixclk = 148500000 tmdsclk = 148500000
```

5. Confirm the current status of the displayed route:

```c
root@rk3568-yocto:/sys/kernel/debug/dri/0# cat summary
Video Port0: ACTIVE
 Connector:HDMI-A-1  Encoder: TMDS-175
    bus_format[2025]: YUV8_1X24
    overlay_mode[1]  output_mode[f]  color_space[3], eotf:0
    Display mode: 1920x1080p60
    clk[148500] real_clk[148500] type[48] flag[5]
    H: 1920 2008 2052 2200
    V: 1080 1084 1089 1125
Smart1-win0: ACTIVE
    win_id: 1
    format: XR24 little-endian (0x343225258) SDR[0] color_space[0] glb_alpha[0xff]
    rotate: xmirror: 0  ymirror: 0  rotate_90: 0  rotate_270: 0
    csc: y2r[0] r2y[1] csc_mode[1]
    zpos: 0
    src: pos[0, 0] rect[1024 x 600]
    dst: pos[38, 0] rect[1844 x 1080]
    buf[0]: addr: 0x000000007e84b000 pitch: 4096 offset: 0
```

You can view information about the resolution of the screen.

6. After checking the above points, if it is still abnormal, you need to grasp the complete log for analysis, and export the fdt of the board for analysis to see if the current screen configuration is effective.

# 4 MIPI configuration

The configuration of RK platform dsi in uboot stage is also obtained through dtsi, and no additional modification is required. The path to the dtsi configuration file for the kernel section is as follows:

RK356X_A11/sunsea/project_sunsea/SIM8668/kernel/arch/arm64/boot/dts/rockchip/simcom/mipi_open.dtsi

For attributes and attribute descriptions, refer to document:

RK356X_A11/kernel/Documentation/devicetree/bindings/display/rockchip/dw_mipi_dsi_rockchip.txt

## 4.1 Schematic confirmation

It is generally necessary to confirm whether the power supply of MIPI, pin connection, RST pin and backlight control part are correct through schematic diagram, which are used for subsequent configuration, and whether the connection of each lane is correct.

## 4.2 Enable MIPI

According to the schematic diagram of SIM8668, MIPI screen is connected to DSI1, so it is necessary to open dsi1 and close dsi0;

Code Path:

```c
&dsi0 {
    status = "disabled";
};
&dsil {
    status = "okay";
};
```

## 4.3 Binding VOP

- In each platform of RK, the image data output by various display interfaces (HDMI, MIPI, etc.) comes from VOP;
- As can be seen from the VOP configuration diagram in Section 2.3, MIPI DSI1 can be bound to VP0 or VP1. On the SIM8668 project, we have previously bound HDMI to VP0, so we need to bind MIPI to VP1;
- When the display device node in dts is open, the ports corresponding to VP0 and VP1 of the display interface will be open, so you need to close the port corresponding to the VOP that is not needed;

```c
&dsi0_in_vp0 {
    status = "disabled";
};
&dsi0_in_vpl {
    status = "disabled";
};
&dsil_in_vp0 {
    status = "disabled";
};
&dsil_in_vpl {
    status = "okay";
};
```

## 4.4 Open the boot logo

- If the u-boot logo is not turned on, the kernel stage cannot display the boot logo, and you can only wait until the system boots to see the image displayed by the application. Enable route_dsi1 in dts to support u-boot logo display; at the same time, you need to turn off the currently unused route_dsi0.
- On this platform (supporting three VOPs), it is necessary to note that the VOP specified by connect in the code configuration below must be consistent with the VOP bound to MIPI DSI1, otherwise problems such as flower screen may occur;

```c
&route_dsi0 {
    status = "disabled";
};
&route_dsil {
    status = "okay";
    connect = <&vpl_out_dsil>;
};
```

## 4.5 Configuration phy

- It can be obtained from the current configuration of dsi1 that the phys configuration is video_phy1, so video_phy1 needs to be enabled;

``` c
&video_phyl {
    status = "okay";
};
```

## 4.6 Configuring PANEL

- The main parameters of PANEL are configured as follows:

```c
&dsi1 {
    status = "okay";
    dsi1_panel: panel@0 {
        status = "okay";
        compatible = "simple-panel-dsi";
        reg = <0>;
        backlight = <&backlight1>;
        reset-gpios = <&gpio3 17 GPIO_ACTIVE_LOW>;
        power-supply = <&vcc3v3_lcd0_n>;
        reset-delay-ms = <60>;
        enable-delay-ms = <60>;
        prepare-delay-ms = <60>;
        unprepare-delay-ms = <60>;
        disable-delay-ms = <60>;
        dsi,flags = <(MIPI_DSI_MODE_VIDEO | MIPI_DSI_MODE_VIDEO_BURST | MIPI_DSI_MODE_LPM)>;
        dsi,format = <MIPI_DSI_FMT_RGB888>;
        dsi,lanes = <4>;
        panel-init-sequence = [
            29 00 04 FF 98 81 03
            23 00 02 38 3C
            05 78 01 11
            05 78 01 29
        ];
        panel-exit-sequence = [
            05 00 01 28
            05 00 01 10
        ];
    };
};
```

Parameter Description:

dsi,flags Description:

- MIPI_DSI_MODE_VIDEO MIPI_DSI_MODE_VIDEO_BURST indicates Video Burst Mode;
- MIPI_DSI_MODE_LPM indicates that initialization sequence is sent in LP mode by default;
- MIPI_DSI_MODE_EOT_PACKET indicates that EOTP feature is turned off;

dsi,format:Pixel Format

dsi,lanes: Lane Number(1~8), greater than 4 indicates Dual-channel MIPI-DSI Panel;

panel-init-sequence: power-on initialization sequence of the screen, refer to command description below for specific parameter configuration mode;

panel-exit-sequence: power-down initialization sequence of the screen, refer to command description below for specific parameter configuration mode;

backlight: backlight selection;

reset-gpio: Reset gpio;

power-supply: power supply configuration of the screen;

```c
disp_timings1: display-timings {
    native-mode = <&dsi1_timing0>;
    dsi1_timing0: timing0 {
        clock-frequency = <71750160>;
        hactive = <800>;
        vactive = <1280>;
        hfront-porch = <52>;
        hsync-len = <8>;
        hback-porch = <48>;
        vfront-porch = <15>;
        vsync-len = <6>;
        vback-porch = <16>;
        hsync-active = <0>;
        vsync-active = <0>;
        de-active = <0>;
        pixelclk-active = <1>;
    };
};

```

hactive: lcd width;

vactive: lcd high;

clock-frequency=htotal * vtotal * fps

 =（hactive + hfront-porch + hsync-len + hback-porch） * （vactive + vfront-porch + vsync-len + vback-porch）* fps

- The initialization sequence command configuration description of the screen:

```c

panel-init-sequence = [
    29 00 04 FF 98 81 03
    ...
    23 00 02 38 3C
    ...
    05 78 01 11
    05 78 01 29
];
```

Format Description: The header has 3 bytes (hexadecimal), representing Data Type, Delay, Payload Length respectively.

The data starting with the fourth byte represents a Payload of Length.

The first command parses as follows:

```c
29 00 04 FF 98 81 03
```

Data Type：0x29 (DCS Long Write)

Delay：0x00 (0 ms)

Payload Length：0x04 (4 Bytes)

Payload：0xff 0x98 0x81 0x03

The final command parses as follows:

```c
05 78 01 29
```

Data Type：0x05 (DCS Short Write, no parameters)

Delay：0x78 (120 ms)

Payload Length：0x01 (1 Bytes)

Payload：0x29

Some common Data types are:

```c
0x03: Binary encoding 00 0011, instruction type is Generic Short WRITE without parameters, data type is Short
‌0x13: Binary encoding 01 0011, instruction type is Generic Short WRITE with 1 parameter, data type is Short
‌0x23: Binary encoding 10 0011, instruction type is Generic Short WRITE with 2 parameters, data type is Short
‌0x29: Binary encoding 10 1001, instruction type is Generic Long Write, data type is Long
```

## 4.7 FAQ

1. How to enable EOTP (EoT packet) features:

   DSI driver determines whether EOTP is enabled according to MIPI_DSI_MODE_EOT_PACKET, if flags are not

   Configure MIPI_DSI_MODE_EOT_PACKET, enable EOTP feature, if flags are configured

   MIPI_DSI_MODE_EOT_PACKET, turn off EOTP feature.

2. How do I enable discontinuous clocks?

   DSI driver judges whether to enable discontinuous clock according to MIPI_DSI_CLOCK_NON_CONTINUOUS. If flags is not configured with MIPI_DSI_CLOCK_NON_CONTINUOUS, it indicates continuous clock. If flags is configured with MIPI_DSI_MODE_EOT_PACKET, it indicates discontinuous clock.

3. If the mipi screen cannot be displayed, first check the backlight configuration, then confirm the lcd power supply, check the initialization parameter 

4. After checking the above points, if it is still abnormal, you need to grasp the complete log for analysis, and export the fdt of the board for analysis to see if the current screen configuration is effective.

# 5 LVDS configuration

The configuration of RK platform lvds in uboot stage is also obtained through dtsi, and no additional modifications are required. The path to the dtsi configuration file for the kernel section is as follows:

RK356X_A11/sunsea/project_sunsea/SIM8668/kernel/arch/arm64/boot/dts/rockchip/simcom/lvds.dtsi

For attributes and attribute descriptions, refer to document:

RK356X_A11/kernel/Documentation/devicetree/bindings/display/rockchip/rockchip-lvds.txt

## 5.1 Schematic confirmation

Generally, it is necessary to confirm whether the power supply of lvds screen, pin connection, RST pin and backlight control part are correct through schematic diagram, which are used for subsequent configuration, and confirm whether the connection of each lane is correct.

## 5.2 Enable LVDS

Code path: RK356X_A11/sunsea/project_sunsea/SIM8668/kernel/arch/arm64/boot/dts/rockchip/simcom/lvds.dtsi

```c
&lvds {
	status = "okay";
};
```

## 5.3 Binding VOP

1. In each platform of RK, the image data output by various display interfaces (HDMI, MIPI, etc.) comes from VOP;
2. As can be seen from the VOP configuration diagram in Section 2.3, LVDS can be tied to VP1 or VP2. On the SIM8668 project, we have previously bound HDMI to VP0 and MIPI to VP1, so LVDS can only be bound to VP2 for the time being;
3. When the display device node in dts is open, the ports corresponding to VP1 and VP2 of the display interface will be open, so you need to close the port corresponding to the VOP that is not needed;

```c
&lvds_in_vp2 {
	status = "okay";
};
```

## 5.4 Open the boot logo

- If the u-boot logo is not turned on, the kernel stage cannot display the boot logo, and you can only wait until the system boots to see the image displayed by the application. Enable route_lvds1 in dts to support u-boot logo display;
- On this platform (supporting three VOPs), it should be noted that the VOP specified by connect in the code configuration below must be consistent with the VOP bound to LVDS, otherwise problems such as color screen may occur;

```c
&route_lvds {
	connect = <&vp2_out_lvds>;
	status = "okay";
};
```

## 5.5 Configuration phy

- You can get from the current configuration of lvds, phys configuration is video_phy0, so you need to enable video_phy0;

```c
&video_phy0 {
	status = "okay";
};
```

## 5.6 Configuring PANEL

- The main parameters of PANEL are configured as follows:

```c
/ {
    panel: panel {
        compatible = "simple-panel";
        backlight = <&backlight2>;
        power-supply = <&vdd_lcd0_1v8>;
        enable-delay-ms = <20>;
        prepare-delay-ms = <20>;
        unprepare-delay-ms = <20>;
        disable-delay-ms = <20>;
        bus-format = <MEDIA_BUS_FMT_RGB888_1X7X4_SPWG>;
        width-mm = <164>;
        height-mm = <100>;
        reset-gpios = <&gpio0 16 GPIO_ACTIVE_LOW>;

        display-timings {
            native-mode = <&timing0>;

            timing0: timing0 {
                clock-frequency = <50834000>;
                hactive = <1024>;
                vactive = <600>;
                hback-porch = <160>;
                hfront-porch = <24>;
                vback-porch = <24>;
                vfront-porch = <5>;
                hsync-len = <136>;
                vsync-len = <6>;
                hsync-active = <0>;
                vsync-active = <0>;
                de-active = <0>;
                pixelclk-active = <0>;
            }; 
        }; 
    }; 
}; 
```

Parameter Description:

bus,format: Data mapping method of LVDS signal

- MEDIA_BUS_FMT_RGB666_1X7X3_SPWG：“vesa-18”
- MEDIA_BUS_FMT_RGB888_1X7X4_SPWG：“vesa-24”
- MEDIA_BUS_FMT_RGB888_1X7X4_JEIDA：“jeida-24”
- MEDIA_BUS_FMT_RGB888_1X7X3_JEIDA：“jeida-18”

backlight: backlight selection;

reset-gpio: Reset gpio;

power-supply: power supply configuration of the screen;

hactive: lcd width;

vactive: lcd high;

clock-frequency=htotal * vtotal * fps

 =（hactive + hfront-porch + hsync-len + hback-porch） * （vactive 		+ vfront-porch + vsync-len + vback-porch）* fps

# 6 EDP Configuration

The configuration of RK platform edp in uboot stage is also obtained through dtsi, and no additional modifications are required. The path to the dtsi configuration file for the kernel section is as follows:

RK356X_A11/sunsea/project_sunsea/SIM8668/kernel/arch/arm64/boot/dts/rockchip/simcom/edp.dtsi

For attributes and attribute descriptions, refer to document:

RK356X_A11/kernel/Documentation/devicetree/bindings/display/rockchip/analogix_dp-rockchip.txt

## 6.1 Schematic confirmation

Generally, it is necessary to confirm the power supply, reset pin and backlight control part of edp through schematic diagram, which are used for subsequent configuration, and confirm whether the connection of each lane is correct.

## 6.2 uboot configuration, modify power supply

Code Path:

RK356X_A11/sunsea/project_sunsea/SIM8668/u-boot/drivers/video/drm/rockchip_display.c

rockchip_show_logo function Enable power before displaying logo.

```c

int rockchip_show_logo(void)
{
    struct display_state *s;
    int ret = 0;
    struct gpio_desc edp_enable;

    /* usbl GPIO0_C5 is gpio21 */
    ret = dm_gpio_lookup_name("21", &edp_enable);
    if (!ret) {
        dm_gpio_request(&edp_enable, "edp_power_en");
        dm_gpio_set_dir_flags(&edp_enable, GPIOD_IS_OUT);
        dm_gpio_set_value(&edp_enable, 1);
        printf("power en edp_power_en\n");
    } else {
        printf("power en edp_power_en fail\n");
    }

    msleep(500);
    list_for_each_entry(s, &rockchip_display_list, head) {
        s->logo.mode = s->logo.mode;
```

## 6.3 Enable LVDS

Code Path:

RK356X_A11/sunsea/project_sunsea/SIM8668/kernel/arch/arm64/boot/dts/rockchip/simcom/edp.dtsi

```c
&edp {
    status = "okay";
};
```

## 6.4 Binding VOP

- In each platform of RK, the image data output by various display interfaces (HDMI, MIPI, etc.) comes from VOP;
- As can be seen from the VOP configuration diagram in Section 2.3, EDP can be bound to VP0 or VP1. On the SIM8668 project, according to the project requirements, we have a version of HDMI+EDP+LVDS, so EDP can only be bound to VP1 temporarily;
- When the display device node in dts is open, the ports corresponding to VP1 and VP2 of the display interface will be open, so you need to close the port corresponding to the VOP that is not needed;

```c
&edp_in_vp0 {
    status = "disabled";
};

&edp_in_vp1 {
    status = "okay";
};
```

## 6.5 Open the boot logo

- If the u-boot logo is not turned on, the kernel stage cannot display the boot logo, and you can only wait until the system boots to see the image displayed by the application. Turn route_edp on in dts to support u-boot logo display;
- On this platform (supporting three VOPs), it should be noted that the VOP specified by connect in the code configuration below must be consistent with the VOP bound to LVDS, otherwise problems such as color screen may occur;

```c
&route_edp {
    connect = <&vpl_out_edp>;
    status = "okay";
};
```

## 6.6 Configuration phy

- You can get from the current configuration of lvds, phys configuration is video_phy0, so you need to enable video_phy0;

```c
&edp_phy {
    status = "okay";
};
```

## 6.7 Configuring PANEL

- The main parameters of PANEL are configured as follows:

```c
edp_panel {
    compatible = "simple-panel";
    backlight = <&backlight>;
    power-supply = <&vcc3v_led1_n>;
    prepare-delay-ms = <1>;
    enable-delay-ms = <12>;
    unprepare-delay-ms = <120>;
    disable-delay-ms = <12>;
    display_timings {
        native-mode = <&edp_timing>;

        edp_timing: timing0 {
            clock-frequency = <152560000>;
            hactive = <1020>;
            vactive = <1000>;
            hfront-porch = <40>;
            hsync-len = <144>;
            hback-porch = <80>;
            vfront-porch = <3>;
            vsync-len = <68>;
            vback-porch = <9>;
            hsync-active = <0>;
            vsync-active = <0>;
            de-active = <0>;
            pixelclk-active = <0>;
        };
    };

    port {
        panel_in_edp: endpoint {
            remote-endpoint = <&edp_out_panel>;
        };
    };
};
```

Parameter Description:

backlight: backlight selection;

power-supply: power supply configuration of the screen;

hactive: lcd width;

vactive: lcd high;

clock-frequency=htotal * vtotal * fps

 =（hactive + hfront-porch + hsync-len + hback-porch） * （vactive + vfront-porch + vsync-len + vback-porch）* fps

The remote-endpoint attribute under the panel_in_edp node points to the specific endpoint of the edp phy. At the same time, the panel_in_edp node is also referenced by the remote-endpoint attribute of the corresponding edp phy, so that it can be mapped one by one.

## 6.8 Configuring Backlighting

At present, the edp screen of evb uses pwm2 backlight, and the configuration is as follows:

```c
&pwm2 {
    status = "okay";
};

&backlight {
    pwms = <&pwm2 0 250000 0>;
};
```

## 6.9 Is hot plug supported?

- If the edp screen supports hot plug, you need to configure hot plug detection gpio, which is configured through the attribute hpd-gpios; if it does not support hot plug, you need to delete this attribute and configure it as force-hpd.

## 6.10 EDP DEBUG

- Enable panel self-test mode

  If the above two checks are completed, aux can obtain relevant information, but it is still not displayed, you need to enable the self-test mode of the screen.

```c
&edp {
    force-hpd;
    status = "okay";
    //delete - property hpd - gpios;
    panel - self - test;
    port {
        reg = <1>;
        edp_out_panel: endpoint@0 {
            remote - endpoint = <&panel_in_edp>;
        };
    };
};

```

If the panel can be displayed after enabling the panel self-test mode, then the panel has been working normally, aux can be positive

Frequent correspondence.

- Enable edp self-test mode

```c
&edp {
    force-hpd;
    status = "okay";
    //delete - property hpd - gpios;
    analogix,video - bist - enable;
    ports {
        port@1 {
            reg = <1>;
            edp_out_panel: endpoint@0 {
                remote - endpoint = <&panel_in_edp>;
            };
        };
    };
};
```

If the panel can be displayed after enabling edp self-test mode, it means that the panel has worked normally, aux can communicate normally, and edp main link is normal.

- Confirm current connection status:

```c
rk3568_r:/# cat /sys/class/drm/card0-eDP-1/status
connected
```

If status is disconnected, either hpd is low or aux cannot communicate.

If edp display log is supported, check whether the corresponding aux communication is successful in the boot log. If successful, the following log information will appear.

```c
Maximum visible display size: 34 cm x 19 cm
Power management features: active off, suspend, standby
Estabilished timings:
Standard timings:
		1920x1080 		59 Hz(detailed)
Monitor ID:HKC MY
Monitor ID:PC156CS01'1
edpefe0c0000: detailed mode clock 152560 kHz, flags[a]
	H:1920 1968 2000 2192
	V:1080 1083 1089 1160
bus_format:100e
VOP update mode to:1920x1080p0, type:eDPO for VPl

```

If Aux communication fails, it is generally due to the equipment not working properly or the screen line problem, resulting in no response. The equipment should be checked.

Whether the power supply of the terminal is normal and the cable is arranged.

- Confirm the current status of the displayed route:

```c
r3568_r:/ # cat /d/dri/0/summary
Video Port0: DISABLED
Video Port1: ACTIVE
  Connector: eDP-1
    bus_format[100a]: RGB888_1X24
    overlay_mode[0] output_mode[f] color_space[0]
  Display mode: 1024x768p60
    clk[65000] real_clk[65000] type[40] flag[a]
    H: 1024 1048 1184 1344
    V: 768 771 777 806
  Esmart0-win0: ACTIVE
    win_id: 3
    format: AB24 little-endian (0x34324241) SDR[0] color_space[0] glb_alpha[0xff]
    rotate: xmirror: 0 ymirror: 0 rotate_90: 0 rotate_270: 0
    csc: y2r[0] r2y[0] csc mode[0]
    zpos: 2
    src: pos[0, 0] rect[1024 x 48]
    dst: pos[0, 84] rect[1024 x 48]
    buf[0]: addr: 0x0000000000e04000 pitch: 4096 offset: 0
  Esmart0-win1: ACTIVE
    win_id: 3
    format: AB24 little-endian (0x34324241) SDR[0] color_space[0] glb_alpha[0xff]
    rotate: xmirror: 0 ymirror: 0 rotate_90: 0 rotate_270: 0
    csc: y2r[0] r2y[0] csc mode[0]
    zpos: 2
    src: pos[0, 0] rect[1024 x 96]
    dst: pos[0, 588] rect[1024 x 96]
    buf[0]: addr: 0x0000000000d14000 pitch: 4096 offset: 0
  Cluster0-win0: ACTIVE
    win_id: 4
    format: XB24 little-endian (0x34324258) [AFBC] SDR[0] color_space[0] glb_alpha[0xff]
    rotate: xmirror: 0 ymirror: 0 rotate_90: 0 rotate_270: 0
    csc: y2r[0] r2y[0] csc mode[0]
    zpos: 0
    src: pos[0, 265] rect[1280 x 750]
    dst: pos[0, 84] rect[1024 x 600]
    buf[0]: addr: 0x0000000000e64000 pitch: 5120 offset: 0
  Cluster0-win1: ACTIVE
    win_id: 5
    format: AB24 little-endian (0x34324241) [AFBC] SDR[0] color_space[0] glb_alpha[0xff]
    rotate: xmirror: 0 ymirror: 0 rotate_90: 0 rotate_270: 0
    csc: y2r[0] r2y[0] csc mode[0]
    zpos: 1
```

You can view information about the resolution of the screen.

- After checking the above points, if it is still abnormal, you need to grasp the complete log for analysis, and export the fdt of the board for analysis to see if the current screen configuration is effective.
