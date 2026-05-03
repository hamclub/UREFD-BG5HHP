#Copyright (C) 2021 by Thomas A. Early, N7TAE

#include tcd.mk
EXE = tcd

CC ?=gcc
CXX ?=g++
BINDIR ?=/usr/local/bin
DEBUG ?=0
SWAMBE2 ?=0
SWMODES ?=1

CXXFLAGS = -W -Wall -std=c++17 -Ilib -Icodec2 -fno-rtti
ifeq ($(DEBUG), 1)
CXXFLAGS += -g -DDEBUG -MMD
else
CXXFLAGS += -Os -DNDEBUG
endif

ifeq ($(SWAMBE2), 1)
CXXFLAGS+= -DUSE_SW_AMBE2
LDFLAGS += -lmd380_vocoder
endif

LDFLAGS = -limbe_vocoder -pthread

ifeq ($(SWMODES), 1)
CXXFLAGS+= -DSW_MODES_ONLY
else
LDFLAGS += -lftd2xx
endif

SRCS = $(wildcard *.cpp) $(wildcard codec2/*.cpp)
OBJS = $(SRCS:.cpp=.o)
DEPS = $(SRCS:.cpp=.d)

$(EXE) : $(OBJS)
ifeq ($(SWAMBE2), 1)
	$(CXX) $(OBJS) $(LDFLAGS) -o $@ -Xlinker --section-start=.firmware=0x0800C000 -Xlinker  --section-start=.sram=0x20000000
else
	$(CXX) $(OBJS) $(LDFLAGS) -o $@
endif

%.o : %.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@

clean :
	$(RM) $(EXE) $(OBJS) $(DEPS)

-include $(DEPS)

# The install and uninstall targets need to be run by root
install : $(EXE)
	cp $(EXE) $(BINDIR)
	cp $(EXE).service /etc/systemd/system/
	systemctl enable $(EXE)
	systemctl daemon-reload
	systemctl start $(EXE)

uninstall :
	systemctl stop $(EXE)
	systemctl disable $(EXE)
	systemctl daemon-reload
	rm -f /etc/systemd/system/$(EXE).service
	rm -f $(BINDIR)/$(EXE)
	systemctl daemon-reload
