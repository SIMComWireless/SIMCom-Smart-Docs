# SIM866X_SENSOR Migration Documentation

## Version History

| **versions**|**date**|**author**|**remark**|
| :------- | :-------- | :------- | :------- |
| 1.00     |2026.3.16 | Liu Yi|The first version|

## 1 Introduction

This article mainly introduces the architecture and basic configuration of rockchip sensors; the latest kernel 4.4 has made some modifications to the architecture of sensors to meet the requirements of android cts and vts testing; adding a new sensor driver requires some adaptation work.

## 2 Android sensors architecture

### 2.1 Overall architecture of Android sensor system

The complete hierarchical call process of sensor data in Android system from hardware to application is divided into 5 layers from top to bottom:

**APPLICATIONS**

- The top layer is the specific**Application** (such as step-counting, compass and other apps), which is directly oriented to the user and realizes functions through sensors.

**APPLICATION FRAMEWORK**

- **SensorManager**: The entry class for the application to acquire sensor services, responsible for managing all sensors, registering/deregistering listening, and acquiring sensor data.
- **Sensor JNI**: Bridge between Java layer and underlying Native layer, converting Java calls to C/C++ implementations.

**LIBRARIES**

- **Sensor Manager (Native)**: The underlying implementation of JNI, interacting with system services.
- **Sensor Service**: System-level service that manages the status, permissions, and data distribution of all sensors in a unified manner.
- **Sensor HAL (Hardware Abstraction Layer)**: A hardware abstraction layer that isolates the Linux kernel from upper layer services and provides standardized interfaces so that the upper layer does not need to care about specific hardware driver details.

**LINUX KERNEL (Linux kernel layer)**

- **Input Subsystem**: Linux input subsystem, unified processing of events for various input devices (including sensors).
- **Event Dev**: Event device node that exposes sensor event interfaces to upper layers.
- **Accelerometer Driver**: Driver for specific sensors such as accelerometers.
- **I2C driver**: I2C bus driver responsible for communicating with sensor hardware.

**HARDWARE (Hardware Layer)**

- **Accelerometer**: Accelerometer sensor hardware, responsible for collecting physical data.
- **I2C Controller**: I2C controller that communicates with sensor chips as bus master.

### 2.2 Sensor kernel-driver interaction model

**Upper Control Instruction Flow (Down)**

The upper layer sends control commands to Sensor-dev via the ioctl system call:

- `active`: Activates/hibernates sensors.
- `Batch/setdelay`: Sets the data batch mode or sampling delay.
- When`Sensor-dev`receives the command, it calls`Sensor-chip driver`via`Ctl_ops` perform hardware configuration.

**Bottom Data Reporting Flow (Up)**

- `Sensor-chip driver`obtains the raw data collected by the hardware through`Interrupt` or`poll`.
- The driver encapsulates the data as an**Input event**and submits it to`Sensor-dev`.
- The upper layer listens to`Sensor-dev`through`poll`system call to read sensor event data.

**Driver registration**

- `Sensor-chip driver`registers itself with`Sensor-dev` through`register`interface when starting, completing device initialization and function exposure.

## 3 Rockchip sensors HAL

Code path: hardware/rockchip/sensor/st, supports the latest SENSORS_DEVICE_API_VERSION_1_3 android sensors hal API interface.

The interfaces implemented include:

- get_sensors_list -Returns a list of all sensors
- activate -starts or stops the sensor
- batch -Sets sensor parameters such as sample rate and maximum report delay
- setDelay -For version 1.0 of HAL only, sets the sample rate of the specified sensor
- flush -Flushes the FIFO of the specified sensor and reports a flush completion event upon completion
- poll -Returns available sensor events

The flush interface is not really implemented, because it has to be specific to hardware fifo support, because we support
There are many sensor models, so it is impossible to achieve a unified interface implementation, so here is just a bypass, directly return
META_DATA_FLUSH_COMPLETE type data.

## 4 Introduction to Rockchip sensors kernel driver

Code path: kernel/drivers/input/sensors, where sensor-dev.c is the core code, integration
There are different types of sensors, including accel, gyro, lsensor, psensor, compass, etc.

## 5 Add a sensor driver process

Here, for example, add one of the most commonly used gsensors, similar to other types of sensors. Android hal does not need any modification, hal is configured according to +-2G Gsensor range, adc 16bit, that is, the corresponding value of 1G is 16384; so the driver layer adds gsensor driver, and the reported value needs to be adapted according to the range and adc accuracy. The steps are as follows, taking mpu6500 acc as an example:

### 5.1 Adding sensor id

Add sensor_id to sensor-dev.c, where the "mpu6500_acc" field corresponds to the compatible field in dts (old version mode)

For example:

