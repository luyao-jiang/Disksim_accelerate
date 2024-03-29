
# DiskSim Storage Subsystem Simulation Environment (Version 4.0)
# Revision Authors: John Bucy, Greg Ganger
# Contributors: John Griffin, Jiri Schindler, Steve Schlosser
#
# Copyright (c) of Carnegie Mellon University, 2001-2008.
#
# This software is being provided by the copyright holders under the
# following license. By obtaining, using and/or copying this software,
# you agree that you have read, understood, and will comply with the
# following terms and conditions:
#
# Permission to reproduce, use, and prepare derivative works of this
# software is granted provided the copyright and "No Warranty" statements
# are included with all reproductions and derivative works and associated
# documentation. This software may also be redistributed without charge
# provided that the copyright and "No Warranty" statements are included
# in all redistributions.
#
# NO WARRANTY. THIS SOFTWARE IS FURNISHED ON AN "AS IS" BASIS.
# CARNEGIE MELLON UNIVERSITY MAKES NO WARRANTIES OF ANY KIND, EITHER
# EXPRESSED OR IMPLIED AS TO THE MATTER INCLUDING, BUT NOT LIMITED
# TO: WARRANTY OF FITNESS FOR PURPOSE OR MERCHANTABILITY, EXCLUSIVITY
# OF RESULTS OR RESULTS OBTAINED FROM USE OF THIS SOFTWARE. CARNEGIE
# MELLON UNIVERSITY DOES NOT MAKE ANY WARRANTY OF ANY KIND WITH RESPECT
# TO FREEDOM FROM PATENT, TRADEMARK, OR COPYRIGHT INFRINGEMENT.
# COPYRIGHT HOLDERS WILL BEAR NO LIABILITY FOR ANY USE OF THIS SOFTWARE
# OR DOCUMENTATION.

BISON = bison
FLEX = flex


include .paths

# setup a PKG_CONFIG_PATH in order to compile with opencv
#PKG_CONFIG_PATH=/uf21/lj5bp/Documents/research/opencv/lib/pkgconfig:${PKG_CONFIG_PATH}
#export PKG_CONFIG_PATH


LDFLAGS = -pg -lm -lz -L. -ldisksim $(DISKMODEL_LDFLAGS) $(MEMSMODEL_LDFLAGS) \
                            $(SSDMODEL_LDFLAGS) \
                            $(LIBPARAM_LDFLAGS) $(LIBDDBG_LDFLAGS) `pkg-config --libs opencv`

HP_FAST_OFLAGS = +O4
NCR_FAST_OFLAGS = -O4 -Hoff=BEHAVED 
FREEBLOCKS_OFLAGS =
DEBUG_OFLAGS = -gdwarf-2 -g3 -DASSERTS  #-DDEBUG=1
PROF_OFLAGS = -g -DASSERTS -p
GPROF_OFLAGS = -g -DASSERTS -pg
FBSYSSIM_OFLAGS = -O6 -fomit-frame-pointer -fexpensive-optimizations -fschedule-insns2


ifeq ($(DEBUG),1)
	CFLAGS = -pg -I. $(DISKMODEL_CFLAGS) $(LIBPARAM_CFLAGS) $(LIBDDBG_CFLAGS) $(DEBUG_OFLAGS) $(FREEBLOCKS_OFLAGS) $(MEMSMODEL_CFLAGS) $(SSDMODEL_CFLAGS) `pkg-config --cflags opencv` -I../include/ -D_INLINE -DDEBUG
else
	CFLAGS = -I. -O3 -Wno-array-bounds $(DISKMODEL_CFLAGS) $(LIBPARAM_CFLAGS) $(LIBDDBG_CFLAGS) $(FREEBLOCKS_OFLAGS) $(MEMSMODEL_CFLAGS) $(SSDMODEL_CFLAGS) `pkg-config --cflags opencv` -I../include/ -D_INLINE 
#	CFLAGS = -pg -I. $(DISKMODEL_CFLAGS) $(LIBPARAM_CFLAGS) $(LIBDDBG_CFLAGS) $(DEBUG_OFLAGS) $(FREEBLOCKS_OFLAGS) $(MEMSMODEL_CFLAGS) $(SSDMODEL_CFLAGS) `pkg-config --cflags opencv` -I../include/ -D_INLINE -DDEBUG
endif



