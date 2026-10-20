# Step 1: Create Workspace
```bash
mkdir -p ~/Desktop/OneWifi_Workspace
cd ~/Desktop/OneWifi_Workspace
```
# Step 2: Clone Required Repositories
All repositories are cloned inside ~/Desktop/OneWifi_Workspace
```bash
git clone https://github.com/rdkcentral/OneWifi.git
git clone https://github.com/rdkcentral/rdk-wifi-hal.git
git clone https://github.com/rdkcentral/rdkb-halif-wifi.git halinterface
git clone https://github.com/xmidt-org/trower-base64.git
git clone https://github.com/rdkcentral/unified-wifi-mesh.git
```
# Step 3: Enter OneWifi
```bash
cd ~/Desktop/OneWifi_Workspace/OneWifi
```
# Step 4: Navigate to the OneWifi directory
```bash
cd ~/Desktop/OneWifi_Workspace/OneWifi
```
## Create and open Makefile.custom
vim Makefile.custom
```bash
# Custom Makefile for OneWifi

CC := gcc
CXX := g++

PROJECT_DIR := $(CURDIR)
SOURCE_DIR := $(PROJECT_DIR)/source
BUILD_DIR := $(PROJECT_DIR)/build_custom
OBJ_DIR := $(BUILD_DIR)/obj

CFLAGS := -Wall -Wextra -g \
    -I$(PROJECT_DIR) \
    -I$(PROJECT_DIR)/include \
    -I$(PROJECT_DIR)/include/tr_181/ml \
    -I$(PROJECT_DIR)/lib/const \
    -I$(PROJECT_DIR)/lib/common \
    -I$(PROJECT_DIR)/lib/log \
    -I$(PROJECT_DIR)/lib/ds \
    -I$(PROJECT_DIR)/lib/ovsdb \
    -I$(PROJECT_DIR)/lib/pjs \
    -I$(PROJECT_DIR)/lib/json_util \
    -I$(PROJECT_DIR)/lib/inc \
    -I$(PROJECT_DIR)/lib/osa \
    -I$(PROJECT_DIR)/lib/schema \
    -I$(PROJECT_DIR)/lib/datapipeline \
    -I$(PROJECT_DIR)/lib/pktgen \
    -I$(PROJECT_DIR)/source/utils \
    -I$(PROJECT_DIR)/source/utils/mtrx/inc \
    -I$(PROJECT_DIR)/source/utils/quality_mgr/inc \
    -I$(PROJECT_DIR)/source/utils/math_utils/inc \
    -I$(PROJECT_DIR)/source/ccsp \
    -I$(PROJECT_DIR)/source/stats \
    -I$(PROJECT_DIR)/source/stubs \
    -I$(PROJECT_DIR)/source/db \
    -I$(PROJECT_DIR)/source/core \
    -I$(PROJECT_DIR)/source/core/services \
    -I$(PROJECT_DIR)/source/dml/linux \
    -I$(PROJECT_DIR)/source/dml/tr_181/sbapi \
    -I$(PROJECT_DIR)/source/dml/tr_181/ml \
    -I$(PROJECT_DIR)/source/dml/wifi_ssp \
    -I$(PROJECT_DIR)/source/sampleapps \
    -I$(PROJECT_DIR)/source/apps \
    -I$(PROJECT_DIR)/source/apps/analytics \
    -I$(PROJECT_DIR)/source/apps/multi_ap \
    -I$(PROJECT_DIR)/source/apps/blaster \
    -I$(PROJECT_DIR)/source/apps/cac \
    -I$(PROJECT_DIR)/source/apps/csi \
    -I$(PROJECT_DIR)/source/apps/harvester \
    -I$(PROJECT_DIR)/source/apps/levl \
    -I$(PROJECT_DIR)/source/apps/memwraptool \
    -I$(PROJECT_DIR)/source/apps/sta_mgr \
    -I$(PROJECT_DIR)/source/apps/motion \
    -I$(PROJECT_DIR)/source/apps/ocs \
    -I$(PROJECT_DIR)/source/apps/sm \
    -I$(PROJECT_DIR)/source/apps/whix \
    -I$(PROJECT_DIR)/source/apps/easyconnect \
    -I$(PROJECT_DIR)/source/apps/em \
    -I$(PROJECT_DIR)/source/apps/linkquality \
    -I$(PROJECT_DIR)/source/apps/wifi_sensing/inc \
    -I$(PROJECT_DIR)/source/platform/linux \
    -I$(PROJECT_DIR)/source/platform/common \
    -I$(PROJECT_DIR)/source/platform/linux/he_bus/inc \
    -I$(PROJECT_DIR)/../halinterface/include \
    -I$(PROJECT_DIR)/../rdk-wifi-hal \
    -I$(PROJECT_DIR)/../rdk-wifi-hal/src \
    -I$(PROJECT_DIR)/../rdk-wifi-hal/include \
    -I$(PROJECT_DIR)/../rdk-wifi-libhostap/source/hostap-2.10/src \
    -I$(PROJECT_DIR)/../rdk-wifi-libhostap/source/hostap-2.10/wpa_supplicant \
    -I$(PROJECT_DIR)/../trower-base64/include \
    -I$(PROJECT_DIR)/../unified-wifi-mesh/inc \
    -I$(PROJECT_DIR)/build/linux/compat \
    -I$(PROJECT_DIR)/include/dml_webconfig \
    -I$(PROJECT_DIR)/source/services \
    -I$(PROJECT_DIR)/source/services/mesh \
    -I$(PROJECT_DIR)/source/services/public \
    -I$(PROJECT_DIR)/source/services/private \
    -I$(PROJECT_DIR)/lib/qm \
    -I$(SOURCE_DIR)/utils/math_utils/inc \
    -I/usr/include/cjson \
    -D_ANSC_LINUX \
    -D_COSA_INTEL_USG_ATOM_ \
    -DUSE_NOTIFY_COMPONENT \
    -DCISCO_XB3_PLATFORM_CHANGES \
    -DDUAL_CORE_XB3 \
    -DFEATURE_ONE_WIFI \
    -DWIFI_HAL_VERSION_3 \
    -DFEATURE_SUPPORT_PASSPOINT \
    -DFEATURE_SUPPORT_WEBCONFIG \
    -DRASPBERRY_PI_PORT \
    -DNL80211_ACL \
    -D_PLATFORM_RASPBERRYPI_ \
    -DEASY_MESH_NODE \
    -DEM_APP \
    -DONEWIFI_STA_MGR_APP_SUPPORT \
    -DPLATFORM_LINUX

CXXFLAGS := $(CFLAGS)

# Source files selected by the original Raspberry Pi Makefile
C_SOURCES := \
    $(SOURCE_DIR)/db/wifi_db.c \
    $(wildcard $(SOURCE_DIR)/core/services/*.c) \
    $(wildcard $(SOURCE_DIR)/apps/*.c) \
    $(wildcard $(SOURCE_DIR)/apps/linkquality/*.c) \
    $(wildcard $(SOURCE_DIR)/apps/em/*.c) \
    $(wildcard $(SOURCE_DIR)/stubs/*.c) \
    $(wildcard $(SOURCE_DIR)/stats/*.c) \
    $(PROJECT_DIR)/lib/common/util.c \
    $(SOURCE_DIR)/core/wifi_8021x.c \
    $(SOURCE_DIR)/core/wifi_ctrl_wifiapi_handlers.c \
    $(SOURCE_DIR)/core/wifi_ctrl.c \
    $(SOURCE_DIR)/core/wifi_ctrl_rbus_handlers.c \
    $(SOURCE_DIR)/core/wifi_ctrl_queue_handlers.c \
    $(SOURCE_DIR)/core/wifi_ctrl_webconfig.c \
    $(SOURCE_DIR)/core/wifi_multidoc_webconfig.c \
    $(SOURCE_DIR)/core/wifi_mgr.c \
    $(SOURCE_DIR)/core/wifi_events.c \
    $(SOURCE_DIR)/ccsp/ccsp.c \
    $(SOURCE_DIR)/utils/collection.c \
    $(SOURCE_DIR)/utils/scheduler.c \
    $(SOURCE_DIR)/utils/mtrx/src/mtrx.c \
    $(SOURCE_DIR)/dml/linux/wifi_dml.c \
    $(SOURCE_DIR)/platform/linux/misc.c \
    $(SOURCE_DIR)/platform/linux/bus.c \
    $(BUILD_DIR)/src/platform/common/common.c \
    $(SOURCE_DIR)/apps/sta_mgr/wifi_sta_mgr.c \
    $(SOURCE_DIR)/webconfig/wifi_webconfig.c

C_SOURCES := $(filter-out \
    $(SOURCE_DIR)/core/services/vap_svc_mesh_pod.c, \
    $(C_SOURCES))

WEBCONFIG_SOURCES := \
    $(wildcard $(SOURCE_DIR)/webconfig/wifi_decoder.c) \
    $(wildcard $(SOURCE_DIR)/webconfig/wifi_encoder.c) \
    $(wildcard $(SOURCE_DIR)/webconfig/wifi_webconfig_*.c) \
    $(SOURCE_DIR)/utils/wifi_util.c

WEBCONFIG_SOURCES += $(SOURCE_DIR)/webconfig/wifi_easymesh_translator.c
C_SOURCES += $(WEBCONFIG_SOURCES)

CPP_SOURCES := \
    $(wildcard $(SOURCE_DIR)/utils/math_utils/src/*.cpp) \
    $(wildcard $(SOURCE_DIR)/utils/quality_mgr/src/*.cpp)

HEBUS_SOURCES := $(wildcard $(SOURCE_DIR)/platform/linux/he_bus/src/*.c)

# Keep generated object files inside build_custom/obj
C_SOURCES += $(BUILD_DIR)/src/utils/jsonconv.c

C_OBJECTS := $(patsubst $(SOURCE_DIR)/%.c,$(OBJ_DIR)/%.o,$(filter $(SOURCE_DIR)/%.c,$(C_SOURCES)))
C_OBJECTS += $(OBJ_DIR)/platform/common/common.o
C_OBJECTS += $(patsubst $(SOURCE_DIR)/platform/linux/he_bus/src/%.c,$(OBJ_DIR)/platform/linux/he_bus/src/%.o,$(HEBUS_SOURCES))
C_OBJECTS += $(OBJ_DIR)/custom/util.o

CPP_OBJECTS := $(patsubst $(SOURCE_DIR)/%.cpp,$(OBJ_DIR)/%.o,$(CPP_SOURCES))

.PHONY: all info clean
.DEFAULT_GOAL := all

all: info OneWifi

OneWifi: $(C_OBJECTS) $(CPP_OBJECTS)
        $(CXX) -o $(BUILD_DIR)/OneWifi $(C_OBJECTS) $(CPP_OBJECTS) -L$(PROJECT_DIR)/install/lib -lwifihal -lm -luuid -lpthread -ldl -ljansson -lev -lssl -lcrypto -lnl-3 -lnl-genl-3 -lnl-route-3 -lcjson -lavro

info:
        @echo "OneWifi custom Makefile"
        @echo "C sources: $(words $(C_SOURCES))"
        @echo "C++ sources: $(words $(CPP_SOURCES))"
        @echo "C object files planned: $(words $(C_OBJECTS))"
        @echo "C++ object files planned: $(words $(CPP_OBJECTS))"

# Create the build directory
$(OBJ_DIR):
        mkdir -p $(OBJ_DIR)

# Compile C files
$(OBJ_DIR)/%.o: $(SOURCE_DIR)/%.c | $(OBJ_DIR)
        @mkdir -p $(dir $@)
        $(CC) $(CFLAGS) -c $< -o $@

# Compile copied common.c without modifying the original
$(OBJ_DIR)/platform/common/common.o: $(BUILD_DIR)/src/platform/common/common.c | $(OBJ_DIR)
        @mkdir -p $(dir $@)
        $(CC) $(CFLAGS) -c $< -o $@

# Compile custom jsonconv.c copy
$(OBJ_DIR)/custom/jsonconv.o: $(BUILD_DIR)/src/utils/jsonconv.c | $(OBJ_DIR)
        @mkdir -p $(dir $@)
        $(CC) $(CFLAGS) -c $< -o $@

# Compile lib/common/util.c
$(OBJ_DIR)/custom/util.o: $(PROJECT_DIR)/lib/common/util.c | $(OBJ_DIR)
        @mkdir -p $(dir $@)
        $(CC) $(CFLAGS) -c $< -o $@

# Compile C++ files
$(OBJ_DIR)/%.o: $(SOURCE_DIR)/%.cpp | $(OBJ_DIR)
        @mkdir -p $(dir $@)
        $(CXX) $(CXXFLAGS) -c $< -o $@

clean:
        rm -rf $(BUILD_DIR)
```
Save the file and exit.