```c
static const struct i2c_device_id sensor_id[] = {
    /*angle*/
    {"angle_kxtik", ANGLE_ID_KXTIK},
    {"angle_lis3dh", ANGLE_ID_LIS3DH},
    /*gsensor*/
    {"gsensor", ACCEL_ID_ALL},
    {"gs_mma8452", ACCEL_ID_MMA845X},
    {"gs_kxtik", ACCEL_ID_KXTIK},
    {"gs_kxtj9", ACCEL_ID_KXTJ9},
    {"gs_lis3dh", ACCEL_ID_LIS3DH},
    {"gs_mma7660", ACCEL_ID_MMA7660},
    {"gs_mxc6225", ACCEL_ID_MXC6225},
    {"gs_dmard10", ACCEL_ID_DMARD10},
    {"gs_lsm303d", ACCEL_ID_LSM303D},
    {"gs_mc3230", ACCEL_ID_MC3230},
    {"mpu6880_acc", ACCEL_ID_MPU6880},
    {"mpu6500_acc", ACCEL_ID_MPU6500}, // Add new sensor_id
    {"lsm330_acc", ACCEL_ID_LSM330},
    {"bma2xx_acc", ACCEL_ID_BMA2XX},
    {"gs_stk8baxx", ACCEL_ID_STK8BAXX},
    /*compass*/
    {"compass", COMPASS_ID_ALL},
    {"ak8975", COMPASS_ID_AK8975},
    {"ak8963", COMPASS_ID_AK8963},
    {"ak09911", COMPASS_ID_AK09911},
    {"mmc314x", COMPASS_ID_MMC314X},
    // ...
};
```

This method is the same as in older versions of RK code. The new version code has been updated to place i2c_device_id in the sensor driver declaration and release the registration function interface by calling sensor-dev.c

```c
EXPORT_SYMBOL(sensor_register_device);
```

Realize sensor registration. Related implementations are as follows

```c
// 声明与定义
static const struct i2c_device_id accl_mpu6500_id[] = {
    {"mpu6500_acc", ACCEL_ID_MPU6500},  // When there is no compatible option, the secondary method for binding the device via I2C driver is ACCEL_ID_MPU6500, which is the device identification number.
    {}
};

static int accl_mpu6500_probe(struct i2c_client* client,
    const struct i2c_device_id* devid)
{
    return sensor_register_device(client, NULL, devid, &accl_mpu6500_ops);
}
```

ACCEL_ID_MPU6500 is defined in include/linux/sensor-dev.h:

```c
enum sensor_id {
	ID_INVALID = 0,
    ...
    ACCEL_ID_MPU6500, // Newly added "sensor_id", corresponding to other sensors, simply add the corresponding id. The general naming format is "SENSOR_TYPE_ID_SENSOR_NAME".
    ...
    SENSOR_NUM_ID,
};
```

### 5.2 Add gsensor chip driver

The requested URL/input/sensors/accel/was not found on this server.

1) Implementation of gsensor_mpu6500_ops Example:

```c
struct sensor_operate gsensor_mpu6500_ops = {
	.name = "mpu6500_acc", //Consistent with the names defined in sensor-dev.c/i2c_device_id
	.type = SENSOR_TYPE_ACCEL, //The type of sensor is gsensor.
    .id_i2c = ACCEL_ID_MPU6500, // Defined in include/linux/sensor-dev.h
	.read_reg = MPU6500_ACCEL_XOUT_H, //The starting register address for reading gsensor data
	.read_len = 6, //The number of bytes of gsensor data that need to be read
	.id_reg         = MPU6500_WHOAMI, //Chip Unique ID Register
	.id_data        = MPU6500_DEVICE_ID, //Chip Unique ID
	.precision       = MPU6500_PRECISION, //The number of bits for sampling the gsensor data by the ADC
	.ctrl_reg = MPU6500_PWR_MGMT_2, //The register address used to enable the gsensor
	.int_status_reg   = MPU6500_INT_STATUS, //Interrupt status register address
	.range               = {-32768, 32768}, //Range: Here, it indicates that the reported range is ±2G.
	.trig = IRQF_TRIGGER_HIGH |IRQF_ONESHOT, // IRQ Type
	.active               = sensor_active, //Used for controlling the gsensor
	.init                 = sensor_init, //Used for initializing the gsensor
	.report = sensor_report_value, //Used for reporting gsensor data
};
```

2) Register sensor driver to sensor-dev.c (old version scheme)

