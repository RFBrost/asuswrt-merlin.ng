#
# Miscellaneous utilities Makefile
#
# Copyright (C) 2009, Broadcom Corporation
# All Rights Reserved.
#
# THIS SOFTWARE IS OFFERED "AS IS", AND BROADCOM GRANTS NO WARRANTIES OF ANY
# KIND, EXPRESS OR IMPLIED, BY STATUTE, COMMUNICATION OR OTHERWISE. BROADCOM
# SPECIFICALLY DISCLAIMS ANY IMPLIED WARRANTIES OF MERCHANTABILITY, FITNESS
# FOR A SPECIFIC PURPOSE OR NONINFRINGEMENT CONCERNING THIS SOFTWARE.
#
# $Id: Makefile 449389 2014-01-16 23:23:38Z $
#

include $(SRCBASE)/router/.config

ifneq ($(wildcard $(SRCBASE)/wl/exe/prebuilt/wl),)
PREBUILT_BCMBIN=1
endif

WLU_OBJS := wlu.o wlu_cmd.o wlu_iov.o wlu_linux.o bcmutils.o wlu_client_shared.o wlu_pipe.o wlu_pipe_linux.o miniopt.o bcmxtlv.o bcm_app_utils.o wlu_rates_matrix.o wlc_ppr.o
ifneq ($(wildcard $(SRCBASE)/wl/exe/wlu_common.c),)
WLU_OBJS += wlu_common.o
endif

ifeq ($(RWL),1)
WL_OBJS := $(WLU_OBJS)
SERVER_OBJS := wlu_server_linux.o wlu_pipe_linux.o wlu_pipe.o wlu_server_shared.o shellproc_linux.o
APUT_OBJS := wfa_apcmdproc.o wfa_apcmdlib.o wfa_apcs.o wfa_apcmdresp.o wfa_apcmdvalid.o wfa_apcmdtbls.o wfa_apcmdcvt.o wfa_val2str.o wfa_apdiy.o wfa_sock.o wfa_aput_all.o

SOCKET_OBJS := $(SERVER_OBJS:%.o=socket/%.o)
WIFI_OBJS := $(SERVER_OBJS:%.o=wifi/%.o)
WL_OBJS := $(WL_OBJS:%.o=wl_obj/%.o)
WFA_APUT_OBJS := $(APUT_OBJS:%.o=aput/%.o)

SERVER_SOCKET  := wl_server_socket
SERVER_WIFI  := wl_server_wifi
WFA_APUT_ALL := wfa_aput_all
WL_EXE  := wl
endif

UTILS :=
ifneq ($(PREBUILT_BCMBIN),1)
UTILS += epi_ttcp
endif
CFLAGS += -s -I$(SRCBASE)/include ${WLAN_StdIncPathA} -I$(SRCBASE)/common/include
CFLAGS += $(if $(WLAN_ComponentIncPath),$(WLAN_ComponentIncPath),$(addprefix -I,$(wildcard $(SRCBASE)/shared/bcmwifi/include)))
ifeq ($(CONFIG_RTR_OPTIMIZE_SIZE),y)
CFLAGS  += -Os
else
CFLAGS  += -O2
endif

CFLAGS += -DLINUX -I$(SRCBASE)/wl/exe

CFLAGS += -DPPR_API

#ifdef WLCNT
CFLAGS += -DWLCNT
#endif

ifeq ($(CONFIG_TRAFFIC_MGMT),y)
CFLAGS += -DTRAFFIC_MGMT
endif

## Enable DMA RX overflow fix
ifeq ($(ET_INGRESS_QOS),1)
CFLAGS += -DET_INGRESS_QOS
endif

vpath %.c $(SRCBASE)/tools/misc $(WLAN_StdSrcDirsA) $(WLAN_ComponentSrcDirs)

ifneq ($(wildcard $(SRCBASE)/et/linux/et.c),)
UTILS += et
vpath %.c $(SRCBASE)/et/linux
endif

ifneq ($(wildcard $(SRCBASE)/wl/exe),)
ifneq ($(RTCONFIG_WLEXE), y)
CFLAGS += -I$(SRCBASE)/wl/exe
UTILS += wl
vpath %.c $(SRCBASE)/wl/exe $(SRCBASE)/shared
endif
endif

