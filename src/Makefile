CURRENT	:= $(shell uname -r)
MDIR	:= drivers/usb/usbip 
KDIR	:= /lib/modules/$(CURRENT)/build
EXTRA_CFLAGS := -DEXPORT_SYMTAB
PWD := $(shell pwd)
DEST := /lib/modules/$(CURRENT)/kernel/$(MDIR)

usbip-core-y := usbip_common.o usbip_event.o
vhci-hcd-y := vhci_sysfs.o vhci_tx.o vhci_rx.o vhci_hcd.o
usbip-host-y := stub_dev.o stub_main.o stub_rx.o stub_tx.o
usbip-vudc-y := vudc_dev.o vudc_sysfs.o vudc_tx.o vudc_rx.o vudc_transfer.o vudc_main.o


obj-m := $(foreach m,$(module_list),$(m).o)
install_list := $(foreach m,$(module_list),install_$(m))

usbip-host:
	make build_modules module_list=usbip-host

all:
	make build_modules module_list="usbip-host usbip-core vhci-hcd usbip-vudc"

install:
	make install_modules module_list=usbip-host

install_all:	
	make install_modules module_list="usbip-host usbip-core vhci-hcd usbip-vudc"

build_modules:
	make -C $(KDIR) M=$(PWD) modules

ifneq ($(install_list),)

$(install_list):
	cp $(@:install_%=%).ko $(DEST)

install_modules: $(install_list)
	/sbin/depmod -a

endif

clean:
	$(MAKE) -C $(KDIR) M=$(PWD) clean
	@rm -f *.mod

.PHONY: modules default install clean $(install_list)

-include $(KDIR)/Rules.make
