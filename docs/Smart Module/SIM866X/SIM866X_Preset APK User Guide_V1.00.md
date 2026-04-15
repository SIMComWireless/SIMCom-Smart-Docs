# SIM866X_Preset APK User Guide

## Version History

| **versions**|**date**     |**author**|**remark**|
| ------ | ---------- | ------ | ------ |
| 1.00   |Yang Huakun|2026.3.17|  The first version   |

## 1 Introduction

### 1.1 Purpose of this article

This article aims to guide developers to quickly and normatively complete APK system provisioning on Android 11 platform of SIM 866x module. With instructions on preset processes, key configuration files, and common considerations, developers can efficiently integrate applications into system images and ensure that they function properly and have complete permissions.

This article is not only applicable to the rapid integration scenario of no-source APK, but also covers key links such as permission adaptation, system signature difference processing, and debugging verification, helping to reduce the cost of troubleshooting in the integration process.

### 1.2 Terminology and description

The key point of built-in application is to configure the compilation file, i.e. Android.mk file. The relevant configuration items are described below:

| ***term***                             |***describe***                                                   |
| -------------------------------------- | ------------------------------------------------------------ |
| ***LOCAL\_PATH := \$(call my-dir)***   | This variable indicates where the source file is located in the development tree. Here, the macro function my-dir provided by the build system returns the path to the current directory (where the Android.mk file itself resides). |
| ***include \$(CLEAR\_VARS)***          | Supplied by the build system, the CLEAR\_VARS variable points to a special GNU Makefile that clears many LOCAL\_XXX variables, GNU Makefile does not clear LOCAL\_PATH |
| ***LOCAL\_MODULE := TestApp***         | Stores the names of the modules you want to build, each module name must be unique and contain no spaces. If not set, LOCAL\_PACKAGE\_NAME is used by default. |
| ***LOCAL\_PACKAGE\_NAME：= TestApp***|The compiled application name.                                         |
| ***LOCAL\_MODULE\_TAGS := optional***|Specifies the version under which the module will be compiled, possibly user, userdebug, optional. where optional is the default label, meaning that the module compiles under all versions. |
| ***LOCAL\_MODULE\_CLASS := APPS***     | Identify where the compiled module is placed, APPS means placed under the/system/app directory.  |
| ***LOCAL\_SRC\_FILES := TestApp.apk***|List of source files.                                                 |
| ***LOCAL\_PRIVILEGED\_MODULE***        | Determines the location of apk after compilation, if set to true, it is generated in/system/priv-app; if not set or false, it is generated in/system/app. |
| ***LOCAL\_CERTIFICATE***               | The following values can be set:<br /> PRESIGND: indicates that the signature used is the original application signature<br /> testkey: default<br /> platform: Apply permissions same as system|
| ***LOCAL\_PREBUILT\_JNI\_LIBS***       | Add the so path to the TestApp source code path libs/xxxx/xxxx.so.             |
| ***LOCAL\_MODULE\_SUFFIX***            | Reference to file suffix XXX.apk.                                          |
| ***include \$(BUILD\_PREBUILT)***      | Think of files as compiled projects.                                         |

## 2 Preset applications

**Prerequisite**: Please attach the 4 documents in the appendix as follows
├── Commom_APK_NoCode_Linux.py
├── EditMe.xml
├── generate_permission.sh
└── generate_somk.sh
Copy to the local, and generate the corresponding py, xml, sh format files; convenient to disassemble the APK below, generate the corresponding files, such as Android.mk, so library, permission, etc.

**Note**: The format of the file is recommended to be consistent with the following, otherwise it may fail to execute.

```
$ file *
Commom_APK_NoCode_Linux.py: Python script, UTF-8 Unicode text executable, with CRLF line terminators
EditMe.xml:                 XML 1.0 document, UTF-8 Unicode text, with CRLF line terminators
generate_permission.sh:     Bourne-Again shell script, UTF-8 Unicode text executable
generate_somk.sh:           Bourne-Again shell script, UTF-8 Unicode text executable
```

**Suggestion**: If copy a sh script of the same type in the source code, rename it generate_permission.sh, and then copy the source code in the appendix to the file.

### 2.1 Preset applications without source code into the system (non-uninstall)

#### 2.1.1 Copy Script and Apply to Project Root

Under Linux:

Copy the following 2 script files and apply TestApp.apk to the SIM8666 source root directory, taking TestApp as an example:

```
├── Commom_APK_NoCode_Linux.py
├── EditMe.xml
└── TestApp.apk
```

#### 2.1.2 Edit Apply Optional Configuration in EditMe.xml

The following edited attributes are automatically written into Android.mk generated in Section 2.1.3:

```xml
<configuration>
    <!-- The name of the required pre-installed APK. For example, if the application is named (TestApp.apk), modify the application name in the LOCAL_MODULE attribute; if it is named TestApp.Apk, simply fill in as TestApp。-->
    <string name="LOCAL_MODULE">TestApp</string>

    <!-- 1.If you want to package the application into the "/system/priv-app/" directory, then keep the default value as true.；
         2.If you want to package it into the "/system/app" directory, then change "true" to "false". -->
    <bool name="LOCAL_PRIVILEGED_MODULE">true</bool>

    <!-- 1.Default is 64
         2.If the so file is 32-bit while the system is 64-bit, please change "64" to "32"..
         3.If the system is 32-bit and the so file is also 32-bit, then please change the value of the LOCAL_MULTILIB attribute from 64 to 32. -->
    <integer name="LOCAL_MULTILIB">64</integer>
</configuration  >
```

#### 2.1.3 Running scripts

Under Linux:
Under the RK356X/Project root path, enter the command:

```bash
python Commom_APK_NoCode_Linux.py
```

An example of the generated effect is shown below.

