
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