```c
static struct sensor_operate *gsensor_get_ops(void)
{
	return &gsensor_mpu6500_ops;
}
static int __init gsensor_mpu6500_init(void)
{
	struct sensor_operate *ops = gsensor_get_ops();
	int type = ops->type;
	return sensor_register_slave(type, NULL, NULL, gsensor_get_ops);
}
static void __exit gsensor_mpu6500_exit(void)
{
    struct sensor_operate *ops = gsensor_get_ops();
    int type = ops->type;
    sensor_unregister_slave(type, NULL, NULL, gsensor_get_ops);
}

Kernel driver install:

module_init(gsensor_mpu6500_init);
module_exit(gsensor_mpu6500_exit);
```

The new version has a separate sensor device driver, which is registered by calling the interface provided by sensor-dev.c, so in fact we only need to port the sensor driver into the code

```c
// Driver Migration:
1. Place the driver in the specified path: kernel/drivers/input/sensors/. RK will create folders based on the type of the sensor driver and simply place the corresponding driver in the folder.
2. Add compilation options
Makefile: obj-$(CONFIG_MPU6500_ACC)	+= mpu6500_acc.o
Kconfig: config MPU6500_ACC
tristate "Sensor mpu6500_acc"
default n // Set to 'y' when enabled help
To have support for your specific gsesnor you will have to
select the proper drivers which depend on this option.
3. Add the compilation switch in sunsea/project_sunsea/SIM8666/ProjectConfig.mk
CONFIG_MPU6500_ACC=n // By default, it is not loaded. Change it to y for loading. This is the recommended method.
```

### 5.3 Modify gsensor range and report data conversion

Different manufacturers of gsensor will have differences in range and accuracy, in order to be compatible with different manufacturers of gsensor chips, while not modifying the android hal layer code, only modifying the driver, to simplify customer development purposes, we must convert different ranges, different precision gsensor data into a unified standard report to hal layer code. As mentioned in the previous section, the HAL layer reports gsensor data with a range of +-2G, adc 16bit, and different gsensors with this standard need to be adapted. Examples are as follows:

1）mpu6500   +- 2G   16bit adc

Mpu6500 adc bit number is 16 bits, the initialization time set range is +-2G, then the register read out the value range is: +-32768, and match with the upper hal layer, so as long as the register read out the value can be reported directly;

```c
struct sensor_operate gsensor_mpu6500_ops = {
    .name                           = "mpu6500_acc",
    .type                           = SENSOR_TYPE_ACCEL,
    .id_i2c						  = ACCEL_ID_MPU6500,
    .read_reg                       = MPU6500_ACCEL_XOUT_H,
    .read_len                       = 6,
    .id_reg                         = MPU6500_WHOAMI,
    .id_data                        = MPU6500_DEVICE_ID,
    .precision                      = MPU6500_PRECISION,
    .ctrl_reg                       = MPU6500_PWR_MGMT_2,
    .int_status_reg                 = MPU6500_INT_STATUS,
    .range                          = {-32768, 32768},
    .trig                           = IRQF_TRIGGER_HIGH|IRQF_ONESHOT,
    .active                         = sensor_active,
    .init                           = sensor_init,
    .report                         = sensor_report_value,
};
```

2) If the range and adc accuracy of gsensor are not +-2G and 16bit, corresponding conversion is required.

```c
Assuming the measurement range is ±XG, and ADC is Ybit, the conversion relationship is as follows:
#define XXX_RANGE        16384*X
#define XXX_PRECISION    Y
#define XXX_BOUNDARY     (0x1 << (XXX_PRECISION - 1))
#define XXX_GRAVITY_STEP (XXX_RANGE / XXX_BOUNDARY)

struct sensor_operate gsensor_xxx_ops = {
    .name                           = "xxx_acc",
    .type                           = SENSOR_TYPE_ACCEL,
    .id_i2c                         = ACCEL_ID_XXX,
    .read_reg                       = XXX_ACCEL_XOUT_H,
    .read_len                       = 6,
    .id_reg                         = XXX_WHOAMI,
    .id_data                        = XXX_DEVICE_ID,
    .precision                      = Y,
    .ctrl_reg                       = XXX_PWR_MGMT_2,
    .int_status_reg                 = XXX_INT_STATUS,
    .range                          = {-16384*X, +16384*X},
    .trig                           = (IRQF_TRIGGER_LOW | IRQF_ONESHOT),
    .active                         = sensor_active,
    .init                           = sensor_init,
    .report                         = sensor_report_value,
};
```

Report value conversion: report_data = read_data* XXX_GRAVITY_STEP; //read_data is the value read from the actual register, and report_data is the value that can be reported to the hal layer after conversion

### 5.4 Modify gsensor layout direction

For the definition of coordinate axis in Android system, refer to RK official document:*Rockchip_Developer_Guide_Sensors_CN.pdf* Chapter 6.4

