#
# Copyright (c) 2014 Citrix Systems, Inc.
#

BLKTAP_ROOT= ..
include $(BLKTAP_ROOT)/Rules.mk

LIBVHDDIR  = $(BLKTAP_ROOT)/vhd/lib

IBIN       = tapdisk2 td-util tapdisk-stream tapdisk-diff
LOCK_UTIL  = lock-util
INST_DIR   = /usr/sbin

CFLAGS    += -Wall
CFLAGS    += -g
CFLAGS    += -Wno-unused
CFLAGS    += -fno-strict-aliasing
CFLAGS    += -I../include
CFLAGS    += -D_GNU_SOURCE
CFLAGS    += -DUSE_NFS_LOCKS
CFLAGS    += -DTAPDISK

LIBS      += -lrt

# Get gcc to generate the dependencies for us.
CFLAGS    += -Wp,-MD,.$(@F).d
DEPS       = .*.d

tapdisk2 td-util tapdisk-stream tapdisk-diff: LIBS += -L$(LIBVHDDIR) -lvhd -luuid -licbinn_resolved -lpthread -ldl

ifneq ($(USE_SYSTEM_LIBRARIES),y)
LIBAIO_DIR = $(XEN_ROOT)/tools/libaio/src
tapdisk2 tapdisk-stream tapdisk-diff: AIOLIBS := $(LIBAIO_DIR)/libaio.a
tapdisk2 tapdisk-stream tapdisk-diff: CFLAGS  += -I$(LIBAIO_DIR) -I$(XEN_LIBXC)
tapdisk2 tapdisk-client tapdisk-stream tapdisk-diff block-log: CFLAGS += -I$(XEN_ROOT)/tools/include
else
tapdisk2 tapdisk-stream tapdisk-diff: AIOLIBS := -laio
endif

ifeq ($(VHD_STATIC),y)
td-util: CFLAGS += -static
endif

tapdisk tapdisk2 tapdisk-stream tapdisk-diff: LIBS += -lcrypto -Lcrypto -lxts-aes
tapdisk tapdisk2 tapdisk-stream tapdisk-diff: crypto/libxts-aes.a

crypto/libxts-aes.a: crypto/*.c crypto/*.h
	$(MAKE) -C crypto libxts-aes.a

TAP-OBJS  := scheduler.o
TAP-OBJS  += tapdisk-control.o
TAP-OBJS  += tapdisk-vbd.o
TAP-OBJS  += tapdisk-image.o
TAP-OBJS  += tapdisk-driver.o
TAP-OBJS  += tapdisk-disktype.o
TAP-OBJS  += tapdisk-interface.o
TAP-OBJS  += tapdisk-server.o
TAP-OBJS  += tapdisk-queue.o
TAP-OBJS  += tapdisk-filter.o
TAP-OBJS  += tapdisk-logfile.o
TAP-OBJS  += tapdisk-log.o
TAP-OBJS  += tapdisk-utils.o
TAP-OBJS  += tapdisk-syslog.o
TAP-OBJS  += tapdisk-stats.o
TAP-OBJS  += tapdisk-storage.o
TAP-OBJS  += io-optimize.o
TAP-OBJS  += lock.o

MISC-OBJS := atomicio.o

BLK-OBJS  := block-aio.o
BLK-OBJS  += block-ram.o
BLK-OBJS  += block-cache.o
BLK-OBJS  += block-vhd.o
BLK-OBJS  += block-vindex.o
BLK-OBJS  += block-lcache.o
BLK-OBJS  += block-crypto.o

all: $(IBIN) lock-util

TAPDISK-OBJS := $(TAP-OBJS) $(BLK-OBJS)

tapdisk2: tapdisk2.c $(TAPDISK-OBJS) $(MISC-OBJS)
	$(CC) $(CFLAGS) -o $@ $(TAPDISK-OBJS) $(MISC-OBJS) $< $(LIBS) $(AIOLIBS)

tapdisk-client: tapdisk-client.o
	$(CC) $(CFLAGS) -o $@ $^ $(LIBS)

tapdisk-stream tapdisk-diff: %: %.o $(TAPDISK-OBJS)
	$(CC) $(CFLAGS) -o $@ $< $(TAPDISK-OBJS) $(LIBS) $(AIOLIBS)

td-util: td.o tapdisk-utils.o
	$(CC) $(CFLAGS) -o $@ $^ $(LIBS)

lock-util: lock.c
	$(CC) $(CFLAGS) -DUTIL -o lock-util lock.c $(LIBS)

install: all
	$(INSTALL_DIR) -p $(DESTDIR)$(INST_DIR)
	$(INSTALL_PROG) $(IBIN) $(LOCK_UTIL) $(DESTDIR)$(INST_DIR)

clean:
	rm -rf *.o *~ $(DEPS) xen TAGS $(IBIN) $(LIB) $(LOCK_UTIL)
	$(MAKE) -C crypto clean

.PHONY: clean install

-include $(DEPS)
