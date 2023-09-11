# Based on c_src.mk from erlang.mk by Loic Hoguin <essen@ninenines.eu>

CURDIR := $(shell pwd)
BASEDIR := $(abspath $(CURDIR)/..)

PROJECT ?= $(notdir $(BASEDIR))
PROJECT := $(strip $(PROJECT))

BUILD_DIR = $(REBAR_BUILD_DIR)/lib/$(PROJECT)
PRIV_DIR = $(BUILD_DIR)/priv

ERTS_INCLUDE_DIR ?= $(shell erl -noshell -eval "io:format(\"~ts/erts-~ts/include/\", [code:root_dir(), erlang:system_info(version)])." -s init stop)
ERL_INTERFACE_INCLUDE_DIR ?= $(shell erl -noshell -eval "io:format(\"~ts\", [code:lib_dir(erl_interface, include)])." -s init stop)
ERL_INTERFACE_LIB_DIR ?= $(shell erl -noshell -eval "io:format(\"~ts\", [code:lib_dir(erl_interface, lib)])." -s init stop)

C_SRC_DIR = $(CURDIR)
C_SRC_OUTPUT ?= $(BUILD_DIR)/priv/$(PROJECT).so
C_SRC_BUILD_DIR ?= $(BUILD_DIR)/c_build

# System type and C compiler/flags.

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
endif

CFLAGS += -fPIC -I $(ERTS_INCLUDE_DIR) -I $(ERL_INTERFACE_INCLUDE_DIR)
CXXFLAGS += -fPIC -I $(ERTS_INCLUDE_DIR) -I $(ERL_INTERFACE_INCLUDE_DIR)

LDLIBS += -L $(ERL_INTERFACE_LIB_DIR) -lei
LDFLAGS += -shared

# Verbosity.

c_verbose_0 = @echo " C     " $(?F);
c_verbose = $(c_verbose_$(V))

cpp_verbose_0 = @echo " CPP   " $(?F);
cpp_verbose = $(cpp_verbose_$(V))

link_verbose_0 = @echo " LD    " $(@F);
link_verbose = $(link_verbose_$(V))

SOURCES := $(shell find $(C_SRC_DIR) -type f \( -name "*.c" -o -name "*.C" -o -name "*.cc" -o -name "*.cpp" \))
OBJECTS = $(SOURCES:$(C_SRC_DIR)/%.c=$(C_SRC_BUILD_DIR)/%.o)

COMPILE_C = $(c_verbose) $(CC) $(CFLAGS) $(CPPFLAGS) -c
COMPILE_CPP = $(cpp_verbose) $(CXX) $(CXXFLAGS) $(CPPFLAGS) -c

$(C_SRC_OUTPUT): $(PRIV_DIR) $(C_SRC_BUILD_DIR) $(OBJECTS)
	$(link_verbose) $(CC) $(OBJECTS) $(LDFLAGS) $(LDLIBS) -o $(C_SRC_OUTPUT)

$(PRIV_DIR) $(C_SRC_BUILD_DIR):
	@rm -f $@
	@mkdir -p $@

$(C_SRC_BUILD_DIR)/%.o: $(C_SRC_DIR)/%.c
	$(COMPILE_C) $(OUTPUT_OPTION) -o $@ $<

$(C_SRC_BUILD_DIR)/%.o: $(C_SRC_DIR)/%.cc
	$(COMPILE_CPP) $(OUTPUT_OPTION) -o $@ $<

$(C_SRC_BUILD_DIR)/%.o: $(C_SRC_DIR)/%.C
	$(COMPILE_CPP) $(OUTPUT_OPTION) -o $@ $<

$(C_SRC_BUILD_DIR)/%.o: $(C_SRC_DIR)/%.cpp
	$(COMPILE_CPP) $(OUTPUT_OPTION) -o $@ $<

clean:
	@rm -f $(C_SRC_OUTPUT) $(OBJECTS)
