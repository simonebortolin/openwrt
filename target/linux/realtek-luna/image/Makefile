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
# The CIG U-Boot validates a CRC-32/BZIP2 (poly 0x04C11DB7) stored as the
# last 4 bytes (big-endian) of the image; without it the web UI rejects the
# file. Algorithm from hack-gpon/hack-gpon-private cig_f6005_unlock/crc.c.
define Build/cig-cramfs-factory
	rm -rf $@.cramfs-root
	mkdir -p $@.cramfs-root/bootimg
	cp $(KDIR)/tmp/$(KERNEL_INITRAMFS_IMAGE) $@.cramfs-root/uImage
	mkfs.cramfs -N big -n UCIG $@.cramfs-root $@
	rm -rf $@.cramfs-root
	printf '%s\n' \
		'import sys, struct' \
		'd = open(sys.argv[1], "rb").read()' \
		'c = 0xFFFFFFFF' \
		'for b in d:' \
		'    c ^= b << 24' \
		'    for _ in range(8):' \
		'        c = ((c << 1) ^ 0x04C11DB7 if c & 0x80000000 else c << 1) & 0xFFFFFFFF' \
		'open(sys.argv[1], "ab").write(struct.pack(">I", c ^ 0xFFFFFFFF))' \
		| python3 - $@
endef

include $(SUBTARGET).mk

$(eval $(call BuildImage))
