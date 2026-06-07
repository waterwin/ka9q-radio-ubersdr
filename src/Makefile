# -*- makefile -*-

# Primary Linux makefile for ka9q-radio package
# Copyright 2017-2025, Phil Karn, KA9Q
# By default, builds release version
# To build for debugging say "make BUILD=debug"

BUILD ?= release

ifeq ($(BUILD),debug)
     DOPTS = -g
else
     DOPTS = -DNDEBUG=1 -O3
endif

ifdef SANITIZE
     DOPTS += -fsanitize=address -fsanitize=undefined
     LDOPTS = -fsanitize=address -fsanitize=undefined
endif

# Use -march=native only on x86; on aarch64 use a safe ARMv8 baseline so that
# cross-compiled Docker images (built under QEMU) don't emit instructions that
# are illegal on the target hardware (e.g. Pi 5 Cortex-A76 vs QEMU virtual CPU).
# Native aarch64 builds (e.g. compiling directly on a Pi) still benefit from -O3.
UNAME_M := $(shell uname -m)
ifeq ($(UNAME_M),aarch64)
    ARCHOPTS = -march=armv8-a+crc
else
    ARCHOPTS = -march=native
endif
# do NOT set -ffast-math or -ffinite-math-only; NANs are widely used as 'variable not set' sentinels
COPTS = -std=gnu11 -pthread -Wall -funsafe-math-optimizations -fno-math-errno -fcx-limited-range -freciprocal-math -fno-trapping-math -D_GNU_SOURCE=1 -Wextra -MMD -MP
COPTS += -fPIC
CFLAGS = $(DOPTS) $(ARCHOPTS) $(COPTS) $(INCLUDES)
BINDIR = /usr/local/bin
SODIR = /usr/local/lib/ka9q-radio
DAEMONDIR = /usr/local/sbin
DAEMONS = aprs aprsfeed cwd packetd radiod
EXECS = control fft-gen jt-decoded metadump monitor pcmrecord pl powers tune wd-record
LIBRADIO = ax25.o morse.o bandplan.o dump.o modes.o avahi.o avahi_browse.o attr.o filter.o iir.o decode_status.o status.o misc.o multicast.o rtp.o osc.o config.o
DYNAMIC_DRIVERS = rx888.so sig_gen.so cwsl_websdr.so
RADIOD_OBJECTS = main.o audio.o fm.o wfm.o linear.o spectrum.o radio.o radio_status.o rtcp.o fcd.o hid-libusb.o libradio.a
CFILES = aprs.c aprsfeed.c attr.c audio.c avahi.c avahi_browse.c ax25.c bandplan.c config.c control.c cwsl_websdr.c cwd.c decimate.c decode_status.c dump.c ezusb.c fcd.c filter.c fm.c hid-libusb.c iir.c jt-decoded.c linear.c main.c metadump.c misc.c modes.c monitor.c monitor-data.c monitor-display.c monitor-repeater.c morse.c multicast.c opusd.c opussend.c osc.c packetd.c pcmcat.c pcmrecord.c pcmsend.c pcmspawn.c pl.c powers.c radio.c radio_status.c rdsd.c rtcp.c rtp.c rx888.c setfilt.c show-pkt.c show-sig.c sig_gen.c spectrum.c status.c stereod.c tune.c wd-record.c wfm.c
HFILES = attr.h ax25.h bandplan.h conf.h config.h decimate.h ezusb.h fcd.h fcdhidcmd.h filter.h hidapi.h iir.h misc.h monitor.h morse.h multicast.h osc.h radio.h rx888.h status.h

# Build FOBOS support if installed, or if 'make FOBOS=1'
ifeq ($(wildcard /usr/local/include/fobos.h /usr/include/fobos.h),)
else
   FOBOS=1
endif
# if FOBOS is set, build it too
ifdef FOBOS
   CFILES += fobos.c
   DYNAMIC_DRIVERS += fobos.so
endif

# Build on raspberry pi only
ifeq ($(wildcard /usr/include/pigpio.h),)
else
   EXECS += set_xcvr