Verify the Makefile:
```bash
ls -l Makefile.custom
cat Makefile.custom
```
# Step 5: Build OneWifi Using Makefile.custom
## Navigate to the OneWifi directory
```bash
cd ~/Desktop/OneWifi_Workspace/OneWifi
```
## Build OneWifi using four parallel jobs
```bash
make -f Makefile.custom -j4
```
# Step 6: Fix Missing Hostap Header Include Path
During compilation, the following header was not found:

common/ieee802_11_defs.h

## Navigate to the OneWifi directory
```bash
cd ~/Desktop/OneWifi_Workspace/OneWifi
```
Check the Hostap source directory:
```bash
ls ~/hostapd-2.10/src/common/ieee802_11_defs.h
```
## Open `Makefile.custom`:
```bash
vim Makefile.custom
```
Add the following include path to the existing `CFLAGS` configuration:

### makefile
```bash
-I$(HOME)/hostapd-2.10/src
```
Save the file and exit.

## Verify the include path:
```bash
grep -n 'hostapd-2.10/src' Makefile.custom
```
## Rebuild OneWifi:
```bash
make -f Makefile.custom -j4
```
This include-path correction allowed compilation to proceed past the missing-header error. Compatibility with the required RDK-specific Hostap implementation still needs to be verified.

