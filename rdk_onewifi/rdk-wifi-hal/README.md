# RDK Wi-Fi HAL Custom Static Library Build

# Step 1: Navigate to the HAL Workspace

Run all the following build commands from the HAL repository directory:

```bash
cd ~/Desktop/exp
pwd
ls -la
```
# Step 2: Enter rdk-wifi-hal
```bash
cd ~/Desktop/exp/rdk-wifi-hal
```
# Step 3: Navigate to the rdk-wifi-hal directory

Create and Open Makefile.custom

vim Makefile.custom
```bash
CC := gcc
AR := ar
RANLIB := ranlib

ROOT := $(CURDIR)
SRC := $(ROOT)/src
BUILD := $(ROOT)/build_custom
OBJ := $(BUILD)/obj
LIB := $(BUILD)/librdk_wifihal.a

HOSTAP_ROOT := /tmp/hostap211-patch-work/source/hostap-2.11/src
HOSTAP_SUPPLICANT_ROOT := /tmp/hostap211-patch-work/source/hostap-2.11/wpa_supplicant

CPPFLAGS += -D_ANSC_LINUX -D_ANSC_USER -D_ANSC_LITTLE_ENDIAN_
CPPFLAGS += -DHOSTAPD_2_11
CPPFLAGS += -I$(ROOT)/include
CPPFLAGS += -I$(ROOT)/src
CPPFLAGS += -I$(ROOT)/util_crypto
CPPFLAGS += -I$(ROOT)/../util
CPPFLAGS += -I$(HOSTAP_ROOT)
CPPFLAGS += -I$(HOSTAP_ROOT)/utils
CPPFLAGS += -I$(HOSTAP_ROOT)/ap
CPPFLAGS += -I$(HOSTAP_ROOT)/drivers
CPPFLAGS += -I$(ROOT)/../halinterface/include
CPPFLAGS += -I/usr/include/cjson
CPPFLAGS += -I$(ROOT)/../rdk-wifi-libhostap/source/hostap-2.11-test
CPPFLAGS += -I/usr/include/libnl3
CPPFLAGS += -DCONFIG_WEP
CPPFLAGS += -DCONFIG_HS20
CPPFLAGS += -DFEATURE_SUPPORT_RADIUSGREYLIST
CPPFLAGS += -DWIFI_HAL_VERSION_3
CPPFLAGS += -DCONFIG_WPS
CPPFLAGS += -DRDK_ONEWIFI
CPPFLAGS += -DPLATFORM_LINUX
CPPFLAGS += -D_PLATFORM_BANANAPI_R4_
CPPFLAGS += -DBANANA_PI_PORT
CPPFLAGS += -I$(HOSTAP_SUPPLICANT_ROOT)

CFLAGS += -Wall -Wextra -Wno-unused-parameter
CFLAGS += -Wno-pointer-sign -Wno-sign-compare -Wno-type-limits
CFLAGS += -Wno-format-truncation -Wno-discarded-qualifiers
CFLAGS += -fPIC

SOURCES := \
        $(SRC)/wifi_hal_rx_test.c \
        $(SRC)/wifi_hal_data_rx.c \
        $(SRC)/wifi_hal_dpp.c \
        $(SRC)/wifi_hal_anqp.c \
        $(SRC)/wifi_hal_frame_test.c \
        $(ROOT)/util_crypto/aes_siv.c \
        $(SRC)/wifi_tlv.c \
        $(SRC)/wifi_hal_rdk_util.c \
        $(SRC)/wifi_hal.c \
        $(SRC)/wifi_hal_hostapd.c \
        $(SRC)/wifi_hal_nl80211.c \
        $(SRC)/wifi_hal_nl80211_events.c \
        $(SRC)/wifi_hal_nl80211_utils.c \
        $(SRC)/collection.c \
        $(SRC)/wifi_hal_mgmt_rx_one_wifi.c \
        $(SRC)/wifi_hal_wnm_rrm.c

OBJECTS := $(patsubst $(ROOT)/%.c,$(OBJ)/%.o,$(SOURCES))

.PHONY: all clean

all: $(LIB)

$(LIB): $(OBJECTS)
        @mkdir -p $(BUILD)
        $(AR) rcs $@ $(OBJECTS)
        $(RANLIB) $@
        @echo "Created $(LIB)"

$(OBJ)/%.o: $(ROOT)/%.c
        @mkdir -p $(dir $@)
        $(CC) $(CPPFLAGS) $(CFLAGS) -c $< -o $@

clean:
        rm -rf $(BUILD)
```

Verify that the custom Makefile exists:

```bash
ls -l Makefile.custom
```

# Step 4: Configure the Hostap 2.11 Source Path