```bash
$ python Commom_APK_NoCode_Linux.py
Start adding applications: TestApp
Obtaining the Application Path: /home/chenyubin/workspace_95/space/test/linux/TestApp.apk
/home/chenyubin/workspace_95/space/test/linux/TestApp

Start extracting files: /home/chenyubin/workspace_95/space/test/linux/TestApp.zip
Successfully decompressed to the path: /home/chenyubin/workspace_95/space/test/linux/TestApp

Creating an Application Path: /home/chenyubin/workspace_95/space/test/linux/packages/apps/TestApp
copy /home/chenyubin/workspace_95/space/test/linux/TestApp/lib to /home/chenyubin/workspace_95/space/test/linux/packages/apps/TestApp/lib
Copy /home/chenyubin/workspace_95/space/test/linux/TestApp.apk to /home/chenyubin/workspace_95/space/test/linux/packages/apps/TestApp/TestApp.apk
Contain .so list
[u'@lib/armeabi-v7a/libprivate_protobuf.so', u'@lib/armeabi-v7a/libopencv_world.so', u'@lib/armeabi-v7a/libse-lib.so', u'@lib/armeabi-v7a/libmmkv.so', u'@lib/armeabi-v7a/libYTFaceRetrievalInt.so', u'@lib/armeabi-v7a/libwechatbacktrace.so', u'@lib/armeabi-v7a/libblasV8.so', u'@lib/armeabi-v7a/libse_base.so', u'@lib/armeabi-v7a/libwechatxlog.so', u'@lib/armeabi-v7a/libyuv.so', u'@lib/armeabi-v7a/libhevcdec.so', u'@lib/armeabi-v7a/libjpeg-turbo1500.so', u'@lib/armeabi-v7a/libusb100.so', u'@lib/armeabi-v7a/libilink_network.so', u'@lib/armeabi-v7a/lib3000Driver.so', u'@lib/armeabi-v7a/libimisensor.so', u'@lib/armeabi-v7a/libYTFaceFeature.so', u'@lib/armeabi-v7a/libUVCCamera.so', u'@lib/armeabi-v7a/libBugly-rqd.so', u'@lib/armeabi-v7a/libwq_upgrade.so', u'@lib/armeabi-v7a/libYTProtectPrivacy.so', u'@lib/armeabi-v7a/libImiSdk.jni.so', u'@lib/armeabi-v7a/libmagic-lib.so', u'@lib/armeabi-v7a/libmatrix-pthreadhook.so', u'@lib/armeabi-v7a/liblinkid.so', u'@lib/armeabi-v7a/libPSLink.so', u'@lib/armeabi-v7a/libdilu-yuv.so', u'@lib/armeabi-v7a/libimilog.so', u'@lib/armeabi-v7a/libturingmfa.so', u'@lib/armeabi-v7a/libwarp_handle.so', u'@lib/armeabi-v7a/libYTFacePay.so', u'@lib/armeabi-v7a/libwechatnormsg_stl.so', u'@lib/armeabi-v7a/libwarp_v37.so', u'@lib/armeabi-v7a/libilink_xlog.so', u'@lib/armeabi-v7a/libnative-lib.so', u'@lib/armeabi-v7a/libwcdb.so', u'@lib/armeabi-v7a/libc++_shared.so', u'@lib/armeabi-v7a/libRKUpgrade.so', u'@lib/armeabi-v7a/libyuvsc.so', u'@lib/armeabi-v7a/libiminect.so', u'@lib/armeabi-v7a/libwq_transmit.so', u'@lib/armeabi-v7a/libdepth-tools.so', u'@lib/armeabi-v7a/libwxpay_tks.so', u'@lib/armeabi-v7a/libijksdl.so', u'@lib/armeabi-v7a/libsynthesizer.so', u'@lib/armeabi-v7a/librsjni.so', u'@lib/armeabi-v7a/libgraph_bitmap.so', u'@lib/armeabi-v7a/libtidclient.so', u'@lib/armeabi-v7a/libcross.so', u'@lib/armeabi-v7a/libRSSupport.so', u'@lib/armeabi-v7a/libxhook-lib.so', u'@lib/armeabi-v7a/libvpxJNI.so', u'@lib/armeabi-v7a/libImiCamera.jni.so', u'@lib/armeabi-v7a/libQBarMod.so', u'@lib/armeabi-v7a/libstlport_shared.so', u'@lib/armeabi-v7a/libmatrix-hookcommon.so', u'@lib/armeabi-v7a/libmarsstn.so', u'@lib/armeabi-v7a/liboskcontrib.so', u'@lib/armeabi-v7a/libtencentlocsapp.so', u'@lib/armeabi-v7a/libowl.so', u'@lib/armeabi-v7a/libImiCamera.so', u'@lib/armeabi-v7a/libusb-1.0.9.so', u'@lib/armeabi-v7a/libwxpaytls.so', u'@lib/armeabi-v7a/liblog.so', u'@lib/armeabi-v7a/libserial_port.so', u'@lib/armeabi-v7a/libuvc.so', u'@lib/armeabi-v7a/librsjni_androidx.so', u'@lib/armeabi-v7a/libilink_tdi.so', u'@lib/armeabi-v7a/libilink_im.so', u'@lib/armeabi-v7a/lib1180Driver.so', u'@lib/armeabi-v7a/libilink_dev.so', u'@lib/armeabi-v7a/libvpx.so', u'@lib/armeabi-v7a/libtencentloc.so', u'@lib/armeabi-v7a/libijkffmpeg.so', u'@lib/armeabi-v7a/libmarsxlog.so', u'@lib/armeabi-v7a/libijkplayer.so', u'@lib/armeabi-v7a/libxlog_native.so', u'@lib/armeabi-v7a/libmatrix-memoryhook.so', u'@lib/armeabi-v7a/libdeptrum_lite.jni.so', u'@lib/armeabi-v7a/libXnDeviceIminect.so', u'@lib/armeabi-v7a/libYTUtils.so', u'@lib/armeabi-v7a/libYTCommon.so', u'@lib/armeabi-v7a/libyuvutil.so']
/home/chenyubin/workspace_95/space/test/linux/packages/apps/TestApp/Android.mk has been successfully generated



SUCCESS
```

py script generates content:

Create TestApp directory under packages/apps/directory, copy TestApp.apk, create Android.mk file, resolve to lib/directory if apk exists so library

```bash
$ tree packages/apps/TestApp/ -L 1
packages/apps/TestApp/
├── Android.mk
├── lib
└── TestApp.apk

1 directory, 2 files
```

#### 2.1.4 Edit Android.mk file (ignore this section without special customization)

Section 2.1.3 runs the script to create the file Android.mk in the packages/apps/TestApp directory, which reads as follows:

```makefile
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)
# Module name should match apk name to be installed
LOCAL_MODULE := TestApp

#不管是user 还是eng 版本都会编译此应用
LOCAL_MODULE_TAGS := optional

LOCAL_SRC_FILES := TestApp.apk

#将应用打包进”/system/priv-app/”目录；如果为false，或者不加这句话，就会打包进”/system/app” 目录
LOCAL_PRIVILEGED_MODULE := true

LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
LOCAL_PREBUILT_JNI_LIBS := \
	@lib/armeabi-v7a/libxxx.so \
	@lib/armeabi/libxxx.so \
	@lib/arm64-v8a/libxxx.so \
	@lib/x86/libxxx.so  

#如果so文件是32位，而系统是64位的，那么需要加上这句
#LOCAL_MULTILIB := 32

# PRESIGND，表示用的是apk原本的签名
LOCAL_CERTIFICATE := PRESIGNED
include $(BUILD_PREBUILT)
```

In the generated Android.mk, if the lib file in TestApp.Apk does not have a so suffix file, the above code does not display LOCAL\_PREBUILT\_JNI\_LIBS.

