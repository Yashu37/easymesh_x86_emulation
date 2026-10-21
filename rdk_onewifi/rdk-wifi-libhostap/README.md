# libhostap-workspace — Hostapd Build and Static Library Creation

# Step 1: Navigate to the Workspace
Use the same workspace path throughout this procedure.

```bash
cd ~/Desktop/libhostap-workspace
pwd
ls -la
```
# Step 2: Enter rdk-wifi-libhostap 
```bash
cd ~/Desktop/libhostap-workspace/rdk-wifi-libhostap
```
# Step 3: Navigate to the rdk-wifi-libhostap directory
Create and Open Makefile:

vim Makefile
```bash

HOSTAP_DIR := $(CURDIR)/source/hostap-2.10
OBJ_DIR := $(HOSTAP_DIR)/build/src
BUILD_DIR := $(CURDIR)/build
LIB := $(BUILD_DIR)/libhostap.a

.PHONY: all clean

all:
        mkdir -p "$(BUILD_DIR)/archive_objs"
        rm -f "$(BUILD_DIR)/archive_objs/"*.o "$(LIB)"
        @set -e; \
        find "$(OBJ_DIR)" -type f -name '*.o' -print | while read obj; do \
            rel=$${obj#$(OBJ_DIR)/}; \
            name=$$(echo "$$rel" | tr '/' '_'); \
            cp "$$obj" "$(BUILD_DIR)/archive_objs/$$name"; \
        done
        ar rcs "$(LIB)" "$(BUILD_DIR)"/archive_objs/*.o
        ranlib "$(LIB)"
        @echo "Created $(LIB)"

clean:
        rm -rf "$(BUILD_DIR)"

```

# Step 4: Check the Hostap Configuration

Navigate to the Hostap source directory:

```bash
cd ~/Desktop/libhostap-workspace/rdk-wifi-libhostap/source/hostap-2.10
```

Check the available configuration files:

```bash
ls -a | grep -E 'defconfig|\.config'
ls hostapd/defconfig
```

If `hostapd/.config` does not exist, create it from the default configuration:

```bash
cp hostapd/defconfig hostapd/.config
```

Do not overwrite an existing configuration unnecessarily, because it may contain required project-specific settings.

Check the TLS configuration:

```bash
grep -nE '^CONFIG_TLS=|^#CONFIG_TLS=' hostapd/.config
```

Hostapd defaults to OpenSSL when `CONFIG_TLS` is not explicitly set. Check the build output to confirm which crypto implementation is actually compiled.

# Step 5: Enable WPS

If the RDK-specific source code requires WPS-related configuration fields, ensure WPS is enabled in `hostapd/.config`.

```bash
cd ~/Desktop/libhostap-workspace/rdk-wifi-libhostap/source/hostap-2.10

grep -n 'CONFIG_WPS' hostapd/.config
```

If the file contains `#CONFIG_WPS=y`, enable it:

```bash
sed -i 's/^#CONFIG_WPS=y/CONFIG_WPS=y/' hostapd/.config
```

Verify the change:

```bash
grep -n '^CONFIG_WPS=y' hostapd/.config
```

Expected output:

```text
CONFIG_WPS=y
```

# Step 6: Enable the RDK Greylist Feature Flag

The RDK-specific greylist code requires the `FEATURE_SUPPORT_RADIUSGREYLIST` macro to be defined when compiling the relevant source files.

Add the feature flag to `src/ap/Makefile`:

```bash
cd ~/Desktop/libhostap-workspace/rdk-wifi-libhostap/source/hostap-2.10

sed -i '1i CFLAGS += -DFEATURE_SUPPORT_RADIUSGREYLIST' src/ap/Makefile
```

Verify the change:

```bash
head -5 src/ap/Makefile
```

The output should include:

```makefile
CFLAGS += -DFEATURE_SUPPORT_RADIUSGREYLIST
```

Verify both Makefiles:

```bash
grep -n 'FEATURE_SUPPORT_RADIUSGREYLIST' \
  src/ap/Makefile hostapd/Makefile
```

Do not add duplicate entries if the flag is already present.

# Step 7: Declare `greylist_get_cmmac()`

The function `greylist_get_cmmac()` is implemented in `src/ap/greylist.c`. A separate implementation is not needed.

The issue was that `src/ap/greylist.h` did not declare the function, even though `src/ap/ieee802_1x.c` calls it.

Open the header:

```bash
cd ~/Desktop/libhostap-workspace/rdk-wifi-libhostap/source/hostap-2.10
vim src/ap/greylist.h
```

Find the existing declaration:

```c
u8 greylist_get_client_snr(...);
```

Add the following declaration after the existing greylist function declarations, using the actual declaration in your header as the reference:

```c
int greylist_get_cmmac(void);
```

Save and exit Vim:

1. Press `Esc`.
2. Type `:wq`.
3. Press `Enter`.

Verify the declaration and implementation:

