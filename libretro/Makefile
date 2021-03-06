DEBUG = 0
DYNAREC = 0
HAVE_THREADS = 1
HAVE_GRIFFIN = 0

ifeq ($(platform),)
	platform = unix
	ifeq ($(shell uname -a),)
		platform = win
	else ifneq ($(findstring Darwin,$(shell uname -a)),)
		platform = osx
	else ifneq ($(findstring MINGW,$(shell uname -a)),)
		platform = win
	endif
endif

# system platform
system_platform = unix
ifeq ($(shell uname -a),)
	EXE_EXT = .exe
	system_platform = win
else ifneq ($(findstring Darwin,$(shell uname -a)),)
	system_platform = osx
	arch = intel
	ifeq ($(shell uname -p),powerpc)
		arch = ppc
	endif
else ifneq ($(findstring MINGW,$(shell uname -a)),)
	system_platform = win
endif

CORE_DIR := .

TARGET_NAME = yabause
CC_AS ?= $(CC)
GIT_VERSION := " $(shell git rev-parse --short HEAD || echo unknown)"
ifneq ($(GIT_VERSION)," unknown")
	CFLAGS += -DGIT_VERSION=\"$(GIT_VERSION)\"
endif

# Unix
ifneq (,$(findstring unix,$(platform)))
	TARGET := $(TARGET_NAME)_libretro.so
	fpic := -fPIC
	SHARED := -shared -Wl,--no-undefined -Wl,--version-script=link.T
	#DYNAREC = 2
	LDFLAGS += -lpthread
	HAVE_PTHREADS = 1

else ifneq (,$(findstring linux-portable,$(platform)))
	TARGET := $(TARGET_NAME)_libretro.so
	fpic := -fPIC -nostdlib
	SHARED := -shared -Wl,--version-script=link.T
	#DYNAREC = 2
	HAVE_PTHREADS = 1

# OS X
else ifneq (,$(findstring osx,$(platform)))
	TARGET := $(TARGET_NAME)_libretro.dylib
	fpic := -fPIC
	fpic += -mmacosx-version-min=10.1
	SHARED := -dynamiclib
ifeq ($(arch),ppc)
	ENDIANNESS_DEFINES := -DWORDS_BIGENDIAN -DMSB_FIRST -D__ppc__
endif
	#DYNAREC = 2
	LDFLAGS += -lpthread
	HAVE_PTHREADS = 1

else ifneq (,$(findstring theos_ios,$(platform)))
	# Theos iOS
	DEPLOYMENT_IOSVERSION = 5.0
	TARGET = iphone:latest:$(DEPLOYMENT_IOSVERSION)
	ARCHS = armv7 armv7s
	TARGET_IPHONEOS_DEPLOYMENT_VERSION=$(DEPLOYMENT_IOSVERSION)
	THEOS_BUILD_DIR := objs
	include $(THEOS)/makefiles/common.mk

LIBRARY_NAME = $(TARGET_NAME)_libretro_ios

else ifneq (,$(findstring ios,$(platform)))
	# iOS
	TARGET := $(TARGET_NAME)_libretro_ios.dylib
	fpic := -fPIC
	SHARED := -dynamiclib
	CC = clang -arch armv7 -isysroot $(IOSSDK)
	CC_AS =  perl ./libretro/gas-preprocessor.pl $(CC)
	CXX = clang++ -arch armv7 -isysroot $(IOSSDK)

# QNX
else ifneq (,$(findstring qnx,$(platform)))
	TARGET := $(TARGET_NAME)_libretro_$(platform).so
	fpic := -fPIC
	SHARED := -shared -Wl,--version-script=link.T
	CC = qcc -Vgcc_ntoarmv7le
	CC_AS = $(CC)
	CXX = QCC -Vgcc_ntoarmv7le
	AR = qcc -Vgcc_ntoarmv7le
	ENDIANNESS_DEFINES += -D__BLACKBERRY_QNX__ -marm -mcpu=cortex-a9 -mfpu=neon -mfloat-abi=softfp -fms-extensions

# PS3
else ifneq (,$(findstring ps3,$(platform)))
	TARGET := $(TARGET_NAME)_libretro_$(platform).a
	CC = $(CELL_SDK)/host-win32/ppu/bin/ppu-lv2-gcc.exe
	CXX = $(CELL_SDK)/host-win32/ppu/bin/ppu-lv2-g++.exe
	AR = $(CELL_SDK)/host-win32/ppu/bin/ppu-lv2-ar.exe
	ENDIANNESS_DEFINES = -DWORDS_BIGENDIAN -DMSB_FIRST -D__ppc__
	STATIC_LINKING = 1