If LOCAL\_PREBUILT\_JNI\_LIBS exists, make the following modifications:

Because TestApp/lib may have so files that support different cpu types, such as lib/armeabi-v7a/, lib/armeabi/, lib/arm64-v8a/, lib/x86/in the above example, you need to select different so files from the current system 32bit/64biit, and only add a so file of cpu type.

Case 1:

If the current system is 64bit, if lib/armeabi-v7a/, lib/armeabi/, lib/arm64-v8a/, lib/x86/exist at the same time, then select only lib/arm64-v8a/, and do not add extra so files. The method is as follows:

```makefile
替换
LOCAL_PREBUILT_JNI_LIBS := \
	@lib/armeabi-v7a/libxxx.so \
	@lib/armeabi/libxxx.so \
	@lib/arm64-v8a/libxxx.so \
	@lib/x86/libxxx.so  

为
###Clear the temporary variable JNI_LIBS
JNI_LIBS :=
###Recursive search of the current directory
$(foreach FILE,$(shell find $(LOCAL_PATH)/lib/arm64-v8a -name *.so), $(eval JNI_LIBS += $(FILE)))

Replacement reason:
During actual testing, some individual so libraries of certain apk files experienced loading failures. However, by using the method of first clearing the temporary variable JNI_LIBS and then traversing and loading the so libraries, it is ensured that all so libraries can be successfully loaded.
```

Case 2:

If the current system is 64bit, lib does not exist so library supporting 64bit, if lib/arm64-v8a/does not exist, then select lib library supporting 32bit;

If lib/armeabi-v7a/and lib/armeabi/exist in the 32-bit lib library, lib/armeabi-v7a/is preferred, and the replacement content is as follows:

```makefile
###Clear the temporary variable JNI_LIBS量JNI_LIBS
JNI_LIBS :=
###Recursive search of the current directory
$(foreach FILE,$(shell find $(LOCAL_PATH)/lib/armeabi-v7a -name *.so), $(eval JNI_LIBS += $(FILE)))
```
Case 3:

If the current system is 32-bit and lib/armeabi-v7a/, lib/armeabi/, lib/arm64-v8a/, and lib/x86/exist at the same time in lib, you can only select lib/armeabi-v7a/or lib/armeabi/,

lib/armeabi-v7a/is selected because lib/armeabi-v7a/has higher priority than lib/armeabi/ ;

If lib/armeabi-v7a/does not exist, you can only select lib/armeabi/, and replace it with the following:

```makefile
###Clear the temporary variable JNI_LIBS
JNI_LIBS :=
###Recursive search of the current directory
$(foreach FILE,$(shell find $(LOCAL_PATH)/lib/armeabi -name *.so), $(eval JNI_LIBS += $(FILE)))
```

To sum up, replace lib/arm64-v8a, lib/armeabi-v7a, lib/armeabi in the method as needed.

#### 2.1.5 Packaging TestApp Applications

Open the file device/rockchip/rk356 x/rk3566\_r/rk3566\_r.mk

Add TestApp to PRODUCT\_PACKAGES.

```makefile
PRODUCT_PACKAGES += TestApp
```

#### 2.1.6 Deleting Script Files

Delete the two script files and TestApp.apk that were originally placed in the root directory of RK 356X.

```
├── Commom_APK_NoCode_Linux.py
├── EditMe.xml
└── TestApp.apk
```

#### 2.1.7 Rebuild Project

Rebuild the entire project

### 2.2 Add APP permissions

Say it before:

TestApp usually adds users-permission to AndroidManifest.xml to indicate that it obtains a series of system authorizations; therefore, developers need to add corresponding android.permission.\*\*\* Authorization. There are two ways to add,

The first type: add application permission and add default permission authorization;

The second type\*\*\*\*(preferred)\*\*\*: Set the package name whitelist, that is, skip all system authorization restrictions according to the application package name; at the same time, set the ignore USB authorization pop-up box whitelist according to the package name.

Note 1: If apk is a system signature and preset in the system/priv-app directory, please refer to Section 2.2.1 of Mode;

Note 2: If apk is a custom signature and preset in the system/priv-app directory, please refer to Section 2.2.2 of Mode II.

Note 3: If apk is preset under system/app, it generally only contains ordinary permissions (no need to add permission whitelist), but it also has no privileged permissions.

For ease of maintenance, priority is given to using mode 2 (see Section 2.2.2). The advantage is that when apk adds some permissions, you don't have to add them one by one in the source code.

#### 2.2.1 Method 1: List and add permission whitelist

Note: If apk is a system signature and preset in the system/priv-app directory, use this method.

**How to obtain permissions manually:**

Drag TestApp.apk to AndroidStudio to open, get the package name, and list all uses-permissions in AndroidManifest.xml, such as android.permission.READ\_PHONE\_STATE, etc.



**How scripts get permissions:**

Copy generate\_permission.sh to packages/apps/TestApp/directory,

Execute this script to extract uses-permission from apk via aapt tool;

Orders such as:

```bash
./generate_permission.sh TestApp.apk
```

The effect is as follows:

    default-permissions-testapp.xml
    privapp-permissions-testapp.xml

[Note: ./ generate\_permission.sh needs to install aapt tool in linux environment first]

Format:

privapp-permissions-<lowercase appname.xml>, keep the file in packages/apps/TestApp/directory.

```xml
<?xml version="1.0" encoding="utf-8"?>
<permissions>
    <privapp-permissions package="com.v2ray.ang">
        <permission name="android.permission.REBOOT"/>
        ...
    </privapp-permissions>
</permissions>
```

default-permissions-<lowercase appname>.xml, keep the file in packages/apps/TestApp/directory.

```xml
<?xml version="1.0" encoding="utf-8"?>
<exceptions>
   <exception package="com.v2ray.ang">
        <permission name="android.permission.REBOOT"/>
        ...
   </exception>
</exceptions>
```

Open permissions.txt, copy the following and add it to
device/rockchip/rk356x/rk3566\_r/rk3566\_r.mk

```makefile
PRODUCT_PACKAGES += \
    default-permissions-<小写应用名称>.xml \
    privapp-permissions-<小写应用名称>.xml
```

Add the following to packages/apps/TestApp/Android.mk:

```makefile
 #==================================================
# Add default permission authorization
#==================================================
include $(CLEAR_VARS)
LOCAL_MODULE := default-permissions-<小写应用名称>.xml
LOCAL_MODULE_CLASS := ETC
LOCAL_MODULE_PATH := $(TARGET_OUT_ETC)/default-permissions
LOCAL_SRC_FILES := $(LOCAL_MODULE)
include $(BUILD_PREBUILT)
 
 
#==================================================
# Add application permission，Install priv-app permissions file
#==================================================
include $(CLEAR_VARS)
LOCAL_MODULE := privapp-permissions-<application name>.xml
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_CLASS := ETC
LOCAL_MODULE_PATH := ${TARGET_OUT_ETC}/permissions
LOCAL_SRC_FILES := ${LOCAL_MODULE}
include ${BUILD_PREBUILT}
```

