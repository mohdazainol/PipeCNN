#set source and target for host
HOST_EXE  = run.exe
HOST_SRCS = ./host/main.cpp ../common/ocl_util.cpp ../common/timer.cpp
HOST_OBJS = $(HOST_SRCS:%.cpp=%.o)
HOST_INCS = ../common
HOST_DEBUG =

#set source and target for device
#supported vendors are xilinx, altera, amd
VENDOR = altera
#select the host archecture  x86|arm32
PLATFORM = x86
#optinal flows are hw|hw_emu|sw_emu
FLOW        = hw
KERNEL_SRCS = ./device/conv_pipe.cl
KERNEL_NAME = conv
KERNEL_DEFS =
KERNEL_INCS = 
KERNEL_DEBUG = 1
#additional pragma for xilinx
CU_NUM = 1
MAX_MEM_PORT = 1
MEM_PORT_SIZE = 512

#select the device binary name
ifeq ($(VENDOR),xilinx)
DEV_EXE = $(KERNEL_NAME).xclbin
else ifeq ($(VENDOR),altera)
DEV_EXE = $(KERNEL_NAME).aocx
else ifeq ($(VENDOR),amd)
DEV_EXE =
endif

#host compiler options
CROSS-COMPILE = arm-linux-gnueabihf-
ifeq ($(PLATFORM),x86)
CXX := g++
else ifeq ($(PLATFORM),arm32)
CXX := $(CROSS-COMPILE)g++
endif

CXXFLAGS = -g -Wall -std=c++11

ifeq ($(VENDOR),xilinx)
COMP_CONFIG = -I$(XILINX_SDACCEL)/runtime/include/1_2 -I$(HOST_INCS)  -DFPGA_DEVICE
LINK_CONFIG = -L$(XILINX_SDACCEL)/runtime/lib/x86_64 -lxilinxopencl
else ifeq ($(VENDOR),altera)
COMP_CONFIG = $(shell aocl compile-config) -I$(HOST_INCS)  -DFPGA_DEVICE
LINK_CONFIG = $(shell aocl link-config)
else ifeq ($(VENDOR),amd)
DEV_EXE =
endif

#opencl compiler options
#xilinx
ifeq ($(VENDOR),xilinx)
OCC = xocc
OCCFLAGS = --xdevice xilinx:adm-pcie-7v3:1ddr:2.0 --nk $(KERNEL_NAME):$(CU_NUM) -t $(FLOW)
#memory optimization for xilinx
ifeq ($(MAX_MEM_PORT), 1)
OCCFLAGS += --max_memory_ports all
OCCFLAGS += --memory_port_data_width all:$(MEM_PORT_SIZE)
endif
#altera
else ifeq ($(VENDOR),altera)
OCC = aoc
ifeq ($(FLOW),sw_emu)
OCCFLAGS = -v --report -march=emulator -I device/RTL -L device/RTL -l rtl_lib.aoclib 
else ifeq ($(FLOW),hw)
OCCFLAGS = -v --report --profile -I device/RTL -L device/RTL -l rtl_lib.aoclib 
endif
#amd
else ifeq ($(VENDOR),amd)
OCC =
OCCFLAGS =
endif

#debug option
ifeq ($(KERNEL_DEBUG),1)
	OCCFLAGS += -g
endif

.PHONY: all
all: host fpga

.PHONY: host
host: $(HOST_EXE)

.PHONY: fpga
fpga: $(DEV_EXE)

$(HOST_EXE): $(HOST_OBJS)
	$(CXX) $(HOST_OBJS) -o $@ $(LINK_CONFIG)

%.o: %.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@ $(COMP_CONFIG)

$(DEV_EXE): $(KERNEL_SRCS)
	$(OCC) $(OCCFLAGS) $< -o $@

.PHONY: clean
clean:
	rm -rf *.xclbin *.aocx *.aoco *.exe xocc* sdaccel* ./device/*.ll ./host/*.o ../common/*.o *.log *.jou
