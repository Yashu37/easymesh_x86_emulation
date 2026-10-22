# RBUS Custom Makefile Build Guide


# Step 1: Create Workspace

```bash
mkdir -p ~/Desktop/RBUS_Workspace
cd ~/Desktop/RBUS_Workspace
```

# step 2: Clone the RBUS Repository

```bash
git clone https://github.com/rdkcentral/rbus.git
cd rbus
```

If the repository is already cloned, navigate to the existing workspace:

```bash
cd ~/Desktop/RBUS_Workspace/rbus
```

Verify the repository contents:

```bash
pwd
ls
```

# Step 3: Install Build Dependencies

Install the compiler, Make, CMake, and the required development libraries:

```bash
sudo apt update
sudo apt install build-essential cmake pkg-config \
    libcjson-dev libmsgpack-dev
```

Verify the compiler and Make installation:

```bash
gcc --version
make --version
cmake --version
```

# Step 4: Build RBUS Using CMake

First, verify that the standard CMake build works.

From the repository root:

```bash
cd ~/Desktop/RBUS_Workspace/rbus

export RBUS_ROOT="$HOME/Desktop/RBUS_Workspace/rbus"
export RBUS_INSTALL_DIR="$RBUS_ROOT/install"

mkdir -p "$RBUS_INSTALL_DIR"

cmake -S . -B build/rbus \
    -DCMAKE_INSTALL_PREFIX="$RBUS_INSTALL_DIR/usr" \
    -DBUILD_FOR_DESKTOP=ON \
    -DCMAKE_BUILD_TYPE=Debug
```

Compile the project:

```bash
cmake --build build/rbus -j"$(nproc)"
```

Install the generated files:

```bash
cmake --install build/rbus
```

Verify the installed shared libraries:

```bash
find install/usr/lib -maxdepth 1 -name '*.so*' -ls
```

The CMake build provides a baseline for comparing the custom Makefile build.

# Step 5: Create the Custom Makefile

Create a file named `Makefile.custom` in the repository root:

```bash
cd ~/Desktop/RBUS_Workspace/rbus
vim Makefile.custom
```
vim Makefile.custom

```bash
CC := gcc

ROOT := $(CURDIR)
BUILD_DIR := build-make
INSTALL_DIR := $(ROOT)/install-make/usr

CFLAGS := -fPIC -Wall -Wextra -Wno-type-limits -fno-inline -g \
          -DRBUS_ALWAYS_ON -D_TIME_BITS=64 -D_FILE_OFFSET_BITS=64 \
          -DMSG_ROUNDTRIP_TIME=1

CPPFLAGS := -I$(ROOT)/include \
            -I$(ROOT)/src/rbus \
            -I$(ROOT)/src/core \
            -I$(ROOT)/src/rtmessage \
            -I$(ROOT)/src/session_manager \
            -I$(ROOT)/install/usr/include \
            -I/usr/include/cjson

LDFLAGS := -pthread
LDLIBS := -lcjson -lmsgpackc -pthread

RTMESSAGE_SRC := \
    rtConnection.c rtCipher.c rtLog.c rtError.c \
    rtMessageHeader.c rtEncoder.c rtMessage.c rtSocket.c \
    rtVector.c rtList.c rtTime.c rtString.c rtSemaphore.c \
    rtRetainable.c rtBase64.c rtThreadPool.c rtHashMap.c \
    rtMemory.c rtrouteBase.c local_benchmarking.c

RBUSCORE_SRC := rbuscore.c rbuscore_message.c

RBUS_SRC := \
    rbus.c rbus_handle.c rbus_message.c rbus_value.c \
    rbus_property.c rbus_object.c rbus_buffer.c rbus_filter.c \
    rbus_element.c rbus_valuechange.c rbus_subscriptions.c \
    rbus_tokenchain.c rbus_asyncsubscribe.c rbus_intervalsubscription.c

RTMESSAGE_OBJ := $(addprefix $(BUILD_DIR)/rtmessage/,$(RTMESSAGE_SRC:.c=.o))
RBUSCORE_OBJ := $(addprefix $(BUILD_DIR)/core/,$(RBUSCORE_SRC:.c=.o))
RBUS_OBJ := $(addprefix $(BUILD_DIR)/rbus/,$(RBUS_SRC:.c=.o))

.PHONY: all clean install

all: $(BUILD_DIR)/librtMessage.so \
     $(BUILD_DIR)/librbuscore.so \
     $(BUILD_DIR)/librbus.so

$(BUILD_DIR)/rtmessage/%.o: $(ROOT)/src/rtmessage/%.c
        mkdir -p $(@D)
        $(CC) $(CPPFLAGS) $(CFLAGS) -c $< -o $@

$(BUILD_DIR)/core/%.o: $(ROOT)/src/core/%.c
        mkdir -p $(@D)
        $(CC) $(CPPFLAGS) $(CFLAGS) -c $< -o $@

$(BUILD_DIR)/rbus/%.o: $(ROOT)/src/rbus/%.c
        mkdir -p $(@D)
        $(CC) $(CPPFLAGS) $(CFLAGS) -c $< -o $@

$(BUILD_DIR)/librtMessage.so: $(RTMESSAGE_OBJ)
        $(CC) -shared -Wl,-soname,librtMessage.so.0 -o $@ $^ $(LDFLAGS) -lcjson
        ln -sf librtMessage.so $@.0

$(BUILD_DIR)/librbuscore.so: $(RBUSCORE_OBJ) $(BUILD_DIR)/librtMessage.so
        $(CC) -shared -Wl,-soname,librbuscore.so.0 -o $@ $(RBUSCORE_OBJ) \
            -L$(BUILD_DIR) -lrtMessage $(LDLIBS) \
            -Wl,-rpath,'$$ORIGIN'
        ln -sf librbuscore.so $@.0

$(BUILD_DIR)/librbus.so: $(RBUS_OBJ) $(BUILD_DIR)/librbuscore.so
        $(CC) -shared -Wl,-soname,librbus.so.0 -o $@ $(RBUS_OBJ) \
            -L$(BUILD_DIR) -lrbuscore -lrtMessage $(LDLIBS) \
            -Wl,-rpath,'$$ORIGIN'
        ln -sf librbus.so $@.0

install: all
        mkdir -p $(INSTALL_DIR)/lib
        cp $(BUILD_DIR)/*.so $(INSTALL_DIR)/lib/

clean:
        rm -rf $(BUILD_DIR)
```