Finally, delete redundant scripts and.txt temporary files.

#### 2.2.2 Method 2: Set white list according to package name

Note: If apk is a custom signature and preset in the system/priv-app directory, use this method.

Set package name array mPermissionWhitelist as whitelist;

Find the keyword grantPermission, determine whether it is a package name in the white list before authorization,

If yes, set grantPermission = true;

frameworks/base/services/core/java/com/android/server/pm/PackageManagerService.java

```diff
@@ -878,6 +877,19 @@ public class PackageManagerService extends IPackageManager.Stub
     final private ArrayList<IPackageChangeObserver> mPackageChangeObservers =
         new ArrayList<>();

+    /* Add by yanghuakun for skip permission list 2025.10.14  BEGIN */
+    final String[] mPermissionWhitelist = new String[]{
+        "com.test.testapp",
+        "com.test.testapp2"
+    };
+    /* Add by yanghuakun for skip permission list 2025.10.14  END */
+
     /**
      * Unit tests will instantiate, extend and/or mock to mock dependencies / behaviors.
      *
@@ -1765,8 +1777,10 @@ public class PackageManagerService extends IPackageManager.Stub
                         InstallArgs args = data.args;
                         PackageInstalledInfo parentRes = data.res;

-                        final boolean grantPermissions = (args.installFlags
+                        /* FIX by yanghuakun for skip permission list 2025.10.14  BEGIN */
+                        boolean grantPermissions = (args.installFlags
                                 & PackageManager.INSTALL_GRANT_RUNTIME_PERMISSIONS) != 0;
+                        /* FIX by yanghuakun for skip permission list 2025.10.14  END */
                         final boolean killApp = (args.installFlags
                                 & PackageManager.INSTALL_DONT_KILL_APP) == 0;
                         final boolean virtualPreload = ((args.installFlags
@@ -2147,6 +2161,14 @@ public class PackageManagerService extends IPackageManager.Stub
                         autoRevokePermissionsMode == MODE_IGNORED, UserHandle.myUserId());
             }

+            /* FIX by yanghuakun for skip permission list 2025.10.14  BEGIN */
+            for(int p=0; p < mPermissionWhitelist.length; p++) {
+                if (res.pkg.getPackageName().equals(mPermissionWhitelist[p])) {
+                    grantPermissions = true;
+                }
+            }
+            /* FIX by yanghuakun for skip permission list 2025.10.14  END */
+
             // Now that we successfully installed the package, grant runtime
             // permissions if requested before broadcasting the install. Also
             // for legacy apps in permission review mode we clear the permission
```

Find the grantSignaturePermission() method and modify it as follows:

frameworks/base/services/core/java/com/android/server/pm/permission/PermissionManagerService.java

```diff
diff --git a/services/core/java/com/android/server/pm/permission/PermissionManagerService.java b/services/core/java/com/android/server/pm/permission/PermissionManagerService.java
index 8d2363b6e83..84c7dc852e5 100644
--- a/services/core/java/com/android/server/pm/permission/PermissionManagerService.java
+++ b/services/core/java/com/android/server/pm/permission/PermissionManagerService.java
@@ -313,6 +313,20 @@ public class PermissionManagerService extends IPermissionManager.Stub {
     @GuardedBy("mLock")
     private DefaultHomeProvider mDefaultHomeProvider;

+    /*Add by yanghuakun for skip white list 2025.10.14 begin */
+    //½«apk°üÃû¼ÓÈëlistÖÐºó£¬²»ÐèÒªÔÚprivapp-permissions-platform.xml¼ÓÈ¨ÏÞ£¬»áÌø¹ý¡£
+    final String[] mSkipWhitelist = new String[]{
+        "com.test.testapp",
+        "com.test.testapp2",
+    };
+    /*Add by yanghuakun for skip white list 2025.10.14 end */
+
     // TODO: Take a look at the methods defined in the callback.
     // The callback was initially created to support the split between permission
     // manager and the package manager. However, it's started to be used for other

@@ -3593,15 +3650,26 @@ public class PermissionManagerService extends IPermissionManager.Stub {
                             Slog.w(TAG, "Privileged permission " + perm + " for package "
                                     + pkg.getPackageName() + " (" + pkg.getCodePath()
                                     + ") not in privapp-permissions whitelist");
-
-                            if (RoSystemProperties.CONTROL_PRIVAPP_PERMISSIONS_ENFORCE) {
-                                if (mPrivappPermissionsViolations == null) {
-                                    mPrivappPermissionsViolations = new ArraySet<>();
+                            /*Fix by yanghuakun for skip white list 2025.10.14 BEGIN */
+                            boolean mMatch = false;
+                            for(int p=0; p < mSkipWhitelist.length; p++) {
+                                if (pkg.getPackageName().equals(mSkipWhitelist[p])) {
+                                    Slog.w(TAG, "match! mSkipWhitelist[ " + p + "]: " + mSkipWhitelist[p]);
+                                    mMatch = true;
+                                    break;
+                                }
+                            }
+                            if (!mMatch) {
+                                if (RoSystemProperties.CONTROL_PRIVAPP_PERMISSIONS_ENFORCE) {
+                                    if (mPrivappPermissionsViolations == null) {
+                                        mPrivappPermissionsViolations = new ArraySet<>();
+                                    }
+                                    mPrivappPermissionsViolations.add(
+                                            pkg.getPackageName() + " (" + pkg.getCodePath() + "): "
+                                                    + perm);
                                 }
-                                mPrivappPermissionsViolations.add(
-                                        pkg.getPackageName() + " (" + pkg.getCodePath() + "): "
-                                         
```

If the whitelist contains the current package name, set mMatch=true;

After the modification is completed, delete out and compile again.

### 2.3 Preset application test verification

#### 2.3.1 Post-compilation validation

Observing compiled out:

Use find command to find out/target/product apk files, if not generated, please check whether there are omissions in the above configuration.

```bash
$ find ./out/target/product -iname "TestApp.apk"
$ find ./out/target/product -iname "<apk中so库>.so"
```

Method 1 requires querying the following documents, while method 2 does not.

```bash
$ find ./out/target/product -iname "privapp-permissions-<小写应用名称>.xml"
$ find ./out/target/product -iname "default-permissions-<小写应用名称>.xml"
```

#### 2.3.2 Verification after burning

After burning, check whether the application can be started.

For example, 1: Whether there is a desktop icon, click Open manually.

Example 2: Try to launch an application such as Gallery.

```bash
adb shell am start -n com.android.gallery3d/.app.GalleryActivity
```

