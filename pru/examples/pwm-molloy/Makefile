# 
# Copyright (c) 2016 Zubeen Tolani <ZeekHuge - zeekhuge@gmail.com>
# Makefile to make PRU_inlineASM_blinky project
#
# The make file is tried to be made as generic as possible
# So that it can be used to compile other programs too
# 
# Usage:
#	name your main source file as main_pru1.c or main_pru0.c
# 	add targets to variable TARGETS
#	add other files required while linking in variable LINK_PRU1(0)_FW
#	add compile targets, as added to LINK_PRU1(0)_FW for other files.
# 

# PRU_CGT environment variable must point to the TI PRU compiler directory.
# PRU_SUPPROT points to pru-software-support-package
# Both are set in setup.sh

LINKER_COMMAND_FILE=./AM335x_PRU.cmd
LIBS=--library=$(PRU_SUPPORT)/lib/rpmsg_lib.lib
INCLUDE=--include_path=$(PRU_SUPPORT)/include --include_path=$(PRU_SUPPORT)/include/am335x
STACK_SIZE=0x100
HEAP_SIZE=0x100

CFLAGS=-v3 -O2 --display_error_number --endian=little --hardware_mac=on --obj_directory=$(GEN_DIR) --pp_directory=$(GEN_DIR) -ppd -ppa --asm_listing --c_src_interlist # --absolute_listing
LFLAGS=--reread_libs --warn_sections --stack_size=$(STACK_SIZE) --heap_size=$(HEAP_SIZE) -m $(GEN_DIR)/file.map

GEN_DIR=gen

# PRU1_FW		=$(GEN_DIR)/main_pru1_fw.out
PRU0_FW		=$(GEN_DIR)/main_pru0_fw.out

# -----------------------------------------------------
# Variable to edit in the makefile

# add the required firmwares to TARGETS
# TARGETS		=$(PRU1_FW) $(PRU0_FW)
TARGETS		=$(PRU0_FW)

# add the required files while linkin the FW code
# required linking files for PRU0
LINK_PRU0_FW= $(GEN_DIR)/pru0-pwm1.object
# required linking files for PRU1
LINK_PRU1_FW= 

#------------------------------------------------------

.PHONY: all
all: $(TARGETS) pwm-test
	@echo '-	Generated firmwares are : $^'


$(PRU0_FW): $(GEN_DIR)/main_pru0.object $(LINK_PRU0_FW)
	@echo 'LD	$^' 
	@$(PRU_CGT)/bin/lnkpru -i$(PRU_CGT)/lib -i$(PRU_CGT)/include $(LFLAGS) -o $@ $^  $(LINKER_COMMAND_FILE) --library=libc.a $(LIBS) $^

$(PRU1_FW): $(GEN_DIR)/main_pru1.object $(LINK_PRU1_FW)
	@echo 'LD	$^'
	@$(PRU_CGT)/bin/lnkpru -i$(PRU_CGT)/lib -i$(PRU_CGT)/include $(LFLAGS) -o $@ $^  $(LINKER_COMMAND_FILE) --library=libc.a $(LIBS) $^

$(GEN_DIR)/main_pru0.object: main_pru0.c 
	@mkdir -p $(GEN_DIR)
	@echo 'CC	$<'
	@$(PRU_CGT)/bin/clpru --include_path=$(PRU_CGT)/include $(INCLUDE) $(CFLAGS) -fe $@ $<

$(GEN_DIR)/main_pru1.object: main_pru1.c
	@mkdir -p $(GEN_DIR)
	@echo 'CC	$<'
	@$(PRU_CGT)/bin/clpru --include_path=$(PRU_CGT)/include $(INCLUDE) $(CFLAGS) -fe $@ $<


$(GEN_DIR)/pru0-pwm1.object: pru0-pwm1.asm
	@mkdir -p $(GEN_DIR)
	@echo 'CC	$<'
	@$(PRU_CGT)/bin/clpru --include_path=$(PRU_CGT)/include $(INCLUDE) $(CFLAGS) -fe $@ $<

$(GEN_DIR)/pru1-test.object: pru1-test.asm
	@mkdir -p $(GEN_DIR)
	@echo 'CC	$<'
	@$(PRU_CGT)/bin/clpru --include_path=$(PRU_CGT)/include $(INCLUDE) $(CFLAGS) -fe $@ $<

pwm-test:	pwm-test.c
	gcc $< -o $@

.PHONY: install install-pru1 install-pru0 copy_pru0_fw copy_pru1_fw reboot_pru_1 reboot_pru_0

install: $(patsubst $(GEN_DIR)/main_pru%_fw.out, install-pru%, $(TARGETS))

install-pru1: $(PRU1_FW) copy_pru1_fw reboot_pru1
install-pru0: $(PRU0_FW) copy_pru0_fw reboot_pru0

copy_pru1_fw:
	@echo '-	copying firmware to /lib/firmware/am335x_pru1_fw'
	@cp $(PRU1_FW) /lib/firmware/am335x-pru1-fw

copy_pru0_fw: $(PRU0_FW)
	@echo '-	copying firmware to /lib/firmware/am335x_pru0_fw'
	@cp $(PRU0_FW) /lib/firmware/am335x-pru0-fw

reboot_pru1:
	@echo '-	rebooting pru core 1'
	$(shell echo "4a338000.pru1" > /sys/bus/platform/drivers/pru-rproc/unbind 2> /dev/null)
	$(shell echo "4a338000.pru1" > /sys/bus/platform/drivers/pru-rproc/bind)
	@echo "-	pru core 1 is now loaded with $(PRU1_FW)"

reboot_pru0:
	@echo '-	rebooting pru core 0'
	$(shell echo "4a334000.pru0" > /sys/bus/platform/drivers/pru-rproc/unbind 2> /dev/null)
	$(shell echo "4a334000.pru0" > /sys/bus/platform/drivers/pru-rproc/bind)
	@echo "-	pru core 0 is now loaded with $(PRU0_FW)"


.PHONY: clean
clean:
	@echo 'CLEAN	.'
	@rm -rf $(GEN_DIR) pwm-test main_pru0.asm