# Step 7: Correct the Source File Path
During compilation, the build initially failed because the Makefile referenced `common.c` under the generated build directory instead of the actual source directory.

**Incorrect source path:**
### makefile
```bash
$(BUILD_DIR)/src/platform/common/common.c
```
**Correct source path:**
### makefile
```bash
$(SOURCE_DIR)/platform/common/common.c
```
## Navigate to the OneWifi directory:
```bash
cd ~/Desktop/OneWifi_Workspace/OneWifi
```
## Check the source file location:
```bash
find source -type f -path "*/platform/common/common.c"
```
## Open `Makefile.custom`:
```bash
vim Makefile.custom
```
## Update the source-list entry to:
### makefile
```bash
$(SOURCE_DIR)/platform/common/common.c \
```
## Update the object dependency rule to:
### makefile
```bash
$(OBJ_DIR)/platform/common/common.o: $(SOURCE_DIR)/platform/common/common.c | $(OBJ_DIR)
```
## Alternatively, replace the incorrect path automatically using:
```bash
sed -i 's|$(BUILD_DIR)/src/platform/common/common.c|$(SOURCE_DIR)/platform/common/common.c|g' Makefile.custom
```
## Verify the changes:
```bash
grep -n 'platform/common/common.c' Makefile.custom
```
Save and exit Vim if you edited the file manually by pressing `Esc`, typing `:wq`, and pressing `Enter`.