endif

all: $(DAEMONS) $(EXECS) $(DYNAMIC_DRIVERS)

install: $(DAEMONS) $(EXECS) $(DYNAMIC_DRIVERS)
	mkdir -p $(SODIR)
	rsync -a $(DYNAMIC_DRIVERS) $(SODIR)
	ln -sf $(SODIR)/*.so /usr/local/lib
	rsync -a $(DAEMONS) $(DAEMONDIR)
	rsync -a $(EXECS) $(BINDIR)
	setcap cap_sys_nice=ep $(BINDIR)/monitor
	setcap cap_net_admin,cap_sys_nice=ep $(DAEMONDIR)/radiod

clean:
	rm -f *.o *.a *.d *.so $(DAEMONS) $(EXECS) $(DYNAMIC_DRIVERS)

DEPS = $(CFILES:.c=.d) $(OBJS:.o=.d)

-include $(DEPS)

.PHONY: clean all install doc dynamics

# keep all .o files
.SECONDARY:

dynamics: $(DYNAMIC_DRIVERS)

# Executables

fft-gen: fft-gen.o
	$(CC) $(LDOPTS) -o $@ $^  -lfftw3f_threads -lfftw3f  -lm -lpthread

aprs: aprs.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lbsd -lm -lpthread

aprsfeed: aprsfeed.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lbsd -lm -lpthread

control: control.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -liniparser -lncursesw -lbsd -lm -lpthread

cwd: cwd.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lbsd -lm -lpthread

tune: tune.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lbsd -lm -lpthread

setfilt: setfilt.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lbsd -lm -lpthread

show-pkt: show-pkt.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lncursesw -lbsd -lm -lpthread

show-sig: show-sig.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lncursesw -lbsd -lm -lpthread

metadump: metadump.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lbsd -lm -lpthread

monitor: monitor.o monitor-data.o monitor-display.o monitor-repeater.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ `pkg-config --libs ncursesw` -lopus -lportaudio -lasound -lsamplerate -liniparser -lbsd -lm -lpthread

opusd: opusd.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lopus -lbsd -lm -lpthread

opussend: opussend.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lopus -lportaudio -lbsd -lm -lpthread

packetd: packetd.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lfftw3f_threads -lfftw3f -lbsd -lm -lpthread

pcmcat: pcmcat.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lbsd -lm -lpthread

pcmspawn: pcmspawn.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lbsd -lm -lpthread

pcmrecord: pcmrecord.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lopus -logg -lbsd -lm -lpthread

pcmsend: pcmsend.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lportaudio -lbsd -lm -lpthread

powers: powers.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lbsd -lm -lpthread

pl: pl.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lfftw3f_threads -lfftw3f -lbsd -lm -lpthread

radiod: $(RADIOD_OBJECTS)
	$(CC) -rdynamic $(LDOPTS) -o $@ $^  -lopus -lfftw3f_threads -lfftw3f -liniparser -lusb-1.0 -ldl -lbsd -lm -lpthread

rdsd: rdsd.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lfftw3f_threads -lfftw3f -lbsd -lm -lpthread

stereod: stereod.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lfftw3f_threads -lfftw3f -lbsd -lm -lpthread

jt-decoded: jt-decoded.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lbsd -lm -lpthread

wd-record: wd-record.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lbsd -lm -lpthread

# not made unless we're on a rpi
set_xcvr: set_xcvr.o libradio.a
	$(CC) $(LDOPTS) -o $@ $^ -lpigpio -liniparser

# Dynamically loaded modules for SDR front ends
rx888.so: rx888.o ezusb.o hid-libusb.o
	$(CC) -shared -o $@ $^ -lusb-1.0

fobos.so: fobos.o
	$(CC) -shared -o $@ $^ -lfobos

cwsl_websdr.so: cwsl_websdr.o
	$(CC) -shared -o $@ $^

# subroutines useful in more than one program
libradio.a: $(LIBRADIO)
	ar rv $@ $?
	ranlib $@

%.so: %.o
	$(CC) -shared -o $@ $<

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@