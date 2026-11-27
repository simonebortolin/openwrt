# SPDX-License-Identifier: GPL-2.0-only

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/image.mk

# Realtek "Luna" U-Boot expects a legacy uImage: MIPS/Linux/kernel, LZMA
# compressed, load+entry at 0x80000000 (per vendor fwu.sh / on-board uImage).
# Bring-up is run-from-RAM: a uImage carrying an embedded initramfs is
# TFTP'd into RAM and bootm'd; no flash write during bring-up.
KERNEL_LOADADDR:=0x80000000

define Device/Default
  PROFILES := Default
  KERNEL_NAME := vmlinux
  # Legacy vendor U-Boot passes no DT, so append the board DTB to the kernel
  # (CONFIG_MIPS_RAW_APPENDED_DTB); head.S finds it and sets fw_passed_dtb.
  KERNEL_INITRAMFS := kernel-bin | append-dtb | lzma | uImage lzma
  KERNEL := kernel-bin | append-dtb | lzma | uImage lzma
  IMAGE_NAME = $$(IMAGE_PREFIX)-$$(2)
  IMAGES :=
endef

# A RAM-only diagnosis kernel (build-openwrt --variant memory-debug / slab-debug / perf) is
# never flashed, so it is held to no partition: the TFTP boot is its only route to a unit.
flash_kernel_size = $(if $(CONFIG_KERNEL_MEM_ALLOC_PROFILING)$(CONFIG_KERNEL_SLUB_DEBUG)$(CONFIG_KERNEL_PERF_EVENTS),,$(1))

# CIG bootloader cramfs factory image: wraps the initramfs uImage inside a
# big-endian cramfs named "UCIG" (the stock CIG firmware container format).
# The stock ZTE web UI accepts this cramfs, extracts the kernel, and flashes it.
define Build/cig-cramfs-factory
	rm -rf $@.cramfs-root
	mkdir -p $@.cramfs-root/bootimg
	cp $(KDIR)/tmp/$(KERNEL_INITRAMFS_IMAGE) $@.cramfs-root/uImage
	mkfs.cramfs -N big -n UCIG $@.cramfs-root $@
	rm -rf $@.cramfs-root
endef

include $(SUBTARGET).mk

$(eval $(call BuildImage))