Open Makefile.custom from the RDK Wi-Fi HAL workspace:
```bash
cd ~/Desktop/exp/rdk-wifi-hal
vim Makefile.custom
```
Locate the existing Hostap path definition near the beginning of the file:

Replace this:
```bash
HOSTAP_ROOT := /tmp/hostap211-patch-work/source/hostap-2.11/src
HOSTAP_SUPPLICANT_ROOT := /tmp/hostap211-patch-work/source/hostap-2.11/wpa_supplicant
```

With the following:
```bash
HOSTAP_ROOT ?= $(HOME)/hostap-2.11
HOSTAP_SRC := $(HOSTAP_ROOT)/src
HOSTAP_SUPPLICANT_ROOT := $(HOSTAP_ROOT)/wpa_supplicant
Ensure the following include paths are present in `Makefile.custom`:
```
In the same Makefile.custom, locate the existing CPPFLAGS entries and ensure the following include paths are present:
```bash
CPPFLAGS += -I$(HOSTAP_ROOT)
CPPFLAGS += -I$(HOSTAP_SRC)
CPPFLAGS += -I$(HOSTAP_SRC)/utils
CPPFLAGS += -I$(HOSTAP_SRC)/ap
CPPFLAGS += -I$(HOSTAP_SRC)/drivers
CPPFLAGS += -I$(HOSTAP_SUPPLICANT_ROOT)
```
These include paths allow the compiler to locate the required Hostap and WPA supplicant headers.

Note: If your existing Makefile uses HOSTAP_ROOT to point directly to the src directory, its include paths may instead look like this:
```bash
CPPFLAGS += -I$(HOSTAP_ROOT)/utils
CPPFLAGS += -I$(HOSTAP_ROOT)/ap
CPPFLAGS += -I$(HOSTAP_ROOT)/drivers
```
Use one consistent path configuration. With the recommended configuration above, HOSTAP_ROOT points to the Hostap root directory containing hostapd/, src/, and wpa_supplicant/.
Keep existing compiler flags and other project include paths unchanged. Avoid duplicate include entries.

Save the file in Vim by pressing Esc, typing :wq, and pressing Enter.

Verify the configuration:

```bash
grep -nE 'HOSTAP_ROOT|HOSTAP_SRC|HOSTAP_SUPPLICANT_ROOT|CPPFLAGS.*HOSTAP' Makefile.custom
```

### Required Hostap source

The source tree used for the current build is:

```text
/tmp/hostap211-patch-work/source/hostap-2.11
```

Check the required directories and headers:

```bash
ls -ld /tmp/hostap211-patch-work/source/hostap-2.11
ls -l /tmp/hostap211-patch-work/source/hostap-2.11/hostapd/ctrl_iface.h
ls -l /tmp/hostap211-patch-work/source/hostap-2.11/src/utils/includes.h
```

The source tree must contain the required compatibility patches. The `/tmp` path is specific to the current machine and may not exist on other systems.

# Step 5: Check the HAL Source List

```bash
grep -n -A25 '^SOURCES' Makefile.custom
```

Verify that the expected source files exist:

```bash
ls -l src/wifi_hal.c
ls -l src/wifi_hal_hostapd.c
ls -l src/wifi_hal_nl80211.c
ls -l util_crypto/aes_siv.c
```

The source list in `Makefile.custom` determines which HAL source files are compiled into the static library.

#Step 6: Build the Static Library

**Run `make` from the HAL repository root**, where `Makefile.custom` is located.

```bash
cd ~/Desktop/exp/rdk-wifi-hal
```

Use the patched Hostap 2.11 source path:

```bash
set -o pipefail

make -f Makefile.custom \
  HOSTAP_ROOT=/tmp/hostap211-patch-work/source/hostap-2.11 \
  -j"$(nproc)" 2>&1 | tee build_patched.log
```

The command:

- Uses `Makefile.custom` instead of the default `Makefile`.
- Passes the Hostap 2.11 source directory to the build.
- Compiles in parallel using the available CPU cores.
- Displays build output and saves it to `build_patched.log`.
- Uses `pipefail` so a failed build is not hidden by `tee`.

Check the build exit status immediately after the command:

```bash
echo $?
```

A status of `0` indicates that the build command succeeded.

Review errors and warnings:

```bash
grep -nE 'error:|fatal error:|warning:|undefined reference' build_patched.log
```

If the build fails, fix the reported errors before proceeding.

# Step 7: Verify the Generated Library

The expected static library is:

```text
build_custom/librdk_wifihal.a
```

From the HAL repository root, run:

```bash
cd ~/Desktop/exp/rdk-wifi-hal

ls -lh build_custom/librdk_wifihal.a
file build_custom/librdk_wifihal.a
```

