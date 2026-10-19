# Step 1: Create Workspace
mkdir -p ~/Desktop/OneWifi_Workspace
cd ~/Desktop/OneWifi_Workspace

# Step 2: Clone Required Repositories
All repositories are cloned inside ~/Desktop/OneWifi_Workspace

git clone https://github.com/rdkcentral/OneWifi.git
git clone https://github.com/rdkcentral/rdk-wifi-hal.git
git clone https://github.com/rdkcentral/rdkb-halif-wifi.git halinterface
git clone https://github.com/xmidt-org/trower-base64.git
git clone https://github.com/rdkcentral/unified-wifi-mesh.git

# Step 3: Enter OneWifi
cd ~/Desktop/OneWifi_Workspace/OneWifi

# Step 4: Navigate to the OneWifi directory
cd ~/Desktop/OneWifi_Workspace/OneWifi

## Create and open Makefile.custom
vim Makefile.custom

Add the required compiler flags (`CFLAGS`), header include paths, source files (`C_SOURCES`), object files (`C_OBJECTS`), compilation rules, and linker settings.
Save the file and exit.

Verify the Makefile:
ls -l Makefile.custom
cat Makefile.custom

# Step 5: Build OneWifi Using Makefile.custom
## Navigate to the OneWifi directory
cd ~/Desktop/OneWifi_Workspace/OneWifi

## Build OneWifi using four parallel jobs
make -f Makefile.custom -j4

# Step 6: Fix Missing Hostap Header Include Path
During compilation, the following header was not found:

common/ieee802_11_defs.h

## Navigate to the OneWifi directory
cd ~/Desktop/OneWifi_Workspace/OneWifi

Check the Hostap source directory:
ls ~/hostapd-2.10/src/common/ieee802_11_defs.h

## Open `Makefile.custom`:
vim Makefile.custom

Add the following include path to the existing `CFLAGS` configuration:

### makefile
-I$(HOME)/hostapd-2.10/src

Save the file and exit.

## Verify the include path:
grep -n 'hostapd-2.10/src' Makefile.custom

## Rebuild OneWifi:
make -f Makefile.custom -j4

This include-path correction allowed compilation to proceed past the missing-header error. Compatibility with the required RDK-specific Hostap implementation still needs to be verified.

# Step 7: Correct the Source File Path
During compilation, the build initially failed because the Makefile referenced `common.c` under the generated build directory instead of the actual source directory.

**Incorrect source path:**
### makefile
$(BUILD_DIR)/src/platform/common/common.c

**Correct source path:**
### makefile
$(SOURCE_DIR)/platform/common/common.c

## Navigate to the OneWifi directory:
cd ~/Desktop/OneWifi_Workspace/OneWifi

## Check the source file location:
find source -type f -path "*/platform/common/common.c"

## Open `Makefile.custom`:
vim Makefile.custom

## Update the source-list entry to:
### makefile
$(SOURCE_DIR)/platform/common/common.c \

## Update the object dependency rule to:
### makefile

$(OBJ_DIR)/platform/common/common.o: $(SOURCE_DIR)/platform/common/common.c | $(OBJ_DIR)

## Alternatively, replace the incorrect path automatically using:
sed -i 's|$(BUILD_DIR)/src/platform/common/common.c|$(SOURCE_DIR)/platform/common/common.c|g' Makefile.custom

## Verify the changes:
grep -n 'platform/common/common.c' Makefile.custom

Save and exit Vim if you edited the file manually by pressing `Esc`, typing `:wq`, and pressing `Enter`.

## Rebuild OneWifi:
make -f Makefile.custom -j4

This correction ensures that the Makefile references the actual `common.c` source file when compiling the platform code.## Step 8: Fix Duplicate Object File Entry

## Issue
## During the linking stage, the following error occurred:
multiple definition of `platform_init'
The object file `build_custom/obj/platform/common/common.o` was included twice in the final linker command.

## Root Cause
The object file was already generated through the `C_SOURCES` list and converted into an object entry by the `C_OBJECTS` rule. It was then added again explicitly using:

### makefile
C_OBJECTS += $(OBJ_DIR)/platform/common/common.o

This caused the same object file to appear twice in the linker command.

## Fix
## Navigate to the OneWifi directory:
cd ~/Desktop/OneWifi_Workspace/OneWifi

## Remove the redundant object-list entry from `Makefile.custom`:
sed -i '\|^C_OBJECTS += $(OBJ_DIR)/platform/common/common.o$|d' Makefile.custom

## Verification
## Check whether the duplicate entry remains:
grep -n 'C_OBJECTS.*platform/common/common.o' Makefile.custom

No output indicates that the explicit duplicate entry has been removed.

## Rebuild
## Run the custom Makefile again:
make -f Makefile.custom -j4

# Step 8: Hostap Setup
cd ~/Desktop/OneWifi_Workspace/OneWifi
sh build/linux/rpi/setup.sh

## Verify that the setup completes successfully with All patches applied successfully.

# Step 9: Verify Dependencies

**RDK Wi-Fi HAL**
ls ~/Desktop/OneWifi_Workspace/rdk-wifi-hal/src

**Raspberry Pi platform source**
ls ~/Desktop/OneWifi_Workspace/rdk-wifi-hal/platform/raspberry-pi

**HAL interface headers**
ls ~/Desktop/OneWifi_Workspace/halinterface/include

**Trower Base64**
ls ~/Desktop/OneWifi_Workspace/trower-base64

**Hostap source**
ls ~/Desktop/OneWifi_Workspace/rdk-wifi-libhostap/source/hostap-2.10/src

# Step 10: Build libwifihal.a
## Navigate to the OneWifi directory:
cd ~/Desktop/OneWifi_Workspace/OneWifi

## Build the HAL static library using the Raspberry Pi Makefile:
make -f build/linux/rpi/makefile -j"$(nproc)" "$(pwd)/install/lib/libwifihal.a"

## Verify the generated archive:
ls -lh install/lib/libwifihal.a
ar -t install/lib/libwifihal.a | head -20
file install/lib/libwifihal.a

# Step 11: Build OneWifi Using Makefile.custom
## Navigate to the OneWifi directory:
cd ~/Desktop/OneWifi_Workspace/OneWifi

## Build the OneWifi executable:
make -f Makefile.custom -j"$(nproc)"

## Verify the generated binary:
ls -lh build_custom/OneWifi

file build_custom/OneWifi