ifeq ($(RWL),1)
ifeq ($(BUILD_WFA_APUT),1)
ifneq ($(wildcard $(SRCBASE)/router/aput),)
CFLAGS += -I$(SRCBASE)/router/aput
UTILS += $(WFA_APUT_ALL)
vpath %.c $(SRCBASE)/router/aput
endif
endif

ifneq ($(wildcard $(SRCBASE)/wl/exe),)
UTILS += $(SERVER_SOCKET)
vpath %.c $(SRCBASE)/wl/exe $(SRCBASE)/shared
endif

ifneq ($(wildcard $(SRCBASE)/wl/exe),)
UTILS += $(SERVER_WIFI)
vpath %.c $(SRCBASE)/wl/exe $(SRCBASE)/shared
endif
endif

LDFLAGS += -L$(TOP_PLATFORM)/nvram${BCMEX}${EX7} -lnvram ${EXTRA_NV_LDFLAGS}
LDFLAGS += -L$(TOP)/shared -L$(INSTALLDIR)/shared/usr/lib -lshared -lpthread
LDFLAGS += -lgcc_s

ifeq ($(RTCONFIG_QTN),y)
CFLAGS += -I$(TOP)/libqcsapi_client -I$(TOP)/httpd -I./sysdeps/qtn
LDFLAGS += -L$(TOP)/libqcsapi_client -lqcsapi_client
endif

ifeq ($(RTCONFIG_5301X),y)
CFLAGS += -DBCM5301X
endif
ifeq ($(PREBUILT_BCMBIN),1)
UTILS := wl
endif

# contains e.g. WlGetDriverCfg.sh
export APPS_UTILS_DIR := $(BASEDIR)/components/apps/utils

all: $(UTILS)

ifeq ($(PREBUILT_BCMBIN),1)
wl:
	-cp -f $(SRCBASE)/wl/exe/prebuilt/wl wl
else
wl: $(WLU_OBJS)
	$(CC) $^ -o $@ $(LDFLAGS)
endif

ifeq ($(RWL),1)
$(WFA_APUT_ALL): $(WFA_APUT_OBJS)
	$(CC) $(LDFLAGS) -o $@ $^

aput/%.o: %.c
	@mkdir -p aput
	$(CC) -c $(CFLAGS) -o $@ $^
$(SERVER_SOCKET): $(SOCKET_OBJS)
	$(CC) $(LDFLAGS) -o $@ $^

socket/%.o: %.c
	@mkdir -p socket
	$(CC) -c $(CFLAGS) -DRWL_SOCKET -o $@ $^

$(SERVER_WIFI): $(WIFI_OBJS)
	$(CC) $(LDFLAGS) -o $@ $^

wifi/%.o: %.c
	@mkdir -p wifi
	$(CC) -c $(CFLAGS) -DRWL_WIFI -o $@ $^
endif

ifeq ($(PREBUILT_BCMBIN),1)
robocfg:
	-cp -f $(SRCBASE)/wl/exe/prebuilt/robocfg robocfg
else
robocfg: robocfg.c etc53xx.h
endif

clean:
	rm -f *.o *~ $(UTILS)
	rm -rf socket wifi

install: all
	install -d $(INSTALLDIR)/usr/sbin
	install $(UTILS) $(INSTALLDIR)/usr/sbin
	$(STRIP) $(foreach file,$(UTILS),$(INSTALLDIR)/usr/sbin/$(file))
	install -m 755 $(APPS_UTILS_DIR)/WlGetDriverCfg.sh $(INSTALLDIR)/usr/sbin
	install -m 755 $(APPS_UTILS_DIR)/WlGetDriverStats.sh $(INSTALLDIR)/usr/sbin
	install -m 755 $(APPS_UTILS_DIR)/debug_loop_acs_dfs.sh $(INSTALLDIR)/usr/sbin
	install -m 755 $(APPS_UTILS_DIR)/mapgetcfg.sh $(INSTALLDIR)/usr/sbin

