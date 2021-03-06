include $(TOPDIR)/env.mk
include $(TOPDIR)/rules.mk

EFI_ARCH = $(shell $(CC) -dumpmachine | cut -f1 -d- | sed s,i[3456789]86,ia32,)

# XXX: __attribute__((ms_abi)) is only available since gcc 4.7
CFLAGS += \
	  -fno-merge-constants -fno-strict-aliasing \
	  -ffreestanding -fshort-wchar \
	  -fno-stack-protector -fno-stack-check \
	  -fpic -fno-builtin -nostdinc \
	  $(addprefix -I, $(TOPDIR)/src/include $(gnuefi_includedir) \
	  $(shell $(CC) -print-file-name=include)) \
	  -DCONFIG_$(EFI_ARCH) #-DGNU_EFI_USE_MS_ABI \
	  -DETU_VERSION=L\"$(ETU_VERSION)\"

ifeq ($(EFI_ARCH),x86_64)
    DEFAULT_ARCH = x64
    CFLAGS += -m64 -mno-red-zone -mno-mmx -maccumulate-outgoing-args -mno-sse -DDEFAULT_ARCH=\"$(DEFAULT_ARCH)\"
else ifeq ($(EFI_ARCH),ia32)
    DEFAULT_ARCH = ia32
    CFLAGS += -m32 -mno-mmx -mno-sse -maccumulate-outgoing-args -DDEFAULT_ARCH=\"$(DEFAULT_ARCH)\"
else ifeq ($(EFI_ARCH),aarch64)
    DEFAULT_ARCH = aarch64
    CFLAGS += -DDEFAULT_ARCH=\"$(DEFAULT_ARCH)\"
else
    $(error Unsupported EFI_ARCH $(EFI_ARCH) specified)
endif

LDFLAGS += -nostdlib -shared -Bsymbolic -L$(gnuefi_libdir) \
	   -T $(gnuefi_libdir)/elf_$(EFI_ARCH)_efi.lds $(gnuefi_libdir)/crt0-efi-$(EFI_ARCH).o \
	   $(EXTRA_LDFLAGS)

LIB_NAME := libefitcg2

LIB_TARGET := $(LIB_NAME).a

OBJS_$(LIB_NAME) := \
	build_info.o \
	tcg2.o \
	etet.o

TARGETS = \
	fake-etet.efi \
	dump-etet.efi \
	tpm2-efi-info.efi \
	tpm2-capability.efi \
	tpm2-eventlog.efi

FAKE_ETET_OBJS = fake-etet.o
DUMP_ETET_OBJS = dump-etet.o
TPM2_EFI_INFO_OBJS = tpm2-efi-info.o
TPM2_CAPABILITY_OBJS = tpm2-capability.o
TPM2_EVENTLOG_OBJS = tpm2-eventlog.o

.PHONE: all clean install

all: $(TARGETS) Makefile
	@echo "EFI_ARCH: $(EFI_ARCH)"

clean:
	$(RM) $(TARGETS) *.so *.debug $(FAKE_ETET_OBJS) \
		$(DUMP_ETET_OBJS) $(TPM2_EFI_INFO_OBJS) \
		$(TPM2_CAPABILITY_OBJS) $(TPM2_EVENTLOG_OBJS) \
		build_info.c
	$(RM) $(LIB_TARGET) $(OBJS_$(LIB_NAME)) \
		$(addsuffix .*, $(LIB_TARGET))

install: fake-etet.efi tpm2-capability.efi
	cp -f $^ $(EFI_DESTDIR)

$(LIB_NAME).a: $(OBJS_$(LIB_NAME))
	$(AR) rcs $@ $^

fake-etet.so: $(FAKE_ETET_OBJS) $(LIB_TARGET)
dump-etet.so: $(DUMP_ETET_OBJS) $(LIB_TARGET)
tpm2-efi-info.so: $(TPM2_EFI_INFO_OBJS) $(LIB_TARGET)
tpm2-capability.so: $(TPM2_CAPABILITY_OBJS) $(LIB_TARGET)
tpm2-eventlog.so: $(TPM2_EVENTLOG_OBJS) $(LIB_TARGET)

build_info.c: build_info.c.in
	sed -e "s~@@GIT_COMMIT@@~$(shell if [ -d $(TOPDIR)/.git ]; then git log -1 --pretty=format:%H | tr -d '\n'; elif [ -f $(TOPDIR)/commit ]; then cat $(TOPDIR)/commit | tr -d '\n'; else echo -n ???????; fi)~" \
	    -e "s~@@BUILD_MACHINE@@~$(shell bash -c 'whoami | tr -d "\n"; echo -n @; uname=`uname -a`; echo -n $${uname//\~/_} | tr -d "\n"')~" < $^ > $@