## Rebuild OneWifi:
```bash
make -f Makefile.custom -j4
```
This correction ensures that the Makefile references the actual `common.c` source file when compiling the platform code.## Step 8: Fix Duplicate Object File Entry

## Issue
## During the linking stage, the following error occurred:
multiple definition of `platform_init'
The object file `build_custom/obj/platform/common/common.o` was included twice in the final linker command.

## Root Cause
The object file was already generated through the `C_SOURCES` list and converted into an object entry by the `C_OBJECTS` rule. It was then added again explicitly using:

### makefile
```bash
C_OBJECTS += $(OBJ_DIR)/platform/common/common.o
```
This caused the same object file to appear twice in the linker command.

## Fix
## Navigate to the OneWifi directory:
```bash
cd ~/Desktop/OneWifi_Workspace/OneWifi
```
## Remove the redundant object-list entry from `Makefile.custom`:
```bash
sed -i '\|^C_OBJECTS += $(OBJ_DIR)/platform/common/common.o$|d' Makefile.custom
```
## Verification
## Check whether the duplicate entry remains:
```bash
grep -n 'C_OBJECTS.*platform/common/common.o' Makefile.custom
```
No output indicates that the explicit duplicate entry has been removed.

## Rebuild
## Run the custom Makefile again:
```bash
make -f Makefile.custom -j4
```
# Step 8: Hostap Setup
```bash
cd ~/Desktop/OneWifi_Workspace/OneWifi
sh build/linux/rpi/setup.sh
```
## Verify that the setup completes successfully with All patches applied successfully.

# Step 9: Verify Dependencies

**RDK Wi-Fi HAL**
```bash
ls ~/Desktop/OneWifi_Workspace/rdk-wifi-hal/src
```
**Raspberry Pi platform source**
```bash
ls ~/Desktop/OneWifi_Workspace/rdk-wifi-hal/platform/raspberry-pi
```
**HAL interface headers**
```bash
ls ~/Desktop/OneWifi_Workspace/halinterface/include
```
**Trower Base64**
```bash
ls ~/Desktop/OneWifi_Workspace/trower-base64
```
**Hostap source**
```bash
ls ~/Desktop/OneWifi_Workspace/rdk-wifi-libhostap/source/hostap-2.10/src
```
# Step 10: Build libwifihal.a
## Navigate to the OneWifi directory:
```bash
cd ~/Desktop/OneWifi_Workspace/OneWifi
```
## Build the HAL static library using the Raspberry Pi Makefile:
```bash
make -f build/linux/rpi/makefile -j"$(nproc)" "$(pwd)/install/lib/libwifihal.a"
```
## Verify the generated archive:
```bash
ls -lh install/lib/libwifihal.a
ar -t install/lib/libwifihal.a | head -20
file install/lib/libwifihal.a
```
# Step 11: Build OneWifi Using Makefile.custom
## Navigate to the OneWifi directory:
```bash
cd ~/Desktop/OneWifi_Workspace/OneWifi
```
## Build the OneWifi executable:
```bash
make -f Makefile.custom -j"$(nproc)"
```
## Verify the generated binary:
```bash
ls -lh build_custom/OneWifi
```
file build_custom/OneWifi
