# SIM866X_AP_Screen Rotation Application User Guide

## **Version History**

| **versions**|**date**|**author**|**remark**|
| -------- | -------- | -------- | -------- |
| 1.00     |2026.3.2  |  Xu Jiahui| The first version   |

## 1 Introduction

This document describes how to set the screen rotation direction on the SIM866x module through system properties.

## 2 Android system portrait forced conversion landscape screen

Add system attributes to the corresponding project mk file

For example:
sunsea/project_sunsea/SIM866*_* */device/rockchip/rk356 x/rk356x.prop

```
ro.vendor.rk_sdk=1

#close rescue mode
persist.sys.disable_rescue=true

# Add by xujiahui for set orientation to 90
persist.panel.orientation=90
```

System default vertical screen:
```
persist.panel.orientation= 0  
```

System default 90 degrees horizontal screen:
```
persist.panel.orientation= 90
```
System default 180 degrees horizontal screen:
```
persist.panel.orientation= 180 
```
System default 270 degrees horizontal screen:
```
persist.panel.orientation= 270 
```
According to the above modification, you can modify the desired system display effect by corresponding angle.
