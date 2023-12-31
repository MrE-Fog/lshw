PACKAGENAME:=lshw
export PACKAGENAME
VERSION?= $(shell git describe --tags --long | cut -d - -f 1,2 | tr - .)

SQLITE?=0
ZLIB?=0

DESTDIR?=/
PREFIX?=/usr
SBINDIR=$(PREFIX)/sbin
MANDIR=$(PREFIX)/share/man
DATADIR=$(PREFIX)/share
INSTALL?=install -p
STRIP?=strip
GZIP?=gzip -9
export DESTDIR
export PREFIX
export SBINDIR
export MANDIR
export DATADIR
export SQLITE
export ZLIB

CXX?=$(CROSS_COMPILE)c++
PKG_CONFIG ?= pkg-config
INCLUDES=-I./core/
DEFINES=-DPREFIX=\"$(PREFIX)\" -DSBINDIR=\"$(SBINDIR)\" -DMANDIR=\"$(MANDIR)\" -DDATADIR=\"$(DATADIR)\" -DVERSION=\"$(VERSION)\"
CXXFLAGS=-g -Wall -g $(INCLUDES) $(DEFINES) $(RPM_OPT_FLAGS)
ifeq ($(SQLITE), 1)
	CXXFLAGS+= -DSQLITE $(shell $(PKG_CONFIG) --cflags sqlite3)
endif
ifeq ($(ZLIB), 1)
	CXXFLAGS+= -DZLIB $(shell $(PKG_CONFIG) --cflags zlib)
endif
LDFLAGS+=-L./core/ -g
ifneq ($(shell $(LD) --help 2| grep -- --as-needed), )
	LDFLAGS+= -Wl,--as-needed
endif
LDSTATIC=-static
LIBS+=-llshw -lresolv
ifeq ($(SQLITE), 1)
	LIBS+= $(shell $(PKG_CONFIG) --libs sqlite3)
endif
ifeq ($(ZLIB), 1)
	LIBS+= $(shell $(PKG_CONFIG) --libs zlib)
endif

export CXXFLAGS
export LIBS
export LDFLAGS

ifeq ($(ZLIB), 1)
DATAFILES = pci.ids.gz usb.ids.gz oui.txt.gz manuf.txt.gz pnp.ids.gz pnpid.txt.gz
else
DATAFILES = pci.ids usb.ids oui.txt manuf.txt pnp.ids pnpid.txt
endif

all: $(PACKAGENAME) $(PACKAGENAME).1 $(DATAFILES)

.cc.o:
	$(CXX) $(CXXFLAGS) -c $< -o $@

%.gz: %
	$(GZIP) -c $< > $@

.PHONY: core
core:
	+make -C core all

$(PACKAGENAME): core $(PACKAGENAME).o
	$(CXX) $(LDFLAGS) -o $@ $(PACKAGENAME).o $(LIBS)

.PHONY: po
po:
	+make -C po all

.PHONY: gui
gui: core
	+make -C gui all

.PHONY: nologo
nologo:
	cp -f gui/artwork/nologo/* gui/artwork/

.PHONY: static
static: $(PACKAGENAME)-static

$(PACKAGENAME)-static: core core/lib$(PACKAGENAME).a $(PACKAGENAME).o
	$(CXX) $(LDSTATIC) $(LDFLAGS) -o $@ $(PACKAGENAME).o $(LIBS)
	$(STRIP) $@

.PHONY: compressed
compressed: $(PACKAGENAME)-compressed

$(PACKAGENAME)-compressed: $(PACKAGENAME)-static
	upx -9 -o $@ $<

$(PACKAGENAME).1: $(PACKAGENAME).sgml
	docbook2man $<

pci.ids:
	wget http://pciids.sourceforge.net/pci.ids

usb.ids:
	wget http://www.linux-usb.org/usb.ids

oui.txt:
	wget http://standards.ieee.org/regauth/oui/oui.txt

manuf.txt:
	wget -O $@ http://anonsvn.wireshark.org/wireshark/trunk/manuf

pnp.ids:
	wget https://git.fedorahosted.org/cgit/hwdata.git/plain/pnp.ids

pnpid.txt:
	wget http://www-pc.uni-regensburg.de/hardware/TECHNIK/PCI_PNP/pnpid.txt

install: all
	$(INSTALL) -d -m 0755 $(DESTDIR)/$(SBINDIR)
	$(INSTALL) -m 0755 $(PACKAGENAME) $(DESTDIR)/$(SBINDIR)
	$(INSTALL) -d -m 0755 $(DESTDIR)/$(MANDIR)/man1
	$(INSTALL) -m 0644 $(PACKAGENAME).1 $(DESTDIR)/$(MANDIR)/man1
	$(INSTALL) -d -m 0755 $(DESTDIR)/$(DATADIR)/$(PACKAGENAME)
	$(INSTALL) -m 0644 $(DATAFILES) $(DESTDIR)/$(DATADIR)/$(PACKAGENAME)
	make -C po install

install-gui: gui
	$(INSTALL) -d -m 0755 $(DESTDIR)/$(SBINDIR)
	$(INSTALL) -m 0755 gui/gtk-$(PACKAGENAME) $(DESTDIR)/$(SBINDIR)
	$(INSTALL) -d -m 0755 $(DESTDIR)/$(DATADIR)/$(PACKAGENAME)/artwork
	$(INSTALL) -d -m 0755 $(DESTDIR)/$(DATADIR)/$(PACKAGENAME)/ui
	$(INSTALL) -m 0644 gui/*.ui $(DESTDIR)/$(DATADIR)/$(PACKAGENAME)/ui
	$(INSTALL) -m 0644 gui/artwork/*.svg $(DESTDIR)/$(DATADIR)/$(PACKAGENAME)/artwork

clean:
	rm -f $(PACKAGENAME).o $(PACKAGENAME) $(PACKAGENAME)-static $(PACKAGENAME)-compressed
	rm -f $(addsuffix .gz,$(DATAFILES))
	make -C core clean
	make -C gui clean

depend:
	@makedepend -Y $(SRCS) 2> /dev/null > /dev/null


# DO NOT DELETE
