mkfile_path := $(abspath $(lastword $(MAKEFILE_LIST)))  
PATH_CURR := $(strip $(patsubst %/, %, $(dir ${mkfile_path})))
PATH_MICROPY := $(strip $(patsubst %/ports/, %, $(dir ${PATH_CURR})))
CSDK_PATH := ${PATH_MICROPY}/lib/GPRS_C_SDK
include ../../py/mkenv.mk

CSDTK_PATH = ${PATH_MICROPY}/lib/csdtk42-linux
# mips-elf-* compiler libraries
export LD_LIBRARY_PATH=$(CSDTK_PATH)/lib:$(CSDTK_PATH)/mingw32/usr/lib

include ${CSDK_PATH}/platform/csdk/memd.def
include ${CSDK_PATH}/platform/chip/defs/8955_base.def
include ${CSDK_PATH}/init/target.def

RELEASE = 0

ifeq ($(strip $(RELEASE)),1)
	CFG_RELEASE = release
else
	CFG_RELEASE = debug
endif

$(info Compile Version:$(CFG_RELEASE))

# qstr definitions (must come before including py.mk)
QSTR_DEFS = qstrdefsport.h

#############################################

# MicroPython feature configurations
MICROPY_PY_USSL = 1
MICROPY_SSL_AXTLS = 1
AXTLS_DEFS_EXTRA = -Dabort=abort_ -include time.h -DRT_MAX_PLAIN_LENGTH=1024 -DRT_EXTRA=4096

FROZEN_MANIFEST ?= boards/manifest.py

# include py core make definitions
include $(TOP)/py/py.mk

#############################################

CROSS_COMPILE = $(CSDTK_PATH)/bin/mips-elf-
MAKE = make
LD  := $(CROSS_COMPILE)ld
LDPP := $(CROSS_COMPILE)cpp -C -P -undef
STDOUT_NULL := 1>/dev/null

