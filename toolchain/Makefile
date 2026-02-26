# toolchain/Makefile
SHELL := /bin/bash

ifndef KERNEL_VER
$(error KERNEL_VER is not set. Pass KERNEL_VER=x.y.z)
endif

ifndef MUSL_CROSS_MAKE_REF
$(error MUSL_CROSS_MAKE_REF is not set. Pass MUSL_CROSS_MAKE_REF=x.y.z)
endif

# ---- Paths ----
OUT      ?= /opt/toolchains
PREFIX   ?= $(OUT)/aarch64-linux-musl
SYSROOT  ?= $(PREFIX)/aarch64-linux-musl/sysroot

WORKDIR  ?= /tmp/toolchain-build
MCM_DIR  ?= $(WORKDIR)/musl-cross-make
KERN_TAR ?= $(WORKDIR)/linux-$(KERNEL_VER).tar.xz
KERN_DIR ?= $(WORKDIR)/linux-$(KERNEL_VER)

TARGET   ?= aarch64-linux-musl

JOBS     := $(shell nproc)

.PHONY: toolchain clean kernel-headers musl-cross-make

toolchain: musl-cross-make kernel-headers
	@echo "Toolchain ready: $(PREFIX)"
	@echo "Sysroot ready:  $(SYSROOT)"
	@echo "Kernel headers installed into sysroot: $(SYSROOT)/usr/include"

musl-cross-make:
	mkdir -p $(WORKDIR) $(OUT)
	if [ ! -d "$(MCM_DIR)/.git" ]; then \
	  git clone https://github.com/richfelker/musl-cross-make.git "$(MCM_DIR)"; \
	fi
	cd "$(MCM_DIR)" && git fetch --all --tags
	cd "$(MCM_DIR)" && git checkout "$(MUSL_CROSS_MAKE_REF)"
	# Install toolchain into /opt/toolchains/aarch64-linux-musl
	cd "$(MCM_DIR)" && make -j$(JOBS) TARGET=$(TARGET) && make -j$(JOBS) TARGET=$(TARGET) install
	rm -rf "$(PREFIX)"
	mkdir -p "$(OUT)"
	mv "$(MCM_DIR)/output" "$(PREFIX)"

kernel-headers:
	mkdir -p $(WORKDIR)
	# Download kernel tarball only if missing
	if [ ! -f "$(KERN_TAR)" ]; then \
	  wget -O "$(KERN_TAR)" "https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-$(KERNEL_VER).tar.xz"; \
	fi
	rm -rf "$(KERN_DIR)"
	tar -C "$(WORKDIR)" -xf "$(KERN_TAR)"
	# Install UAPI headers into the toolchain sysroot
	mkdir -p "$(SYSROOT)"
	$(MAKE) -C "$(KERN_DIR)" ARCH=arm64 headers_install INSTALL_HDR_PATH="$(SYSROOT)/usr"

clean:
	rm -rf "$(WORKDIR)"
