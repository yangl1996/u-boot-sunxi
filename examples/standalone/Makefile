#
# (C) Copyright 2000-2006
# Wolfgang Denk, DENX Software Engineering, wd@denx.de.
#
# SPDX-License-Identifier:	GPL-2.0+
#

include $(TOPDIR)/config.mk

ELF-$(ARCH)  :=
ELF-$(BOARD) :=
ELF-$(CPU)   :=
ELF-y        := hello_world

ELF-$(CONFIG_SMC91111)           += smc91111_eeprom
ELF-$(CONFIG_SMC911X)            += smc911x_eeprom
ELF-$(CONFIG_SPI_FLASH_ATMEL)    += atmel_df_pow2
ELF-i386                         += 82559_eeprom
ELF-mpc5xxx                      += interrupt
ELF-mpc8xx                       += test_burst timer
ELF-mpc8260                      += mem_to_mem_idma2intr
ELF-ppc                          += sched
ELF-oxc                          += eepro100_eeprom

#
# Some versions of make do not handle trailing white spaces properly;
# leading to build failures. The problem was found with GNU Make 3.80.
# Using 'strip' as a workaround for the problem.
#
ELF := $(strip $(ELF-y) $(ELF-$(ARCH)) $(ELF-$(BOARD)) $(ELF-$(CPU)))

SREC := $(addsuffix .srec,$(ELF))
BIN  := $(addsuffix .bin,$(ELF))

COBJS	:= $(ELF:=.o)

LIB	= $(obj)libstubs.o

LIBAOBJS-$(ARCH)     :=
LIBAOBJS-$(CPU)      :=
LIBAOBJS-ppc         += $(ARCH)_longjmp.o $(ARCH)_setjmp.o
LIBAOBJS-mpc8xx      += test_burst_lib.o
LIBAOBJS := $(LIBAOBJS-$(ARCH)) $(LIBAOBJS-$(CPU))

LIBCOBJS = stubs.o

LIBOBJS	= $(addprefix $(obj),$(LIBAOBJS) $(LIBCOBJS))

SRCS	:= $(COBJS:.o=.c) $(LIBCOBJS:.o=.c) $(LIBAOBJS:.o=.S)
OBJS	:= $(addprefix $(obj),$(COBJS))
ELF	:= $(addprefix $(obj),$(ELF))
BIN	:= $(addprefix $(obj),$(BIN))
SREC	:= $(addprefix $(obj),$(SREC))

# Add GCC lib
ifdef USE_PRIVATE_LIBGCC
ifeq ("$(USE_PRIVATE_LIBGCC)", "yes")
PLATFORM_LIBGCC = $(OBJTREE)/arch/$(ARCH)/lib/libgcc.o
else
PLATFORM_LIBGCC = -L $(USE_PRIVATE_LIBGCC) -lgcc
endif
else
PLATFORM_LIBGCC := -L $(shell dirname `$(CC) $(CFLAGS) -print-libgcc-file-name`) -lgcc
endif
PLATFORM_LIBS += $(PLATFORM_LIBGCC)
export PLATFORM_LIBS

CPPFLAGS += -I..

# For PowerPC there's no need to compile standalone applications as a
# relocatable executable.  The relocation data is not needed, and
# also causes the entry point of the standalone application to be
# inconsistent.
ifeq ($(ARCH),powerpc)
AFLAGS := $(filter-out $(RELFLAGS),$(AFLAGS))
CFLAGS := $(filter-out $(RELFLAGS),$(CFLAGS))
CPPFLAGS := $(filter-out $(RELFLAGS),$(CPPFLAGS))
endif

# We don't want gcc reordering functions if possible.  This ensures that an
# application's entry point will be the first function in the application's
# source file.
CFLAGS_NTR := $(call cc-option,-fno-toplevel-reorder)
CFLAGS += $(CFLAGS_NTR)

all:	$(obj).depend $(OBJS) $(LIB) $(SREC) $(BIN) $(ELF)

#########################################################################
$(LIB):	$(obj).depend $(LIBOBJS)
	$(call cmd_link_o_target, $(LIBOBJS))

$(ELF):
$(obj)%:	$(obj)%.o $(LIB)
		$(LD) $(LDFLAGS) -g -Ttext $(CONFIG_STANDALONE_LOAD_ADDR) \
			-o $@ -e $(SYM_PREFIX)$(notdir $(<:.o=)) $< $(LIB) \
			$(PLATFORM_LIBS)

$(SREC):
$(obj)%.srec:	$(obj)%
		$(OBJCOPY) -O srec $< $@ 2>/dev/null

$(BIN):
$(obj)%.bin:	$(obj)%
		$(OBJCOPY) -O binary $< $@ 2>/dev/null

#########################################################################

# defines $(obj).depend target
include $(SRCTREE)/rules.mk

sinclude $(obj).depend

#########################################################################
