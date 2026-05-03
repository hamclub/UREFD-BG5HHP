# Copyright (c) 2021 by Thomas A. Early N7TAE
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.

# locations for the executibles and other files are set here
# NOTE: IF YOU CHANGE THESE, YOU WILL NEED TO UPDATE THE service.* FILES AND
# if you change these locations, make sure the sgs.service file is updated!
# you will also break hard coded paths in the dashboard file, index.php.

EXE = urefd

INICHECK = inicheck

DBUTIL = dbutil

CC ?=gcc
CXX ?=g++
BINDIR ?=/usr/local/bin
DEBUG ?=0
DHT ?=0
HAS_CURL ?=0

CXXFLAGS = -W -Wall -std=c++17 -Ilib -fno-rtti
ifeq ($(DEBUG), 1)
CXXFLAGS += -g -ggdb -DDEBUG -MMD
else
CXXFLAGS += -Os -DNDEBUG
endif

LDFLAGS=-pthread

ifeq ($(DHT), 1)
LDFLAGS += -lopendht
else
CXXFLAGS += -DNO_DHT
endif

ifeq ($(HAS_CURL), 1)
CXXFLAGS += -DHAS_CURL
LDFLAGS += -lcurl
endif

ifeq ($(shell uname -s),Darwin)
	CXXFLAGS+= -I/opt/homebrew/include
	LDFLAGS+= -L/opt/homebrew/lib
endif

SRCS = $(wildcard *.cpp)
OBJS = $(SRCS:.cpp=.o)
DEPS = $(SRCS:.cpp=.d)
DBUTILOBJS = Configure.o CurlGet.o Lookup.o LookupDmr.o LookupNxdn.o LookupYsf.o YSFNode.o Callsign.o

all : $(EXE) $(INICHECK) $(DBUTIL)

$(EXE) : $(OBJS)
	$(CXX) $^ -o $@ $(LDFLAGS)

$(INICHECK) : Configure.cpp CurlGet.o
	$(CXX) -DINICHECK $(CXXFLAGS) $< CurlGet.o -o $@ $(LDFLAGS)

$(DBUTIL) : Main.cpp $(DBUTILOBJS)
	$(CXX) -DUTILITY $(CXXFLAGS) $< $(DBUTILOBJS) -o $@ $(LDFLAGS)

%.o : %.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@

clean :
	$(RM) *.o *.d $(EXE) $(INICHECK) $(DBUTIL)

-include $(DEPS)

install :
	cp -f $(EXE).service /etc/systemd/system/
	cp -f $(EXE) $(BINDIR)
	systemctl enable $(EXE).service
	systemctl daemon-reload
	systemctl start $(EXE)

uninstall :
	systemctl stop $(EXE).service
	systemctl disable $(EXE).service
	$(RM) /etc/systemd/system/$(EXE).service
	systemctl daemon-reload
	$(RM) $(BINDIR)/$(EXE)
