REPORT	:= none
TARGET	:= hw
PLATFORM:= zcu102_es2_ocl
BUILD_DIR:= ./build/$(PLATFORM)_$(TARGET)

CXX 	:= $(XILINX_SDX)/SDK/gnu/aarch64/lin/aarch64-linux/bin/aarch64-linux-gnu-g++
XOCC 	:= $(XILINX_SDX)/bin/xocc

# Points to Utility Directory
COMMON_REPO = ../../../
ABS_COMMON_REPO = $(shell readlink -f $(COMMON_REPO))
include $(ABS_COMMON_REPO)/libs/xcl2/xcl2.mk

# host compiler global settings
CXXFLAGS += -I $(XILINX_SDX)/runtime/include/1_2/ -I/$(XILINX_SDX)/Vivado_HLS/include/ -O2 -g -Wall -fmessage-length=0 -std=c++14
LDFLAGS += -lxilinxopencl -lpthread -lrt -ldl -lcrypt -lstdc++ -L$(XILINX_SDX)/runtime/lib/aarch64 $(xcl2_CXXFLAGS)

# kernel compiler global settings
CLFLAGS = -t hw --platform $(PLATFORM) --save-temps  --clkid 0 --xp param:compiler.xclDataflowFifoDepth=32

ifneq ($(REPORT),none)
CLFLAGS += --report $(REPORT)
endif

BINARY_CONTAINERS += $(BUILD_DIR)/adder.xclbin

BINARY_CONTAINER_1_OBJS += $(BUILD_DIR)/adder.xo
ALL_KERNEL_OBJS += $(BUILD_DIR)/adder.xo

ALL_MESSAGE_FILES = $(subst .xo,.mdb,$(ALL_KERNEL_OBJS)) $(subst .xclbin,.mdb,$(BINARY_CONTAINERS))

HOST_SRCS = src/host.cpp $(xcl2_SRCS)

EXECUTABLE = adder

.PHONY: all clean cleanall docs
all: $(BINARY_CONTAINERS) $(BUILD_DIR)/$(EXECUTABLE) 

#Building Kernel
$(BUILD_DIR)/adder.xo: ./src/adder.cl
	@mkdir -p $(BUILD_DIR)
	-@$(RM) $@
	@echo 'Building Target: $@'
	$(XOCC) $(CLFLAGS) -c -k adder --messageDb $(subst .xo,.mdb,$@) -I"$(<D)" --xp misc:solution_name=_xocc_compile_adder -o"$@" "$<"
	@echo 'SDx Completed Building Target: $@'

$(BUILD_DIR)/adder.xclbin: $(BINARY_CONTAINER_1_OBJS)
	$(XOCC) $(CLFLAGS) -l --nk adder:1 --messageDb $(subst .xclbin,.mdb,$@) --xp misc:solution_name=_xocc_link_adder -o"$@" $(+) --sys_config ocl

#Building Host 
$(BUILD_DIR)/$(EXECUTABLE): $(HOST_SRCS)
	mkdir -p $(BUILD_DIR)
	@echo 'Building Target: $@'
	$(CXX) -o "$@" $(HOST_SRCS) $(CXXFLAGS) $(LDFLAGS)
	@echo 'SDx Completed Building Target: $@'

#Cleaning Stuff
RM = rm -f
RMDIR = rm -rf
clean:
	-$(RM) $(ALL_MESSAGE_FILES) $(EXECUTABLE) 
	-$(RMDIR) embedded_root _xocc_* iprepo _sds
	-$(RM) *.xml *.tcl *.bat image.ub
	-$(RM) init.sh *.bit *.hdf $(PLATFORM)_* *.dat
	-$(RM) platform_desc.txt *.xtxt *.ko *.hpfm *.so
	-$(RMDIR) .Xil

cleanall: clean
	-$(RM) $(BINARY_CONTAINERS) $(ALL_KERNEL_OBJS) $(EXECUTABLE)
	-$(RMDIR) $(BUILD_DIR) ./build

docs:README.md

README.md: description.json
	$(ABS_COMMON_REPO)/ocl/common/readme_gen/readme_gen.py description.json