For example, 3: Some applications are only used as components, no desktop icon, no activity, and need to be verified together with other integrations.

#### 2.3.3 Check if the app is installed

If the preset application is not started, check the installation package through adb mode:

After the device is connected to USB, you can quickly open the powershell window by pressing the shortcut key [win + X i].

For example, check the apk path according to the package name keyword:

```bash
adb shell
pm list packages -f | grep <target_package_name>
```

\--If the installed package can be found, it means that the application has been installed; if the application can be used normally, it means that it has been successfully preset.

\--If the installed package can be found, but the application cannot be used normally, there may be a system compatibility problem with apk, or apk itself reports an error.

\--If no installed package is found, but out/target/product/has found the appropriate apk, there may be a system compatibility issue that prevents the apk from starting successfully.

To sum up, if there is an apk that cannot be started or cannot be used normally, please grab the log check and provide it to the developer to solve.

#### 2.3.4 Catch log analysis exception

The log command is as follows:

```bash
$ adb logcat -b all -b time > log.txt
```



Appendix:

EditMe.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
/*
** Copyright 2009, The Android Open Source Project
**
** Licensed under the Apache License, Version 2.0 (the "License");
** you may not use this file except in compliance with the License.
** You may obtain a copy of the License at
**
**     http://www.apache.org/licenses/LICENSE-2.0
**
** Unless required by applicable law or agreed to in writing, software
** distributed under the License is distributed on an "AS IS" BASIS,
** WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
** See the License for the specific language governing permissions and
** limitations under the License.
*/
-->


<configuration>
    <!-- 所需预置APK名称.  以应用(TestApp.apk) 为例，在LOCAL\_MODULE属性中修改应用名称；如TestApp.Apk，填写为TestApp。-->
    <string name="LOCAL_MODULE">TestApp</string>

    <!-- 1.如需将应用打包进"/system/priv-app/"目录，则保持默认为true；
         2.如需打包进"/system/app"目录，则将true改为false. -->
    <bool name="LOCAL_PRIVILEGED_MODULE">true</bool>

    <!-- 1.默认为64
         2.如果so文件是32位，而系统是64位的，则请把64改为32.
         3. 或系统是32bit，so文件是32bit，则请把LOCAL\_MULTILIB属性值64修改为32 -->
    <integer name="LOCAL_MULTILIB">64</integer>
</configuration  >

```



Commom_APK_NoCode_Linux.py

```python
#!/usr/bin/python
#coding=utf-8
#===========================================================================
#  $DateTime: 2022/03/17 13:37:55 $ 
#  $Author: yanghuakun $

# when          who     what, where, why
# --------      ---     -------------------------------------------------------
# 2022-03-17    yhk     Add apk support.

# ===========================================================================

import struct, os, io
import re
import shutil
from types import *
import math
import struct
import zipfile
from zipfile import *
import sys
reload(sys)
sys.setdefaultencoding('utf-8')
import linecache
import xml.sax
from shutil import copyfile
from shutil import copy
from shutil import move
from xml.dom import minidom
import stat
import time
# 或者: from stat import S_IWUSR [as 别名]

if sys.version_info < (2,5):
    sys.stdout.write("\n\nERROR: This script needs Python version 2.5 or greater, detected as ")
    print sys.version_info
    sys.exit()  # error

def PrintBigErrorFromEditMeXML(sz):
    os.rename("EditMe.txt", "EditMe.xml")
    print "\t _________________ ___________ "
    print "\t|  ___| ___ \\ ___ \\  _  | ___ \\"
    print "\t| |__ | |_/ / |_/ / | | | |_/ /"
    print "\t|  __||    /|    /| | | |    / "
    print "\t| |___| |\\ \\| |\\ \\\\ \\_/ / |\\ \\ "
    print "\t\\____/\\_| \\_\\_| \\_|\\___/\\_| \\_|\n"

    if len(sz)>0:
        print sz
        sys.exit(1)

def PrintBigError(sz):
    print "\t _________________ ___________ "
    print "\t|  ___| ___ \\ ___ \\  _  | ___ \\"
    print "\t| |__ | |_/ / |_/ / | | | |_/ /"
    print "\t|  __||    /|    /| | | |    / "
    print "\t| |___| |\\ \\| |\\ \\\\ \\_/ / |\\ \\ "
    print "\t\\____/\\_| \\_\\_| \\_|\\___/\\_| \\_|\n"

    if len(sz)>0:
        print sz
        sys.exit(1)

def appoint_line(num,file):
   with io.open(file,"r",encoding='utf-8') as f:
       out = f.readlines(num-1)
       return out


so_list = []


# 搜索指定格式的文件
def findAllFilesWithSpecifiedSuffix(target_dir, target_suffix):
    find_res = []
    target_suffix_dot = "." + target_suffix
    walk_generator = os.walk(target_dir)
    for root_path, dirs, files in walk_generator:
        if len(files) < 1:
            continue
        for file in files:
            file_name, suffix_name = os.path.splitext(file)
            if suffix_name == target_suffix_dot:
                allPath = os.path.join(root_path, file)
                resPath = "@lib" + allPath.replace(target_dir,'')
                temp = resPath.replace("\\","/")
                find_res.append(temp)
    return find_res

#Zip文件处理类
class ZFile(object):
    def __init__(self, filename, mode='r', basedir=''):
        self.filename = filename
        self.mode = mode
        if self.mode in ('w', 'a'):
            self.zfile = zipfile.ZipFile(filename, self.mode, compression=zipfile.ZIP_DEFLATED)
        else:
            self.zfile = zipfile.ZipFile(filename, self.mode)
        self.basedir = basedir
        if not self.basedir:
            self.basedir = os.path.dirname(filename)

    def addfile(self, path, arcname=None):
        path = path.replace('//', '/')
        if not arcname:
            if path.startswith(self.basedir):
                arcname = path[len(self.basedir):]
            else:
                arcname = ''
        self.zfile.write(path, arcname)

    def addfiles(self, paths):
        for path in paths:
            if isinstance(path, tuple):
                self.addfile(*path)
            else:
                self.addfile(path)

    def close(self):
        self.zfile.close()

    def extract_to(self, path):
        for p in self.zfile.namelist():
            self.extract(p, path)

    def extract(self, filename, path):
        if not filename.endswith('/'):
            f = os.path.join(path, filename)
            dir = os.path.dirname(f)
            if not os.path.exists(dir):
                os.makedirs(dir)
            file(f, 'wb').write(self.zfile.read(filename))

#解压缩Zip到指定文件夹
def extractZip(zfile, path):
    print "Start extracting files: " + zfile
    z = ZFile(zfile)
    z.extract_to(path)
    print "Successfully decompressed to the path: " + path
    z.close()