# sncps3
else ifneq (,$(findstring sncps3,$(platform)))
	TARGET := $(TARGET_NAME)_libretro_ps3.a
	CC = $(CELL_SDK)/host-win32/sn/bin/ps3ppusnc.exe
	CXX = $(CELL_SDK)/host-win32/sn/bin/ps3ppusnc.exe
	AR = $(CELL_SDK)/host-win32/sn/bin/ps3snarl.exe
	ENDIANNESS_DEFINES = -DWORDS_BIGENDIAN -DMSB_FIRST -D__ppc__
	STATIC_LINKING = 1

# Lightweight PS3 Homebrew SDK
else ifneq (,$(findstring psl1ght,$(platform)))
	TARGET := $(TARGET_NAME)_libretro_$(platform).a
	CC = $(PS3DEV)/ppu/bin/ppu-gcc$(EXE_EXT)
	CC = $(PS3DEV)/ppu/bin/ppu-g++$(EXE_EXT)
	AR = $(PS3DEV)/ppu/bin/ppu-ar$(EXE_EXT)
	ENDIANNESS_DEFINES = -DWORDS_BIGENDIAN -DMSB_FIRST -D__ppc__
	STATIC_LINKING = 1

# PSP
else ifneq (,$(findstring psp1,$(platform)))
	TARGET := $(TARGET_NAME)_libretro_$(platform).a
	CC = psp-gcc$(EXE_EXT)
	CC_AS = $(CC)
	CXX = psp-g++$(EXE_EXT)
	AR = psp-ar$(EXE_EXT)
	ENDIANNESS_DEFINES += -DPSP -G0 
	STATIC_LINKING = 1

# Vita
else ifneq (,$(findstring vita,$(platform)))
	TARGET := $(TARGET_NAME)_libretro_$(platform).a
	CC = arm-vita-eabi-gcc$(EXE_EXT)
	CCX = arm-vita-eabi-g++$(EXE_EXT)
	CC_AS = arm-vita-eabi-gcc$(EXE_EXT)
	AR = arm-vita-eabi-ar$(EXE_EXT)
	ENDIANNESS_DEFINES += -DVITA
	#DYNAREC = 2
	STATIC_LINKING = 1

# Nintendo Game Cube
else ifneq (,$(findstring ngc,$(platform)))
	TARGET := $(TARGET_NAME)_libretro_$(platform).a
	CC = $(DEVKITPPC)/bin/powerpc-eabi-gcc$(EXE_EXT)
	CXX = $(DEVKITPPC)/bin/powerpc-eabi-g++$(EXE_EXT)
	AR = $(DEVKITPPC)/bin/powerpc-eabi-ar$(EXE_EXT)
	ENDIANNESS_DEFINES = -DWORDS_BIGENDIAN -DMSB_FIRST -DGEKKO -DHW_DOL -mrvl -mcpu=750 -meabi -mhard-float -D__ppc__
	STATIC_LINKING = 1

# Nintendo Wii
else ifneq (,$(findstring wii,$(platform)))
	TARGET := $(TARGET_NAME)_libretro_$(platform).a
	CC = $(DEVKITPPC)/bin/powerpc-eabi-gcc$(EXE_EXT)
	CXX = $(DEVKITPPC)/bin/powerpc-eabi-g++$(EXE_EXT)
	AR = $(DEVKITPPC)/bin/powerpc-eabi-ar$(EXE_EXT)
	ENDIANNESS_DEFINES = -DWORDS_BIGENDIAN -DMSB_FIRST -DGEKKO -DHW_RVL -mrvl -mcpu=750 -meabi -mhard-float -D__ppc__
	STATIC_LINKING = 1

# Nintendo Switch (libtransistor)
else ifeq ($(platform), switch-transistor)
	TARGET := $(TARGET_NAME)_libretro_$(platform).a
	include $(LIBTRANSISTOR_HOME)/libtransistor.mk
	STATIC_LINKING=1

# Nintendo Switch (libnx)
else ifeq ($(platform), switch)
include $(DEVKITPRO)/libnx/switch_rules
    EXT=a
    TARGET := $(TARGET_NAME)_libretro_$(platform).$(EXT)
    DEFINES := -DSWITCH=1 -U__linux__ -U__linux -DRARCH_INTERNAL
    CFLAGS	:=	 $(DEFINES) -g \
                -O2 \
				-fPIE -I$(LIBNX)/include/ -ffunction-sections -fdata-sections -ftls-model=local-exec -Wl,--allow-multiple-definition -specs=$(LIBNX)/switch.specs
    CFLAGS += $(INCDIRS)
    CFLAGS	+=	$(INCLUDE)  -D__SWITCH__
    CXXFLAGS := $(ASFLAGS) $(CFLAGS) -fno-rtti -fno-exceptions -std=gnu++11
    CFLAGS += -std=gnu11
HAVE_THREADS = 0
STATIC_LINKING = 1