BUILD_PATH := build
BUILD_PATH_ABS := ${PATH_CURR}/${BUILD_PATH}
FIRMWARE_NAME := firmware_${CFG_RELEASE}
BIN := $(BUILD_PATH_ABS)/${FIRMWARE_NAME}.elf
HEX := $(BUILD_PATH_ABS)/${FIRMWARE_NAME}.srec
BIN_PATH := ${PATH_CURR}/hex
LODBASE   := $(BIN_PATH)/${FIRMWARE_NAME}_
LOD_FILE := ${LODBASE}flash.lod
WITH_PLT_LOD_FILE := ${BIN_PATH}/${FIRMWARE_NAME}_full.lod
WITH_PLT_OTA_FILE := ${BIN_PATH}/${FIRMWARE_NAME}_ota.lod
STRIP_SYMBOL_FILE := ${CSDK_PATH}/platform/compilation/platform_symbols_to_strip
PLATFORM_ELF_FOLDER_PATH := ${CSDK_PATH}/platform/csdk/debug
PLATFORM_ELF_PATH := ${wildcard ${PLATFORM_ELF_FOLDER_PATH}/*.elf}
PLATFORM_ELF_NAME := ${notdir ${PLATFORM_ELF_PATH}}
PLATFORM_LOD_PATH := ${CSDK_PATH}/platform/csdk/${strip ${CFG_RELEASE}}
PLATFORM_LOD_FILE := ${wildcard ${PLATFORM_LOD_PATH}/*.lod}
ELFCOMBINE_TOOL := PATH=$$PATH:$(CSDTK_PATH)/bin ${CSDK_PATH}/platform/compilation/elfCombine.pl
SRECMAP_TOOL := $(CSDTK_PATH)/bin/srecmap 
TARGET_FILE := ${BUILD_PATH_ABS}/targetgen
MAP_CFG_FILE := ${CSDK_PATH}/platform/compilation/8955_map_cfg
LODCOMBINE_TOOL := ${CSDK_PATH}/platform/compilation/lodtool.py


LIBGCC_PATH := ${CSDK_PATH}/platform/lib/gcc/mips-elf/4.4.2
MAP := ${BUILD_PATH_ABS}/${FIRMWARE_NAME}.map
FULL_LIBRARY_PATH := -L${CSDK_PATH}/hex/libcsdk
MYLIBFILES := 
FULL_LIBRARY_EXT := -lcsdk_debug 


LDPPFLAGS += \
        -DUSER_ROM_BASE=${FLASH_BASE}+${USER_ROM_BASE} \
        -DUSER_ROM_SIZE=${USER_ROM_SIZE} \
        -DUSER_RAM_BASE=${EXT_RAM_BASE}+${USER_RAM_BASE} \
        -DUSER_RAM_SIZE=${USER_RAM_SIZE} \
        -DDUSE_BINUTILS_2_19=1
LD_SRC := ${CSDK_PATH}/platform/compilation/cust.ld
LD_FILE := ${BUILD_PATH_ABS}/cust.ld
OFORMAT := --oformat=elf32-littlemips
SREC	:= --input-target=elf32-littlemips --output-target=srec
LDFLAG_USED_ELF_FILES := -just-symbols ${CSDK_PATH}/platform/chip/rom/8955/lib/mem_bridge_rom_CHIP.elf
LDFLAGS += --gc-sections
LDFLAGS += ${FULL_LIBRARY_PATH} --start-group ${MYLIBFILES} ${FULL_LIBRARY_EXT} --end-group
LDFLAGS	+= -L${LIBGCC_PATH} -lgcc 
LDFLAGS += --warn-common  --error-unresolved-symbols 
FULL_SRC_OBJECTS := 
LD_OPTIONAL_OBJECTS := 

INC += -I.
INC += -I$(TOP)
INC += -I$(BUILD_PATH_ABS)
INC += -I${CSDK_PATH}/include \
       -I${CSDK_PATH}/include/std_inc \
	   -I${CSDK_PATH}/include/api_inc \
	   -I${CSDK_PATH}/libs/utils/include \
	   -I${CSDK_PATH}/libs/gps/include \
	   -I${CSDK_PATH}/libs/gps/minmea/src 

CFLAGS = $(INC) -Wshadow \
		  -std=gnu99 \
          -Os -g -fno-inline-small-functions -fno-inline-functions -fno-align-functions -fno-align-jumps -fno-align-loops -fno-align-labels \
          -msoft-float \
          -Wunused-function \
          -Wuninitialized \
          -minterlink-mips16 -fno-strict-aliasing \
          -ffunction-sections -fdata-sections \
          -fno-builtin-iswspace \
          -ffixed-t3 -ffixed-t4 -ffixed-t5 -ffixed-t6 -ffixed-t7 -ffixed-s2 -ffixed-s3 -ffixed-s4 -ffixed-s5 -ffixed-s6 -ffixed-s7 -ffixed-fp \
          -G0 -Wall -march=xcpu -mtune=xcpu -Wa,-march=xcpu,-mtune=xcpu -EL -mexplicit-relocs -fweb -frename-registers -mmemcpy -mmips-tfile -nostartfiles -nostdlib -nostdinc -nodefaultlibs -c -pipe \
          -fwide-exec-charset=UTF-16LE -fshort-wchar \
		  -mips16 
CFLAGS += $(CFLAGS_MOD)

ASFLAGS = --warn -EL -march=xcpu -mtune=xcpu

SRC_C = \
	main.c \
	posix_helpers.c \
	fatal.c \
	uart.c \
	help.c \
	modmachine.c \
	machine_pin.c \
	mphalport.c \
	moduos.c \
	modutime.c \
	rng.c \
	fatfs_port.c \
	modchip.c \
	file_io.c \
	modcellular.c \
	modgps.c \
	modusocket.c \
	modi2c.c \
	machine_adc.c \
	machine_uart.c \
	machine_rtc.c


LIB_SRC_C = $(addprefix lib/,\
	mp-readline/readline.c \
	netutils/netutils.c \
	timeutils/timeutils.c \
	utils/pyexec.c \
	utils/interrupt_char.c \
	utils/sys_stdio_mphal.c \
	)

LIB_SRC_C += \
	lib/oofatfs/ff.c \
	lib/oofatfs/ffunicode.c

DRIVERS_SRC_C = $(addprefix drivers/,\
	bus/softspi.c \
        )

SRC_S = \
	gchelper.s

OBJ_MP =
OBJ_MP += $(PY_O)
OBJ_MP += $(addprefix $(BUILD)/, $(SRC_C:.c=.o))

OBJ = $(OBJ_MP) $(PY_CORE_O)
OBJ += $(addprefix $(BUILD_PATH)/, $(LIB_SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD_PATH)/, $(SRC_S:.s=.o))
OBJ += $(addprefix $(BUILD_PATH)/, $(SRC_MOD:.c=.o))
OBJ += $(addprefix $(BUILD_PATH)/, $(EXTMOD_SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD_PATH)/, $(DRIVERS_SRC_C:.c=.o))

# List of sources for qstr extraction
SRC_QSTR += $(SRC_C)  $(LIB_SRC_C) 
# Append any auto-generated sources that are needed by sources listed in SRC_QSTR
SRC_QSTR_AUTO_DEPS +=

all: lod

lod: ${HEX}
	$(Q)$(MAKE) targetgen
	$(Q)$(SRECMAP_TOOL) -c ${MAP_CFG_FILE} -m ${FLSH_MODEL} -b ${TARGET_FILE} ${HEX} ${LODBASE} ${STDOUT_NULL}
	$(Q)python $(LODCOMBINE_TOOL) merge --platform $(PLATFORM_LOD_FILE) --app $(LOD_FILE) --out $(WITH_PLT_LOD_FILE)
	$(ECHO) "BUILD OK"

${BIN}: $(OBJ)
	@echo "$(shell ./build-libcsdk.sh)"
	$(ECHO) "LINK $@"
	$(Q)test -f ${LD_FILE} && chmod +w ${LD_FILE} || echo ""
	$(Q)${LDPP} ${LDPPFLAGS} ${LD_SRC} > ${LD_FILE}
	$(Q)${LD} -nostdlib -o ${BIN} ${OFORMAT} $^ ${FULL_SRC_OBJECTS} ${LD_OPTIONAL_OBJECTS} \
	${LDFLAG_USED_ELF_FILES}	\
	--script ${LD_FILE} \
	$(LDFLAGS) \
	-Map ${MAP}

bin: ${BIN}
	$(Q)${MKDIR} -p ${BIN_PATH}
	$(Q)${CP} -f ${BIN} ${BIN_PATH}
	$(Q)${CP} -f ${PLATFORM_ELF_PATH} ${BIN_PATH}
	$(Q)${OBJCOPY} --strip-symbols=${STRIP_SYMBOL_FILE} -N send_packet -N calloc -N ff_oem2uni -N ff_uni2oem ${BIN_PATH}/${FIRMWARE_NAME}.elf ${BIN_PATH}/${FIRMWARE_NAME}.elf
	$(Q)${ELFCOMBINE_TOOL} -e1 ${BIN_PATH}/${FIRMWARE_NAME}.elf -e2 ${BIN_PATH}/${PLATFORM_ELF_NAME} -o ${BIN_PATH}/${FIRMWARE_NAME}_final.elf
	$(Q)${RM} -f ${BIN_PATH}/${FIRMWARE_NAME}.elf
	$(Q)${RM} -f ${BIN_PATH}/${PLATFORM_ELF_NAME}

${HEX}:bin
	$(Q)${MAKE} ${BIN}
	$(Q)$(OBJCOPY) ${SREC} ${BIN} ${HEX}

.PHONY: targetgen
TARGET_FILE := ${BUILD_PATH_ABS}/targetgen
targetgen:
	@${ECHO} "#############################################"     > $(TARGET_FILE)
	@${ECHO} "# Target component definitions"                   >> $(TARGET_FILE)
	@${ECHO} "#############################################"    >> $(TARGET_FILE)
	@${ECHO} "FLSH_MODEL:=           $(FLSH_MODEL)"             >> $(TARGET_FILE)
	@${ECHO} "#############################################"    >> $(TARGET_FILE)
	@${ECHO} "# Flash Mapping"                                  >> $(TARGET_FILE)
	@${ECHO} "#############################################"    >> $(TARGET_FILE)
	@${ECHO} "FLASH_SIZE:=           $(FLASH_SIZE)"             >> $(TARGET_FILE)
	@${ECHO} "RAM_SIZE:=             $(RAM_SIZE)"               >> $(TARGET_FILE)
	@${ECHO} "RAM_PHY_SIZE:=         $(RAM_PHY_SIZE)"           >> $(TARGET_FILE)
	@${ECHO} "CALIB_BASE:=           $(CALIB_BASE)"             >> $(TARGET_FILE)
	@${ECHO} "FACT_SETTINGS_BASE:=   $(FACT_SETTINGS_BASE)"     >> $(TARGET_FILE)
	@${ECHO} "CODE_BASE:=            $(CODE_BASE)"              >> $(TARGET_FILE)
	@${ECHO} "USER_DATA_BASE:=       $(USER_DATA_BASE)"         >> $(TARGET_FILE)
	@${ECHO} "USER_DATA_SIZE:=       $(USER_DATA_SIZE)"         >> $(TARGET_FILE)
	@${ECHO} "ROMU_FLASH_LAYOUT:=    $(ROMU_FLASH_LAYOUT)"      >> $(TARGET_FILE)
	@${ECHO} "USER_BASE:=            $(USER_ROM_BASE)"     		>> $(TARGET_FILE)
	@${ECHO} "USER_SIZE:=            $(USER_ROM_SIZE)"     		>> $(TARGET_FILE)
	@${ECHO} ""                                                 >> $(TARGET_FILE)
	@${ECHO} "#############################################"    >> $(TARGET_FILE)
	@${ECHO} "# others"                                  	    >> $(TARGET_FILE)
	@${ECHO} "#############################################"    >> $(TARGET_FILE)


include $(TOP)/py/mkrules.mk