Sensor-dev.c defines the layout value, corresponding to different matrices, you can see the sensor-dev.c code query, so, the adaptation direction can be modified by modifying the layout value in the dts to select the corresponding conversion matrix, to achieve the direction change, to adapt to the requirements of android.

as follow:

```c
lsm330_accel@1e {
	status = "disabled";
	compatible = "lsm330_acc";
	pinctrl-names = "default";
	pinctrl-0 = <&lsm330a_irq_gpio>;
	reg = <0x1e>;
	irq-gpio = <&gpio1 22 IRQ_TYPE_EDGE_RISING>; //gpio1 c6
	type = <SENSOR_TYPE_ACCEL>;
	irq_enable = <1>;
	poll_delay_ms = <30>;
	power-off-in-suspend = <1>;
	layout = <4>;
};
```

### 5.5 Meaning of other values in dts

```c
lsm330_accel@1e {
    status = "disabled";
    compatible = "lsm330_acc"; // Match with the definition of sensor_id in sensor-dev.c
    pinctrl-names = "default"; //The relevant pinctrl definitions, here mainly define the interrupt pins.
    pinctrl-0 = <&lsm330a_irq_gpio>;
    reg = <0x1e>;  // i2c address
    irq-gpio = <&gpio1 22 IRQ_TYPE_EDGE_RISING>;  // IRQ pin and type
    type = <SENSOR_TYPE_ACCEL>;  // The type of sensor must not be mistaken.
    /* Whether to use interrupt mode or not. If you want to use CTS or VTS, it is recommended to use polling. The polling mode in sensor-dev.c can already meet the requirements of CTS testing. If using interrupt mode, the gsensor chip driver needs to configure the sampling rate properly.*/
    irq_enable = <1>;
    poll_delay_ms = <30>;
    layout = <4>;
    reprobe_en = <1>;  // If this value is 1, it indicates that after the sensor driver loading fails, the sensor driver will be attempted to be reloaded multiple times (the number of attempts is defined in the code).
};
```

### 5.6 Sensor-related macro configuration in Android

BoardConfig.mk www.example.com:

```c
# Sensors
BOARD_SENSOR_ST := true
BOARD_SENSOR_MPU_PAD := false
```

What types of sensors are supported? If not, configure false, otherwise the vts and cts tests will fail:

```c
BOARD_GRAVITY_SENSOR_SUPPORT := true
BOARD_COMPASS_SENSOR_SUPPORT := false
BOARD_GYROSCOPE_SENSOR_SUPPORT := false
BOARD_PROXIMITY_SENSOR_SUPPORT := false
BOARD_LIGHT_SENSOR_SUPPORT := true
BOARD_PRESSURE_SENSOR_SUPPORT := false
BOARD_TEMPERATURE_SENSOR_SUPPORT := false
```

6 Calibration of Gsensor and gyro

Gsensor and gyro calibration can be done through the command line, pcba test can also be calibrated, specific check
Look at the PCBA configuration.
Command line calibration method: Keep the machine level and still, enter the following command calibration:

```c
Gsensor: echo 1 > /sys/class/sensor_class/accel_calibration
GYRO : echo 1 > /sys/class/sensor_class/gyro_calibration
```

View calibration values:

```c
cat /sys/class/sensor_class/accel_calibration
cat /sys/class/sensor_class/gyro_calibration
```

If calibration values cannot be viewed, calibration failed, and the kernel log can be printed to determine the cause of failure. After successful calibration, the calibration value will be saved to the vendor storage of nand or emmc, and will not be erased. It will take effect automatically after startup.

## 6 FAQ

### 6.1 Gsensor function unavailable

#### 6.1.1 getevent View g-sensor no data reported

Investigation method:

1. Confirm whether the driver is registered, you can check whether there is an input device named gsensor through getevent; if not, you need to see why it is not registered, it may be a dts configuration problem, driver problem, i2c failure at the same time, chip id mismatch and other possible reasons, check the kernel log, specific analysis of the problem; if there is registration, go down.
2. Confirm whether the android layer related macro is open;
3. Confirm whether hal layer code works normally; print logcat after booting, search sensor field to view related log.

#### 6.1.2 Data values are incorrect

Getevent has data reported, the screen does not rotate or rotates irregularly;

In addition to checking whether the value is correct through the apk of sensor, you can also check whether the gsensor data reported by android is correct through commands:
		setprop sensor.debug.level 2

The command takes effect when the screen is manually closed and dormant. You can view the gsensor value reported by hal through logcat-s SensorsHal; when it is placed horizontally, the theoretically correct value should be +9.8 or-9.8 for one axis and 0 for the other two axes;

If the values are incorrect, verify that the data reporting conversion described in the previous section is correct.

### 6.2 Wrong direction

If the direction is wrong, please refer to section 5.4 above by modifying the layout value of dts.

