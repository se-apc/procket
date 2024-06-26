CURDIR := $(shell pwd)
BASEDIR := $(abspath $(CURDIR)/..)

PROJECT ?= $(notdir $(BASEDIR))
PROJECT := $(strip $(PROJECT))


C_SRC_DIR = $(CURDIR)
C_SRC_OUTPUT ?= $(CURDIR)/../priv/$(PROJECT).so

# System type and C compiler/flags.

ifeq ($(CROSSCOMPILE),)
ERTS_INCLUDE_DIR ?= $(shell erl -noshell -eval "io:format(\"~s/erts-~s/include/\", [code:root_dir(), erlang:system_info(version)])." -s erlang halt)
ERLANG_ARCH ?= $(shell erl -noshell -eval "io:format(\"~B\", [erlang:system_info({wordsize,external}) * 8])." -s erlang halt)

UNAME_SYS := $(shell uname -s)
ifeq ($(UNAME_SYS), Darwin)
	CC ?= cc
	CFLAGS ?= -O3 -std=c99 -finline-functions -Wall -Wmissing-prototypes
	CXXFLAGS ?= -O3 -finline-functions -Wall
	LDFLAGS ?= -flat_namespace -undefined suppress
else ifeq ($(UNAME_SYS), FreeBSD)
	CC ?= cc
	CFLAGS ?= -O3 -std=c99 -finline-functions -Wall -Wmissing-prototypes
	CXXFLAGS ?= -O3 -finline-functions -Wall
else ifeq ($(UNAME_SYS), Linux)
	CC ?= gcc
	CFLAGS ?= -O3 -std=c99 -finline-functions -Wall -Wmissing-prototypes
	CXXFLAGS ?= -O3 -finline-functions -Wall
else ifeq ($(UNAME_SYS), SunOS)
	CFLAGS += -std=c99 -D_POSIX_C_SOURCE=200112L -lsocket -lnsl
	PROCKET_CFLAGS = -lsocket -lnsl
endif

UNAME_ARCH := $(shell uname -m)
ifeq ($(UNAME_ARCH), x86_64)
	PROCKET_CFLAGS += -m$(ERLANG_ARCH)
	CFLAGS += -m$(ERLANG_ARCH)
else ifeq ($(UNAME_ARCH), arm64)
	PROCKET_CFLAGS += -m$(ERLANG_ARCH)
	CFLAGS += -m$(ERLANG_ARCH)
	LDFLAGS += -undefined dynamic_lookup
else ifeq ($(UNAME_ARCH), i686)
	PROCKET_CFLAGS += -m$(ERLANG_ARCH)
	CFLAGS += -m$(ERLANG_ARCH)
else ifeq ($(UNAME_ARCH), i386)
	PROCKET_CFLAGS += -m$(ERLANG_ARCH)
	CFLAGS += -m$(ERLANG_ARCH)
endif
endif

CFLAGS += -fPIC -I $(ERTS_INCLUDE_DIR)
CXXFLAGS += -fPIC -I $(ERTS_INCLUDE_DIR)

CFLAGS += $(ERL_CFLAGS)
CFLAGS += -O3

LDLIBS += -L$(C_SRC_DIR) -lancillary
LDFLAGS += -shared
LDFLAGS += $(ERL_LDFLAGS)

PROCKET_CFLAGS += $(PROCKET_DEFINE)

# Verbosity.

c_verbose_0 = @echo " C     " $(?F);
c_verbose = $(c_verbose_$(V))

cpp_verbose_0 = @echo " CPP   " $(?F);
cpp_verbose = $(cpp_verbose_$(V))

link_verbose_0 = @echo " LD    " $(@F);
link_verbose = $(link_verbose_$(V))

SOURCES := $(C_SRC_DIR)/procket.c
OBJECTS = $(addsuffix .o, $(basename $(SOURCES)))

COMPILE_C = $(c_verbose) $(CC) $(CFLAGS) $(CPPFLAGS) -c
COMPILE_CPP = $(cpp_verbose) $(CXX) $(CXXFLAGS) $(CPPFLAGS) -c

$(C_SRC_OUTPUT): privdir ancillary cmd $(OBJECTS)
	$(link_verbose) $(CC) $(OBJECTS) $(LDFLAGS) $(LDLIBS) -o $(C_SRC_OUTPUT)

%.o: %.c
	$(COMPILE_C) $(PROCKET_DEFINE) $(OUTPUT_OPTION) $<

%.o: %.cc
	$(COMPILE_CPP) $(OUTPUT_OPTION) $<

%.o: %.C
	$(COMPILE_CPP) $(OUTPUT_OPTION) $<

%.o: %.cpp
	$(COMPILE_CPP) $(OUTPUT_OPTION) $<

clean:
	@rm -f $(C_SRC_OUTPUT) $(OBJECTS) $(C_SRC_DIR)/*.a $(C_SRC_DIR)/*.o priv/procket priv/*.so

privdir:
	@mkdir -p $(BASEDIR)/priv/

ancillary:
	$(MAKE) -f $(C_SRC_DIR)/Makefile.ancillary

cmd:
	$(CC) $(PROCKET_CFLAGS) -g -Wall -o $(BASEDIR)/priv/procket -L$(C_SRC_DIR) procket_cmd.c -lancillary