#创建文件夹
def mkdir(path):
    # 去除首位空格
    path = path.strip()
    # 去除尾部 \ 符号
    path = path.rstrip("/")

    # 判断路径是否存在
    # 存在     True
    # 不存在   False
    isExists = os.path.exists(path)

    # 判断结果
    if not isExists:
        # 如果不存在则创建目录
        # 创建目录操作函数
        os.makedirs(path)

        print
        path + ' mkdir success: %s ' % path
        return True
    else:
        # 如果目录存在则不创建，并提示目录已存在
        print
        path + ' %s is exists' % path
        return False

#shutil.copyfile应用,参数必须具体到文件名
def copyfile(srcName,dstfile):
    if os.path.exists(srcName) is False:
        print("%s not exists!" % (srcName))
    else:
        filepath,filename=os.path.split(dstfile)
        if not os.path.exists(filepath):
            os.makedirs(filepath)
        try:
            shutil.copyfile(srcName,dstfile)
            #print("copy %s" % (srcName,dstfile))
        except IOError as e:
            print("Unable to copy file. %s" % e)
        except:
            print("Unexpected error:", sys.exc_info())

#shutil.copytree 参数必须具体到文件夹
def copytreefile(fromtreefile,totreefile):
    if os.path.exists(fromtreefile) is False:
        print("%s not exists!" % (fromtreefile))
    else:
        if os.path.exists(totreefile):
            shutil.rmtree(totreefile)
        time.sleep(1.0)
        try:
            shutil.copytree(fromtreefile,totreefile)
            print("copy %s to %s" %(fromtreefile,totreefile))
        except IOError as e:
            print("Unable to copy tree file. %s" % e)
        except:
            print("Unexpected error:", sys.exc_info())

#检验用户编辑过的EditMe.xml文件
def checkEditMeXML(editMePath):
    reslist = []
    Android_Mk_LOCAL_MODULE = linecache.getline(editMePath, 2).strip()
    Android_Mk_LOCAL_PRIVILEGED_MODULE = linecache.getline(editMePath, 5).strip()
    Android_Mk_LOCAL_MULTILIB = linecache.getline(editMePath, 8).strip()

    if Android_Mk_LOCAL_MODULE.find("LOCAL_MODULE :=")  != -1:
        AppName = Android_Mk_LOCAL_MODULE.replace("LOCAL_MODULE :=",'').strip()
        if AppName is None or AppName=='':
            PrintBigErrorFromEditMeXML("\n\nERROR: App name is empty, please fill in app name in EditMe.xml file  ")
        else:
            reslist.append(AppName)
    else:
        PrintBigErrorFromEditMeXML("\n\nERROR: LOCAL_MODULE, Please check the EditMe. XML  ")

    if Android_Mk_LOCAL_PRIVILEGED_MODULE.find("LOCAL_PRIVILEGED_MODULE :=")  != -1:
        value2 = Android_Mk_LOCAL_PRIVILEGED_MODULE.replace("LOCAL_PRIVILEGED_MODULE :=",'').replace("#",'').strip()
        if value2 is None or value2=='' or  (not (value2.upper() == "TRUE" or value2.upper() == "FALSE") ) :
            PrintBigErrorFromEditMeXML("\n\nERROR: LOCAL_PRIVILEGED_MODULE, Please check the EditMe. XML  ")
        else:
            reslist.append(Android_Mk_LOCAL_PRIVILEGED_MODULE)
    else:
        PrintBigErrorFromEditMeXML("\n\nERROR: LOCAL_PRIVILEGED_MODULE, Please check the EditMe. XML  ")

    if Android_Mk_LOCAL_MULTILIB.find("LOCAL_MULTILIB :=")  != -1:
        value3 = Android_Mk_LOCAL_MULTILIB.replace("LOCAL_MULTILIB :=",'').replace("#",'').strip()
        if value3 is None or value3=='' or (not (value3 == "32" or value3  == "64") ) :
            PrintBigErrorFromEditMeXML("\n\nERROR: LOCAL_MULTILIB, Please check the EditMe. XML  ")
        else:
            reslist.append(Android_Mk_LOCAL_MULTILIB)
    else:
        PrintBigErrorFromEditMeXML("\n\nERROR: LOCAL_MULTILIB, Please check the EditMe. XML  ")

    return  reslist

# 移除文件夹
def rmdir(path) :
    if os.path.exists(path):
        if os.path.isdir(path):
            # （先清空目录下面的文件），最后再删除根目录
            for root, dirs, files in os.walk(path, topdown=False):
                for name in files:
                    filename = os.path.join(root, name)
                    os.chmod(filename, stat.S_IWUSR)
                    os.remove(filename)
                for name in dirs:
                    os.rmdir(os.path.join(root, name))
            time.sleep(0.005)
            os.rmdir(path)

def replaceDoubleFanXieGanToSingleXieGan(path):
    path = path.replace('\\', '/')
    return path