# ARM
else ifneq (,$(findstring armv,$(platform)))
	TARGET := $(TARGET_NAME)_libretro.so
	fpic := -fPIC
	SHARED := -shared -Wl,--no-undefined -Wl,--version-script=link.T
	LDFLAGS += -lpthread
	HAVE_PTHREADS = 1
	ifneq (,$(findstring cortexa8,$(platform)))
		FLAGS += -marm -mcpu=cortex-a8
	else ifneq (,$(findstring cortexa9,$(platform)))
		FLAGS += -marm -mcpu=cortex-a9
	endif
	FLAGS += -marm
	ifneq (,$(findstring neon,$(platform)))
		FLAGS += -mfpu=neon
		HAVE_NEON = 1
	endif
	ifneq (,$(findstring softfloat,$(platform)))
		FLAGS += -mfloat-abi=softfp
	else ifneq (,$(findstring hardfloat,$(platform)))
		FLAGS += -mfloat-abi=hard
	endif
	FLAGS += -DARM

# Emscripten
else ifeq ($(platform), emscripten)
	TARGET := $(TARGET_NAME)_libretro_$(platform).bc
	STATIC_LINKING = 1

# GCW0
else ifeq ($(platform), gcw0)
	TARGET := $(TARGET_NAME)_libretro.so
	CC = /opt/gcw0-toolchain/usr/bin/mipsel-linux-gcc
	CXX = /opt/gcw0-toolchain/usr/bin/mipsel-linux-g++
	AR = /opt/gcw0-toolchain/usr/bin/mipsel-linux-ar
	SHARED := -shared -Wl,--no-undefined -Wl,--version-script=link.T
	#HAVE_PTHREADS = 1
	LDFLAGS += $(PTHREAD_FLAGS) -lrt
	FLAGS += $(PTHREAD_FLAGS) -DHAVE_MKDIR
	FLAGS += -ffast-math -march=mips32 -mtune=mips32r2 -mhard-float
	fpic := -fPIC
	CFLAGS += -DFAMEC_NO_GOTOS

# Windows
else
	TARGET := $(TARGET_NAME)_libretro.dll
	CC = gcc
	CXX = g++
	SHARED := -shared -Wl,--no-undefined -Wl,--version-script=link.T
	LDFLAGS += -static-libgcc -static-libstdc++ -lwinmm
	FLAGS += -DHAVE_WFOPEN
	#DYNAREC = 2

endif

include Makefile.common

OBJECTS += $(SOURCES_C:.c=.o) $(SOURCES_CXX:.cpp=.o) $(C68KEXEC_OBJECT)

ifeq ($(DEBUG), 1)
FLAGS += -O0 -g
else
FLAGS += -O2 -DNDEBUG
endif

LDFLAGS += $(fpic) $(SHARED)
FLAGS += $(fpic)

WARNINGS :=
FLAGS += $(INCFLAGS) -D__LIBRETRO__ $(ENDIANNESS_DEFINES) -DSIZEOF_DOUBLE=8 $(WARNINGS) -DUSE_16BPP=1 -DUSE_RGB_565=1 \
			-DUSE_SCSP2=1 -DNO_CLI -DHAVE_SYS_TIME_H -DHAVE_GETTIMEOFDAY -DVERSION=\"0.9.12\" -DHAVE_STRCASECMP -DINLINE="inline"

CXXFLAGS += $(FLAGS)

CFLAGS += $(FLAGS)
ifeq ($(platform),qnx)
CFLAGS += -Wc,-std=c99
else
CFLAGS += -std=gnu99
endif

ifeq ($(platform), theos_ios)
COMMON_FLAGS := -DIOS $(COMMON_DEFINES) $(INCFLAGS) -I$(THEOS_INCLUDE_PATH) -Wno-error
$(LIBRARY_NAME)_CFLAGS += $(CFLAGS) $(COMMON_FLAGS)
$(LIBRARY_NAME)_CXXFLAGS += $(CXXFLAGS) $(COMMON_FLAGS)
${LIBRARY_NAME}_FILES = $(SOURCES_CXX) $(SOURCES_C) $(SOURCE_DIR)/c68k/c68kexec.c
include $(THEOS_MAKE_PATH)/library.mk
else
all: $(TARGET)
$(TARGET): $(OBJECTS)
ifeq ($(STATIC_LINKING), 1)
	$(AR) rcs $@ $(OBJECTS)
else
	$(CXX) -o $@ $^ $(LDFLAGS)
endif

%.o: %.S
	$(CC_AS) $(CFLAGS) -c $^ -o $@

%.o: %.s
	$(CC_AS) $(CFLAGS) -c $^ -o $@

%.o: %.cpp
	$(CXX) -c -o $@ $< $(CXXFLAGS)

%.o: %.c
	$(CC) -c -o $@ $< $(CFLAGS)
	
$(C68KEXEC_OBJECT): $(C68KEXEC_SOURCE)
	$(CC) -c -o $@ $< $(CFLAGS) -O0

clean:
	rm -f $(TARGET) $(OBJECTS)

.PHONY: clean
endif