```bash
grep -n 'greylist_get_cmmac' src/ap/greylist.h src/ap/greylist.c
```

# Step 8: Compile the Hostapd Components

Navigate to the Hostap source directory:

```bash
cd ~/Desktop/libhostap-workspace/rdk-wifi-libhostap/source/hostap-2.10
```

Check that the WPS setting is enabled:

```bash
grep -n '^CONFIG_WPS=y' hostapd/.config
```

Compile Hostapd:

```bash
make -C hostapd
```

## Step 9.1: Fix the `global` Declaration

The header `src/ap/hostapd.h` declares:

```c
extern struct hapd_global global;
```
Therefore, the definition in `hostapd/main.c` must not be `static`.

Open the file:

```bash
cd ~/Desktop/libhostap-workspace/rdk-wifi-libhostap/source/hostap-2.10
vim hostapd/main.c
```
Ensure the file contains the following definition outside any disabled `#if 0` block:

```c
struct hapd_global global;
```

Do not use:

```c
static struct hapd_global global;
```

Save the file with `Esc`, `:wq`, and `Enter`.

## 9.2 Ensure the Greylist Object Is Linked

Check whether the greylist object is included in the Hostapd Makefile:

```bash
grep -n -C 3 'greylist' hostapd/Makefile
```

If it is not already included in the object list, add the following entry.

Edit the Makefile:

```bash
cd ~/Desktop/libhostap-workspace/rdk-wifi-libhostap/source/hostap-2.10
vim hostapd/Makefile
```

Near the other `OBJS += ../src/ap/...` entries at the top, add:

```makefile
OBJS += ../src/ap/greylist.o
```

Save with `Esc`, then `:wq`, then `Enter`.

## Step 9.3: Rebuild Hostapd and Capture the Build Log

Run:

```bash
cd ~/Desktop/libhostap-workspace/rdk-wifi-libhostap/source/hostap-2.10

make -C hostapd clean
set -o pipefail
make -C hostapd V=1 2>&1 | tee hostapd_build.log
```

## Step 9.4: Check the Build Errors

Search the build log for compilation errors, undefined references, and multiple definitions:

```bash
grep -nE 'undefined reference|error:|multiple definition|make: \*\*\*' hostapd_build.log | tail -30
```

## Step 9.5: Fix the AES-OMAC1 Linker Issue

Edit `hostapd/Makefile`:

```bash
vim hostapd/Makefile
```

Locate the conditional block around line 950:

```makefile
ifneq ($(CONFIG_TLS), openssl)
ifneq ($(CONFIG_TLS), linux)
ifneq ($(CONFIG_TLS), wolfssl)
AESOBJS += ../src/crypto/aes-omac1.o
endif
endif
endif
```

Replace that entire block with:

```makefile
AESOBJS += ../src/crypto/aes-omac1.o
```

This ensures that the AES-OMAC1 implementation is included even when OpenSSL is selected.

Save the file in Vim by pressing `Esc`, typing `:wq`, and pressing `Enter`.

## Step 9.6: Rebuild Hostapd After the Fix

Run the following commands from the Hostap source directory:

```bash
cd ~/Desktop/libhostap-workspace/rdk-wifi-libhostap/source/hostap-2.10

make -C hostapd clean
set -o pipefail
make -C hostapd V=1 2>&1 | tee hostapd_build.log
```

## Step 9.7: Verify the Hostapd Build

Check whether any errors remain:

```bash
grep -nE 'error:|undefined reference|multiple definition|make: \*\*\*' hostapd_build.log | tail -30
```

Check the build command's exit status immediately after the build finishes:

```bash
echo $?
```

Check whether the executable was generated:

```bash
ls -lh hostapd/hostapd
file hostapd/hostapd
```

A successful build should return exit status `0` and show the generated `hostapd/hostapd` executable.

## Step 9.8: Navigate to the Workspace Root

After verifying the Hostapd executable, navigate to the outer workspace:

```bash
cd ~/Desktop/libhostap-workspace/rdk-wifi-libhostap
pwd
```

## Step 9.9: Check the Compiled Object Files

Count the object files generated in the Hostapd build directory:

```bash
find source/hostap-2.10/build/hostapd -type f -name '*.o' | wc -l
```

Count the object files in the Hostap source build directory:

```bash
find source/hostap-2.10/build/src -type f -name '*.o' 2>/dev/null | wc -l
```

These commands verify the presence of compiled object files before proceeding to the outer static library build.

# Step 10: Generate `libhostap.a`

Return to the outer workspace:

```bash
cd ~/Desktop/libhostap-workspace/rdk-wifi-libhostap
```

Run the root Makefile:

```bash
make
```

The root Makefile packages existing object files into the static archive. It does not automatically compile Hostap source files unless compilation rules are explicitly defined.

# Step 13. Verify the Static Library

Check the generated archive:

```bash
cd ~/Desktop/libhostap-workspace/rdk-wifi-libhostap

ls -lh build/libhostap.a
file build/libhostap.a
```

