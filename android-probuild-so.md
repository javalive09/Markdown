# **Android.mk预装so库说明**

# Android.mk变量说明
## CLEAR_VARS

此变量指向的构建脚本用于取消定义下文“开发者定义的变量”部分中列出的几乎所有 LOCAL_XXX 变量。在描述新模块之前，请使用此变量来包含此脚本。使用它的语法为：
```
include $(CLEAR_VARS)
```
## TARGET_ARCH

构建系统解析此 Android.mk 文件时指向的 CPU 系列。此变量将是下列其中一项：arm、arm64、x86 或 x86_64。
## LOCAL_PATH

此变量用于指定当前文件的路径。必须在 Android.mk 文件开头定义此变量。以下示例演示了如何定义此变量：
```
LOCAL_PATH := $(call my-dir)
```
CLEAR_VARS 所指向的脚本不会清除此变量。因此，即使 Android.mk 文件描述了多个模块，您也只需定义此变量一次。
## LOCAL_MODULE

此变量用于存储模块名称。指定的名称在所有模块名称中必须唯一，并且不得包含任何空格。您必须先定义该名称，然后才能添加任何脚本（CLEAR_VARS 的脚本除外）。无需添加 lib 前缀或 .so 或 .a 文件扩展名；构建系统会自动执行这些修改。在整个 Android.mk 和 Application.mk 文件中，请用未经修改的名称引用模块。例如，以下行会导致生成名为 libfoo.so 的共享库模块：
```
LOCAL_MODULE := "foo"
```
如果您希望生成的模块使用除“lib + LOCAL_MODULE 的值”以外的名称，可以使用 LOCAL_MODULE_FILENAME 变量为生成的模块指定自己选择的名称。 LOCAL_MODULE将在每个模块的makefile里定义，如果未定义，编译系统会报错

除应用(apk)以LOCAL_PACKAGE_NAME指定模块名以外，其余的模块都以LOCAL_MODULE指定模块名。

## LOCAL_MODULE_FILENAME

此可选变量使您能够替换构建系统为其生成的文件默认使用的名称。例如，如果 LOCAL_MODULE 的名称为 foo，您可以强制系统将其生成的文件命名为 libnewfoo。以下示例演示了如何完成此操作：
```
LOCAL_MODULE := foo
LOCAL_MODULE_FILENAME := libnewfoo
```
对于共享库模块，此示例将生成一个名为 libnewfoo.so 的文件。

## LOCAL_SRC_FILES

此变量包含构建系统生成模块时所用的源文件列表。只列出构建系统实际传递到编译器的文件，因为构建系统会自动计算所有相关的依赖项。请注意，您可以使用相对（相对于 LOCAL_PATH）和绝对文件路径。

建议您避免使用绝对文件路径；相对路径可以提高 Android.mk 文件的移植性。

## TARGET_PLATFORM

构建系统解析此 Android.mk 文件时指向的 Android API 级别号。例如，Android 5.1 系统映像对应于 Android API 级别 22：android-22。如需查看平台名称和相应 Android 系统映像的完整列表，请参阅 Android NDK 原生 API。
## LOCAL_MODULE_PATH

表示模块编译结果将要存放的目录
## LOCAL_MODULE_STEM

表示编译链接后的目标文件的文件名，不带后缀
## LOCAL_BUILT_MODULE_STEM

表示编译链接后的目标文件的文件名，带后缀

```
LOCAL_BUILT_MODULE_STEM := $(strip $(LOCAL_BUILT_MODULE_STEM))
ifeq ($(LOCAL_BUILT_MODULE_STEM),)
  LOCAL_BUILT_MODULE_STEM := $(LOCAL_INSTALLED_MODULE_STEM)
endif
例：
recovery模块编译recovery可执行文件：
LOCAL_INSTALLED_MODULE_STEM:=recovery
```

## LOCAL_MODULE_CLASS

将用于决定编译时的中间文件存放的位置，LOCAL_MODULE_CLASS在定义目标生成方式的makefile文件里定义，比如executable.mk里定义LOCAL_MODULE_CLASS := EXECUTABLES。 其它的LOCAL_MODULE_CLASS有 LOCAL_MODULE_CLASS := ETC LOCAL_MODULE_CLASS := STATIC_LIBRARIES LOCAL_MODULE_CLASS := EXECUTABLES LOCAL_MODULE_CLASS := FAKE LOCAL_MODULE_CLASS := JAVA_LIBRARIES LOCAL_MODULE_CLASS := SHARED_LIBRARIES LOCAL_MODULE_CLASS := APPS
## LOCAL_MODULE_SUFFIX

表示编译链接后的目标文件的后缀，在预装app的时候为.app，在so库的时候为.so
## BUILD_PREBUILT

种方式把文件当成编译项目，在Android.mk中copy一个file

## 预装so库
```
ifeq ($(strip $(TARGET_ARCH)),arm64)
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := libwt_jni
LOCAL_SRC_FILES := $(LOCAL_MODULE).so
LOCAL_MODULE_STEM := $(LOCAL_MODULE)
LOCAL_MODULE_SUFFIX := $(suffix $(LOCAL_SRC_FILES))
LOCAL_MODULE_CLASS := SHARED_LIBRARIES
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_PATH := $(TARGET_OUT_SHARED_LIBRARIES)
include $(BUILD_PREBUILT)
endif
```

# [Android.mk谷歌链接](https://developer.android.com/ndk/guides/android_mk?hl=zh-cn#target_platform)