The Makefile should:

1. Define the compiler, build directory, include paths, compiler flags, and linker flags.
2. Compile the RTMessage source files.
3. Compile the RBUS Core source files.
4. Compile the RBUS source files.
5. Link the object files into the three shared libraries.
6. Set the appropriate SONAME for each library.
7. Create the `.so.0` symbolic links.
8. Provide a clean target to remove generated build files.


**Note:** The actual source lists, include paths, and linker flags must match the repository version and the dependencies installed on the build system.

# Step 6: Build Using the Custom Makefile

Navigate to the repository root:

```bash
cd ~/Desktop/RBUS_Workspace/rbus
```

Compile all components:

```bash
make -f Makefile.custom -j"$(nproc)"
```
The generated shared libraries and their symbolic links should appear in `build-make/`.

# Step 7: Verify the Generated Shared Libraries

### Check the generated files

```bash
ls -lh build-make/*.so*
```

Expected libraries:

```text
build-make/librtMessage.so
build-make/librtMessage.so.0
build-make/librbuscore.so
build-make/librbuscore.so.0
build-make/librbus.so
build-make/librbus.so.0
```

The `.so.0` entries should be symbolic links pointing to the corresponding `.so` files.

### Check the file types

```bash
file build-make/*.so
```

The outputs should be identified as ELF shared objects for the target architecture.

### Verify the symbolic links

```bash
ls -l build-make/*.so.0
```

Expected link relationships:

```text
librtMessage.so.0 -> librtMessage.so
librbuscore.so.0 -> librbuscore.so
librbus.so.0 -> librbus.so
```

### Verify the SONAME values

```bash
readelf -d build-make/librtMessage.so | grep SONAME
readelf -d build-make/librbuscore.so | grep SONAME
readelf -d build-make/librbus.so | grep SONAME
```

Expected SONAMEs:

```text
librtMessage.so.0
librbuscore.so.0
librbus.so.0
```

### Verify shared-library dependencies

Run:

```bash
LD_LIBRARY_PATH="$PWD/build-make${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}" \
    ldd build-make/librbus.so
```

Check that the RBUS libraries and external dependencies resolve correctly. Investigate any `not found` entries.

Also inspect the other libraries if needed:

```bash
LD_LIBRARY_PATH="$PWD/build-make${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}" \
    ldd build-make/librbuscore.so

LD_LIBRARY_PATH="$PWD/build-make${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}" \
    ldd build-make/librtMessage.so
```

# Step 8: Clean and Rebuild

To remove generated files, run:

```bash
make -f Makefile.custom clean
```

Then rebuild:

```bash
make -f Makefile.custom -j"$(nproc)"
```

Confirm that all three libraries and their symbolic links have been regenerated.


