PLATFORMS = ssv6x5x

KBUILD_TOP := $(PWD)/hardware/wifi/icomm/drivers/ssv6xxx/ssv6x5x
include $(KBUILD_TOP)/$(PLATFORMS).cfg
include $(KBUILD_TOP)/platform-config.mak

PWD := $(shell pwd)

ifeq ($(KERNELRELEASE),)
# current directory is driver
CFGDIR = $(PWD)/../../../../config

-include $(CFGDIR)/build_config.cfg
-include $(CFGDIR)/rules.make

endif

ifeq ($(findstring -DSSV_SUPPORT_HAL, $(ccflags-y)), -DSSV_SUPPORT_HAL)
KMODULE_NAME=ssv6x5x
else
KMODULE_NAME=ssv6051
endif
EXTRA_CFLAGS := -I$(KBUILD_TOP) -I$(KBUILD_TOP)/include

DEF_PARSER_H = $(KBUILD_TOP)/include/ssv_conf_parser.h
$(shell env ccflags="$(ccflags-y)" $(KBUILD_TOP)/parser-conf.sh $(DEF_PARSER_H))

KERN_SRCS := ssvdevice/ssvdevice.c
KERN_SRCS += ssvdevice/ssv_cmd.c

KERN_SRCS += hci/ssv_hci.c

KERN_SRCS += smac/init.c
KERN_SRCS += smac/ssv_skb.c
KERN_SRCS += smac/dev.c
KERN_SRCS += smac/ssv_rc_minstrel.c
KERN_SRCS += smac/ssv_rc_minstrel_ht.c
KERN_SRCS += smac/ap.c
KERN_SRCS += smac/ampdu.c
KERN_SRCS += smac/efuse.c
KERN_SRCS += smac/ssv_pm.c
KERN_SRCS += smac/ssv_skb.c

ifeq ($(findstring -DCONFIG_SSV6XXX_DEBUGFS, $(ccflags-y)), -DCONFIG_SSV6XXX_DEBUGFS)
KERN_SRCS += smac/ssv6xxx_debugfs.c
endif

ifeq ($(findstring -DUSE_LOCAL_CRYPTO, $(ccflags-y)), -DUSE_LOCAL_CRYPTO)
KERN_SRCS += smac/sec_ccmp.c
KERN_SRCS += smac/sec_tkip.c
KERN_SRCS += smac/sec_wep.c
KERN_SRCS += smac/wapi_sms4.c
KERN_SRCS += smac/sec_wpi.c
endif


ifeq ($(findstring -DCONFIG_SMARTLINK, $(ccflags-y)), -DCONFIG_SMARTLINK)
KERN_SRCS += smac/ksmartlink.c
endif
ifeq ($(findstring -DCONFIG_SSV_SMARTLINK, $(ccflags-y)), -DCONFIG_SSV_SMARTLINK)
KERN_SRCS += smac/kssvsmart.c
endif

ifeq ($(findstring -DSSV_SUPPORT_HAL, $(ccflags-y)), -DSSV_SUPPORT_HAL)
KERN_SRCS += smac/hal/hal.c

ifeq ($(findstring -DSSV_SUPPORT_SSV6051, $(ccflags-y)), -DSSV_SUPPORT_SSV6051)
KERN_SRCS += smac/ssv_rc.c
KERN_SRCS += smac/ssv_ht_rc.c
KERN_SRCS += smac/hal/ssv6051/ssv6051_mac.c
KERN_SRCS += smac/hal/ssv6051/ssv6051_phy.c
KERN_SRCS += smac/hal/ssv6051/ssv6051_cabrioA.c
KERN_SRCS += smac/hal/ssv6051/ssv6051_cabrioE.c
endif

ifeq ($(findstring -DSSV_SUPPORT_SSV6006, $(ccflags-y)), -DSSV_SUPPORT_SSV6006)

KERN_SRCS += hwif/usb/usb.c
KERN_SRCS += smac/hal/ssv6006c/ssv6006_common.c
KERN_SRCS += smac/hal/ssv6006c/ssv6006C_mac.c
KERN_SRCS += smac/hal/ssv6006c/ssv6006_phy.c
KERN_SRCS += smac/hal/ssv6006c/ssv6006_turismoC.c
ifeq ($(findstring -DSSV_SUPPORT_SSV6006AB, $(ccflags-y)), -DSSV_SUPPORT_SSV6006AB)
KERN_SRCS += smac/hal/ssv6006/ssv6006_mac.c
KERN_SRCS += smac/hal/ssv6006/ssv6006_cabrioA.c
KERN_SRCS += smac/hal/ssv6006/ssv6006_geminiA.c
KERN_SRCS += smac/hal/ssv6006/ssv6006_turismoA.c
KERN_SRCS += smac/hal/ssv6006/ssv6006_turismoB.c
endif
endif
else
KERN_SRCS += smac/ssv_rc.c
KERN_SRCS += smac/ssv_ht_rc.c
endif

KERN_SRCS += hwif/sdio/sdio.c
#KERNEL_MODULES += crypto

ifeq ($(findstring -DCONFIG_SSV_SUPPORT_AES_ASM, $(ccflags-y)), -DCONFIG_SSV_SUPPORT_AES_ASM)
KERN_SRCS += crypto/aes_glue.c
KERN_SRCS += crypto/sha1_glue.c
KERN_SRCS_S := crypto/aes-armv4.S
KERN_SRCS_S += crypto/sha1-armv4-large.S
endif


#KERN_SRCS += $(PLATFORMS)-generic-wlan.c

$(KMODULE_NAME)-y += $(KERN_SRCS_S:.S=.o)
$(KMODULE_NAME)-y += $(KERN_SRCS:.c=.o)

obj-$(CONFIG_SSV6X5X) += $(KMODULE_NAME).o

all:module strip

module:
	make  -C $(SSV_KERNEL_PATH) ARCH=$(SSV_ARCH) CROSS_COMPILE=$(SSV_CROSS) \
	EXTRA_CFLAGS="$(EXTRA_CFLAGS)" M=$(PWD) modules

install:
	install -p -m 644 $(KMODULE_NAME).ko $(KMODDESTDIR)

uninstall:
	rm -f $(KMODDESTDIR)/$(KMODULE_NAME).ko
    
strip:
	#cp eagle.ko $(KO_NAME).ko
	#cp $(KO_NAME).ko $(DEFAULT_MODULES_DIR)
	#cp ssv6200.ko $(DEFAULT_MODULES_DIR)
	cp platforms/$(PLATFORMS)-wifi.cfg image/$(KMODULE_NAME)-wifi.cfg
	cp $(KMODULE_NAME).ko image/$(KMODULE_NAME).ko
	cp platforms/cli image
ifneq ($(SSV_STRIP),)
	cp $(KMODULE_NAME).ko image/$(KMODULE_NAME)_ori.ko
	$(SSV_STRIP) --strip-unneeded image/$(KMODULE_NAME).ko
	#$(SSV_STRIP) --strip-debug image/$(KMODULE_NAME).ko
endif

clean:
	make -C $(SSV_KERNEL_PATH) ARCH=$(SSV_ARCH) CROSS_COMPILE=$(SSV_CROSS) \
	M=$(PWD) clean