#CC = gcc -O -Wall -Wno-unused -Wimplicit -Werror -MD
CC = gcc -Wall -Wno-unused -Wimplicit -Werror -MD 
# because purify spits out warnings on stdout...
CC-DEP = gcc $(LIBPARAM_CFLAGS) $(DISKMODEL_CFLAGS)

# The following lines create a dependency target based on the state of 
# the modules files. If the .c and .h files are not created, the dependency
# target is modules which invokes "make -C modules". If the files are already
# created, the target are the files themselves. 
# This expression is to avoid remaaking of the libdisksim.a with ar and ranlib
# even if no files have changed. 
MODULEDEPS = $(wildcard modules/*.h modules/*.c)
ifeq ($(MODULEDEPS),)
MODULEDEPS = modules
endif


ifeq ($(DEBUG),1)
  all: disksim.debug rms hplcomb syssim
else
  all: disksim rms hplcomb syssim
endif 

ifeq ($(DEBUG),1)
clean:
	rm -f TAGS *.o disksim.debug syssim rms hplcomb core libdisksim.a
	$(MAKE) -C modules clean
else
clean:
	rm -f TAGS *.o disksim syssim rms hplcomb core libdisksim.a
	$(MAKE) -C modules clean
endif
realclean: clean
	rm -f *.d .depend
	$(MAKE) -C modules clean

distclean: realclean
	rm -f *~ *.a
	rm -rf ../lib ../include
	$(MAKE) -C modules distclean

.PHONY: modules

modules: 
	$(MAKE) -C modules
	mkdir -p ../include/disksim/modules
	cp -pR modules/*.h ../include/disksim/modules
	cp disksim_interface.h ../include

modules/*.h modules/*.c:
	$(MAKE) -C modules

#include .depend

DISKSIM_SRC = disksim.c disksim_intr.c disksim_pfsim.c \
	disksim_pfdisp.c disksim_synthio.c disksim_iotrace.c disksim_iosim.c \
	disksim_logorg.c disksim_redun.c disksim_ioqueue.c disksim_iodriver.c \
	disksim_bus.c disksim_controller.c disksim_ctlrdumb.c \
	disksim_ctlrsmart.c disksim_disk.c disksim_diskctlr.c \
	disksim_diskcache.c \
	disksim_statload.c disksim_stat.c disksim_rand48.c disksim_malloc.c \
	disksim_cache.c disksim_cachemem.c disksim_cachedev.c \
	disksim_simpledisk.c disksim_device.c \
	disksim_loadparams.c \
	raw_layout.c

DISKSIM_OBJ = $(DISKSIM_SRC:.c=.o) 

$(DISKSIM_OBJ): %.o: %.c
	$(CC) -c $(CFLAGS) $< -o $@


# production rule for making disksim with freeblocks support
# FREEBLOCKS_OFLAGS = -DFREEBLOCKS
fbdisksim :
	$(MAKE) FREEBLOCKS_OFLAGS=-DFREEBLOCKS all

rms : rms.c
	$(CC) $< -lm -o $@

hplcomb : hplcomb.c
	$(CC) $< -o $@

libdisksim.a: $(MODULEDEPS) $(DISKSIM_OBJ) disksim_interface.o
	ar cru $@ $(DISKSIM_OBJ) disksim_interface.o modules/*.o
	ranlib $@
	mkdir -p ../lib
	cp libdisksim.a ../lib

.PHONY : disksim syssim
disksim : $(MODULEDEPS) libdisksim.a disksim_main.o 
	$(CC) $(CFLAGS) -o $@ disksim_main.o $(LDFLAGS)

disksim.debug : $(MODULEDEPS) libdisksim.a disksim_main.o 
	$(CC) $(CFLAGS) -o $@ disksim_main.o $(LDFLAGS)

syssim: syssim_driver.o libdisksim.a
	$(CC) $(CFLAGS) -o $@ syssim_driver.o disksim_interface.o $(LDFLAGS)

########################################################################

# rule to automatically generate dependencies from source files
#%.d: %.c
#	set -e; $(CC-DEP) -M $(CPPFLAGS) $<  \
#		| sed 's/\($*\)\.o[ :]*/\1.o $@ : /g' > $@; \
#		[ -s $@ ] 2>/dev/null >/dev/null || rm -f $@


# this is a little less aggressive and annoying than the above
depend: .depend 

.depend: *.c *.h
	$(MAKE) -C modules
	rm -f .depend
	$(foreach file, $(DISKSIM_SRC), \
		$(CC-DEP) $(CFLAGS) -M $(file) >> .depend; )