def main():
    curdir = None
    editMe_Txt = "EditMe.txt"
    editMe_Xml = "EditMe.xml"
    editMePath = None
    AppName = None
    Android_Mk_LOCAL_PRIVILEGED_MODULE = """\n\n#将应用打包进”/system/priv-app/”目录；如果为false，或者不加这句话，就会打包进”/system/app” 目录\n"""
    Android_Mk_LOCAL_MULTILIB = """\n#如果so文件是32位，而系统是64位的，则请把行首'#'符号去掉；反之则保留\n"""
    flag = True

    curdir = os.path.abspath(os.curdir)

    # os.rename(editMe_Xml, editMe_Txt)
    # editMePath = os.path.join(curdir, editMe_Txt)
    # editmexmlList = checkEditMeXML(editMePath)
    # os.rename(editMe_Txt,editMe_Xml)
    # AppName = editmexmlList[0].strip()
    # Android_Mk_LOCAL_PRIVILEGED_MODULE = """\n\n#将应用打包进”/system/priv-app/”目录；如果为false，或者不加这句话，就会打包进”/system/app” 目录\n"""
    # Android_Mk_LOCAL_PRIVILEGED_MODULE += editmexmlList[1].strip()
    # Android_Mk_LOCAL_MULTILIB = """\n#如果so文件是32位，而系统是64位的，则请把行首'#'符号去掉；反之则保留\n"""
    # Android_Mk_LOCAL_MULTILIB += editmexmlList[2].strip()

    editMePath = os.path.join(curdir, editMe_Xml)
    editMePath = replaceDoubleFanXieGanToSingleXieGan(editMePath)
    dom = minidom.parse(editMePath)
    strs = dom.getElementsByTagName("string")
    tag2 = dom.getElementsByTagName("bool")
    tag3 = dom.getElementsByTagName("integer")

    for s in strs:
        if s.getAttribute('name') == 'LOCAL_MODULE':
            flag = False
            AppName = s.firstChild.data
            if AppName is None or AppName == '':
                PrintBigError("\n\nERROR: App name is empty, please fill in app name in EditMe.xml file  ")

    if flag:
        PrintBigError("\n\nERROR: LOCAL_MODULE, Please check the EditMe. XML  ")

    flag = True
    for s in tag2:
        if s.getAttribute('name') == 'LOCAL_PRIVILEGED_MODULE':
            flag = False
            value2 = s.firstChild.data
            if value2 is None or str(value2) == '' or (not (str(value2).upper() == "TRUE" or str(value2).upper() == "FALSE")):
                PrintBigError("\n\nERROR: LOCAL_PRIVILEGED_MODULE, Please check the EditMe. XML  ")
            else:
                Android_Mk_LOCAL_PRIVILEGED_MODULE += "LOCAL_PRIVILEGED_MODULE := " + str(value2)
        else:
            PrintBigError("\n\nERROR: LOCAL_PRIVILEGED_MODULE, Please check the EditMe. XML  ")

    if flag:
        PrintBigError("\n\nERROR: LOCAL_PRIVILEGED_MODULE, Please check the EditMe. XML  ")

    flag = True
    for s in tag3:
        if s.getAttribute('name') == 'LOCAL_MULTILIB':
            flag = False
            value3 = str(s.firstChild.data)
            if value3 is None or value3 == '' or (not (value3 == "32" or value3 == "64")):
                PrintBigError("\n\nERROR: LOCAL_MULTILIB, Please check the EditMe. XML  ")
            else:
                if (value3 == "64"):
                    Android_Mk_LOCAL_MULTILIB += "#LOCAL_MULTILIB := 32"
                else:
                    Android_Mk_LOCAL_MULTILIB += "LOCAL_MULTILIB := 32"

    if flag:
        PrintBigError("\n\nERROR: LOCAL_MULTILIB, Please check the EditMe. XML  ")

    print "Start adding applications: %s" % AppName

    AppName_Apk = AppName + ".apk"
    AppName_Zip = AppName + ".zip"
    AppName_Apk_Path = curdir + "/" + AppName_Apk
    AppName_Zip_Path = curdir + "/" + AppName_Zip
    AppName_mkdir = curdir + "/packages/apps/" + AppName

    print "Obtaining the Application Path: %s" % AppName_Apk_Path

    if os.path.exists(AppName_Apk_Path) is False:
        sys.stdout.write("\n\nERROR: Could not find: %s" % (AppName_Apk))
        sys.exit()

    AppNamePath = curdir + "/" + AppName
    AppLibPath_From = AppNamePath + "/lib"
    AppLibPath_To = AppName_mkdir + "/lib"
    os.rename(AppName + ".apk", AppName + ".zip")
    print(AppNamePath)
    if os.path.exists(AppNamePath) is False:
        mkdir(AppNamePath)
    extractZip(AppName_Zip_Path, AppNamePath)
    mkdir(AppName_mkdir)
    print "Creating an Application Path: %s" % AppName_mkdir
    copytreefile(AppLibPath_From,AppLibPath_To)
    copyfile(AppName_Zip_Path,AppName_mkdir + "/" + AppName_Apk)
    print "Copy %s to %s" % (AppName_Apk_Path,AppName_mkdir + "/" + AppName_Apk)


    Android_Mk_Start = """LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)
# Module name should match apk name to be installed"""

    Android_Mk_LOCAL_MODULE = "\nLOCAL_MODULE := " + AppName

    Android_Mk_LOCAL_MODULE_TAGS = """\n
#不管是user 还是eng 版本都会编译此应用
LOCAL_MODULE_TAGS := optional"""

    Android_Mk_LOCAL_SRC_FILES  = "\nLOCAL_SRC_FILES := " + AppName_Apk

    Android_Mk_LOCAL_MODULE_CLASS = """\n
LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)"""

    Android_Mk_LOCAL_PREBUILT_JNI_LIBS = """\n
LOCAL_PREBUILT_JNI_LIBS := \\
"""

    Android_Mk_End = """\n
LOCAL_DEX_PREOPT := false

# PRESIGND，表示用的是apk原本的签名
LOCAL_CERTIFICATE := PRESIGNED
include $(BUILD_PREBUILT)
"""


    if os.path.exists(AppNamePath + "/lib"):
        so_file_list = findAllFilesWithSpecifiedSuffix(AppNamePath + "/lib","so" )
        m = 0
        maxLen = len(so_file_list)
        if maxLen > 0:
            for li in so_file_list:
                m += 1
                if m < maxLen:
                    Android_Mk_LOCAL_PREBUILT_JNI_LIBS += "    " + li + " \\\n"
                else:
                    Android_Mk_LOCAL_PREBUILT_JNI_LIBS += "    " + li + "\n"

            print "Contain .so list"
            print so_file_list
        else:
            Android_Mk_LOCAL_PREBUILT_JNI_LIBS = ""
    else:
        Android_Mk_LOCAL_PREBUILT_JNI_LIBS = ""

    if Android_Mk_LOCAL_PREBUILT_JNI_LIBS is None or Android_Mk_LOCAL_PREBUILT_JNI_LIBS.strip() == "":
        print
        "Not contain .so list"

    try:
        Android_Mk = Android_Mk_Start + \
                     Android_Mk_LOCAL_MODULE + \
                     Android_Mk_LOCAL_MODULE_TAGS + \
                     Android_Mk_LOCAL_SRC_FILES + \
                     Android_Mk_LOCAL_PRIVILEGED_MODULE + \
                     Android_Mk_LOCAL_MODULE_CLASS + \
                     Android_Mk_LOCAL_PREBUILT_JNI_LIBS + \
                     Android_Mk_LOCAL_MULTILIB + \
                     Android_Mk_End
        f = open(AppName_mkdir + '/Android.txt', 'w')
        f.write(Android_Mk)
        f.close()
        print "%s has been successfully generated  " %(AppName_mkdir + "/Android.mk")
    except:
        os.rename(AppName_Zip, AppName_Apk)
        print("Unexpected error:", sys.exc_info())
        PrintBigError("\n\nERROR: Error generating Android.mk  ")


    os.rename(AppName_mkdir + "/Android.txt", AppName_mkdir + "/Android.mk")
    os.rename(AppName_Zip,AppName_Apk)
    try:
        print
        "Start delete %s" % AppNamePath
        rmdir(AppNamePath)
    except:
        PrintBigError("""\n\nERROR: Failed to delete the %s directory decompressed. 
        Please manually delete : %s """ % (AppName, AppNamePath))


    print "\n\nSUCCESS"
    return 0

if __name__ == '__main__':
    sys.exit(main())
```



generate_permission.sh

```bash
#!/bin/bash

# 启用调试日志并输出到终端与日志文件
set -x
exec > >(tee -a generate_permission.log) 2>&1

