#!/usr/bin/make -f
all:
initial_variables :=$(.VARIABLES)

include ../common.mk

# set this to empty string to see all executed commands
# like tis: $ make Q=
Q :=@

program :=tester
odr :=odr_program

CXX ?=g++

HAVE_CLANG := 0
ifeq ($(shell $(CC) -v 2>&1 | grep -c "clang version"), 1)
    HAVE_CLANG := 1
endif

CXXFLAGS += -std=c++14
CXXFLAGS += -pedantic-errors
CXXFLAGS += -Wall -Wextra -Wswitch-enum -Wnarrowing
ifneq ($(HAVE_CLANG),1)
    CXXFLAGS += -Wabi-tag
else
    CXXFLAGS += -DHAVE_CLANG
endif
CXXFLAGS += -Werror
CXXFLAGS += -fPIC
CXXFLAGS += -fsanitize=address
CXXFLAGS += -fsanitize=leak
CXXFLAGS += -fsanitize=undefined
#CXXFLAGS += -flto
CXXFLAGS += -g3
CXXFLAGS += -I../include/
CXXFLAGS += -I../third_party/bandit/
CXXFLAGS += -DDEBUG
#CXXFLAGS += -O3 -flto -Werror
CXXFLAGS += -O0
CXXFLAGS += $(CXXEF)

LDFLAGS += -fuse-linker-plugin
LDFLAGS += -Wl,--no-undefined
LDFLAGS += -Wl,--warn-common
LDFLAGS += -Wl,-z,defs
LDFLAGS += -rdynamic
LDFLAGS += -L/usr/lib64/mysql/
LDFLAGS += $(LDEF)
LDLIBS += -lstdc++
LDLIBS += -lpthread
LDLIBS += -lmysqlclient


mangle-path =$(subst /,-_-,$1)
demangle-path =$(subst -_-,/,$1)
library_header_files :=$(sort $(patsubst ../include/%,%,$(shell find ../include/ -name "*.hpp")))
odr_library_header_files :=$(filter-out superior_mysqlpp/dnsa_connection_pool.hpp,$(library_header_files))
odr_source_files :=./odr/main.odr.cpp $(foreach i,1 2,$(foreach header_file,$(odr_library_header_files),./odr/$(call mangle-path,$(header_file:.hpp=.$(i).odr.cpp))))
odr_object_files :=$(odr_source_files:.cpp=.o)
odr_dependency_files :=$(odr_object_files:.o=.d)

source_files :=$(sort $(shell find ./ -name "*.cpp" -not -name "*.odr.cpp"))
object_files :=$(source_files:.cpp=.o)
dependency_files :=$(object_files:.o=.d)

# =====================
# === compile rules ===
# =====================
# for debugging
Q :=@

print-variables:
	$(foreach v,$(filter-out $(initial_variables) initial_variables,$(.VARIABLES)),$(info $(v)=$($(v))))

# disable all default rules
.SUFFIXES:
.PHONY: all build test clean build-library build-odr-test test-basic
.PRECIOUS: ./odr/. $(odr_source_files)

# rule for all dirs (makefile does not get "%/:")
%/.:
	$(Q) mkdir -p $@
	$(info Generating test files for ODR...)

odr/main.odr.cpp: $(filter-out $(wildcard ./odr/.), ./odr/.)
	$(Q) echo "int main(){return 0;}" > $@

%.1.odr.cpp: $(filter-out $(wildcard ./odr/.), ./odr/.)
	$(Q) echo "#include <$(call demangle-path,$(patsubst odr/%.1.odr.cpp,%.hpp,$@))>" > $@

%.2.odr.cpp: $(filter-out $(wildcard ./odr/.), ./odr/.)
	$(Q) echo "#include <$(call demangle-path,$(patsubst odr/%.2.odr.cpp,%.hpp,$@))>" > $@


%.odr.o: %.odr.cpp
	$(Q)$(strip $(COMPILE.cpp) -MMD $(OUTPUT_OPTION) $<)
$(foreach file,$(odr_dependency_files),$(eval -include $(file)))

%.o: %.cpp
	$(strip $(COMPILE.cpp) -MMD $(OUTPUT_OPTION) $<)
$(foreach file,$(dependency_files),$(eval -include $(file)))


$(program): $(object_files)
	$(strip $(LINK.cpp) $^ $(LOADLIBES) $(LDLIBS) -o $@)

$(odr): $(odr_object_files)
	$(Q) $(strip $(LINK.cpp) $^ $(LOADLIBES) $(LDLIBS) -o $@)

# =====================
# === runable rules ===
# =====================
all: build



build-odr: $(odr)
build-basic: $(program)

build: build-basic build-odr

test-basic: tester
	./runtest.sh

test: test-basic build-odr 

clean-files =find ./ -type f -path "$1" -exec $(RM) {} \;

clean:
	$(RM) $(program) $(odr)
	$(RM) -fR ./odr/
	$(call clean-files,*.d)
	$(call clean-files,*.o)