# 检查参数数量
if [ "$#" -ne 1 ]; then
    echo "Usage: $0 <apk_path>"
    exit 1
fi

APK_PATH="$1"

# 检查文件是否存在
if [ ! -f "$APK_PATH" ]; then
    echo "Error: File does not exist: $APK_PATH"
    exit 1
fi

# 获取 APK 名称（去掉 .apk 并转为小写）
APK_NAME=$(basename "$APK_PATH" .apk | tr '[:upper:]' '[:lower:]')
echo "APK Name: $APK_NAME"

# 使用 aapt 获取包名
PACKAGE_NAME=$(aapt dump badging "$APK_PATH" | grep package: | awk '{print $2}' | sed s/name=//g | sed s/\'//g)

if [ -z "$PACKAGE_NAME" ]; then
    echo "Error: Could not extract package name from APK."
    exit 1
fi
echo "Package Name: $PACKAGE_NAME"

# 提取权限列表
PERMISSIONS=$(aapt dump badging "$APK_PATH" | grep 'uses-permission:' | awk -F "'" '{print $2}')

if [ -z "$PERMISSIONS" ]; then
    echo "Warning: No permissions found in the APK file."
else
    echo "Permissions Found:"
    echo "$PERMISSIONS"
fi

# 设置 XML 文件名
DEFAULT_XML="default-permissions-$APK_NAME.xml"
PRIVAPP_XML="privapp-permissions-$APK_NAME.xml"
PERMISSIONS_TXT="permissions.txt"

# ========== 写入 privapp-permissions XML ==========
cat > "$PRIVAPP_XML" <<EOF
<?xml version="1.0" encoding="utf-8"?>
<permissions>
    <privapp-permissions package="$PACKAGE_NAME">
EOF

if [ -n "$PERMISSIONS" ]; then
    for permission in $PERMISSIONS; do
        echo "        <permission name=\"$permission\"/>" >> "$PRIVAPP_XML"
    done
else
    echo "        <!-- No permissions found -->" >> "$PRIVAPP_XML"
fi

cat >> "$PRIVAPP_XML" <<EOF
    </privapp-permissions>
</permissions>
EOF

echo "Generated $PRIVAPP_XML"

# ========== 写入 default-permissions XML ==========
cat > "$DEFAULT_XML" <<EOF
<?xml version="1.0" encoding="utf-8"?>
<exceptions>
    <exception package="$PACKAGE_NAME">
EOF

if [ -n "$PERMISSIONS" ]; then
    for permission in $PERMISSIONS; do
        echo "        <permission name=\"$permission\"/>" >> "$DEFAULT_XML"
    done
else
    echo "        <!-- No permissions found -->" >> "$DEFAULT_XML"
fi

cat >> "$DEFAULT_XML" <<EOF
    </exception>
</exceptions>
EOF

echo "Generated $DEFAULT_XML"

# 清空 permissions.txt 或创建新文件
> "$PERMISSIONS_TXT"

# ========== 写入 PRODUCT_PACKAGES 部分 ==========
cat >> "$PERMISSIONS_TXT" <<EOF
# Automatically generated by generate_permission.sh
PRODUCT_PACKAGES += \\
    $DEFAULT_XML \\
    $PRIVAPP_XML

EOF

# ========== 写入 default-permissions 构建规则 ==========
cat >> "$PERMISSIONS_TXT" <<EOF
#==================================================
# 添加默认权限授权
#==================================================
include \$(CLEAR_VARS)
LOCAL_MODULE := $DEFAULT_XML
LOCAL_MODULE_CLASS := ETC
LOCAL_MODULE_PATH := \$(TARGET_OUT_ETC)/default-permissions
LOCAL_SRC_FILES := \$(LOCAL_MODULE)
include \$(BUILD_PREBUILT)

EOF

# ========== 写入 privapp-permissions 构建规则 ==========
cat >> "$PERMISSIONS_TXT" <<EOF
#==================================================
# 添加应用权限，Install priv-app permissions file
#==================================================
include \$(CLEAR_VARS)
LOCAL_MODULE := $PRIVAPP_XML
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_CLASS := ETC
LOCAL_MODULE_PATH := \$(TARGET_OUT_ETC)/permissions
LOCAL_SRC_FILES := \$(LOCAL_MODULE)
include \$(BUILD_PREBUILT)

EOF

echo "Generated $PERMISSIONS_TXT with build rules and XML references for:"
echo "  - $DEFAULT_XML"
echo "  - $PRIVAPP_XML"
```



generate_somk.sh

```bash
#!/bin/bash

# 检查参数数量
if [ "$#" -ne 1 ]; then
    echo "Usage: $0 <lib_path>"
    echo "Example: $0 lib/armeabi-v7a"
    exit 1
fi

LIB_PATH="$1"

# 检查路径是否以 lib 开头
if [[ ! "$LIB_PATH" =~ ^lib ]]; then
    echo "Error: Path must start with 'lib'"
    exit 1
fi

# 检查路径是否存在
if [ ! -d "$LIB_PATH" ]; then
    echo "Error: Directory does not exist: $LIB_PATH"
    exit 1
fi

OUTPUT_FILE="somk.txt"

> "$OUTPUT_FILE"

# 收集所有 .so 文件名（basename）
mapfile -t SO_MODULES < <(find "$LIB_PATH" -type f -name "*.so" | xargs -n1 basename)

if [ ${#SO_MODULES[@]} -eq 0 ]; then
    echo "No .so files found under $LIB_PATH"
    exit 1
fi

# 生成 PRODUCT_PACKAGES += \ 块
{
    echo "PRODUCT_PACKAGES += \\"
} >> "$OUTPUT_FILE"

for module in "${SO_MODULES[@]}"; do
    echo "    $module \\" >> "$OUTPUT_FILE"
done

# 删除最后一行的反斜杠
sed -i '$ s/\\$//' "$OUTPUT_FILE"

# 添加空行分隔
echo "" >> "$OUTPUT_FILE"

# 生成每个 .so 的构建规则
for module in "${SO_MODULES[@]}"; do
    # 确保路径结尾没有 /，防止 // 出现
    clean_path="${LIB_PATH%/}"
    so_file="$clean_path/$module"

    cat >> "$OUTPUT_FILE" <<EOF
include \$(CLEAR_VARS)
LOCAL_MODULE := $module
LOCAL_MODULE_CLASS := SHARED_LIBRARIES
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_PATH := \$(TARGET_OUT_SHARED_LIBRARIES)
LOCAL_SRC_FILES := $so_file
LOCAL_CHECK_ELF_FILES := false
include \$(BUILD_PREBUILT)

EOF
done

echo "Generated $OUTPUT_FILE with prebuilt rules for ${#SO_MODULES[@]} .so files under $LIB_PATH"
```