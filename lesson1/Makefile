TOP = top
TOPNAME = $(TOP)
MAIN = top.topMain
NXDC_FILES = constr/top.nxdc
INC_PATH ?=
NVBOARD_HOME ?= $(abspath ..)

MILL ?= mill

# Chisel 3.5.0 + Scala 2.12.12 需要 JDK 8-16(JDK 21 下旧 Scala 编译器会崩溃),
# 因此固定使用 JDK 11。若想覆盖,请通过环境变量传入 JAVA_HOME。
JAVA_HOME ?= $(HOME)/jdks/jdk11
export JAVA_HOME

VERILATOR = verilator
VERILATOR_CFLAGS += -MMD --build -cc  \
				-O3 --x-assign fast --x-initial fast --noassert

BUILD_DIR = ./build
OBJ_DIR = $(BUILD_DIR)/obj_dir
BIN = $(BUILD_DIR)/$(TOPNAME)

### Scala/Chisel
SCALA_FILE = $(shell find ./scala -name '*.scala')
SCALA_2_VERILOG_DIR = vsrc
VSRCS = $(SCALA_2_VERILOG_DIR)/$(TOPNAME).v
default: $(BIN)

$(shell mkdir -p $(BUILD_DIR))

# constraint file
SRC_AUTO_BIND = $(abspath $(BUILD_DIR)/auto_bind.cpp)
$(SRC_AUTO_BIND): $(NXDC_FILES)
	python3 $(NVBOARD_HOME)/scripts/auto_pin_bind.py $^ $@

# project source
CSRCS = $(shell find $(abspath ./csrc) -name "*.c" -or -name "*.cc" -or -name "*.cpp")
CSRCS += $(SRC_AUTO_BIND)

# rules for NVBoard
include $(NVBOARD_HOME)/scripts/nvboard.mk

# rules for verilator
INCFLAGS = $(addprefix -I, $(INC_PATH))
CXXFLAGS += $(INCFLAGS) -DTOP_NAME="\"V$(TOPNAME)\""

$(BIN): $(VSRCS) $(CSRCS) $(NVBOARD_ARCHIVE)
	@rm -rf $(OBJ_DIR)
	$(VERILATOR) $(VERILATOR_CFLAGS) \
		--top-module $(TOPNAME) $^ \
		$(addprefix -CFLAGS , $(CXXFLAGS)) $(addprefix -LDFLAGS , $(LDFLAGS)) \
		--Mdir $(OBJ_DIR) --exe -o $(abspath $(BIN))


$(VSRCS): $(SCALA_FILE) build.sc
	@rm -rf $(SCALA_2_VERILOG_DIR)
	@mkdir -p $(SCALA_2_VERILOG_DIR)
	$(MILL) -i $(TOP).runMain $(MAIN) -td $(SCALA_2_VERILOG_DIR) --emit-modules verilog

verilog: $(VSRCS)

all: default

run: $(BIN)
	@$^

clean:
	rm -rf $(BUILD_DIR)
	rm -rf $(SCALA_2_VERILOG_DIR)
	rm -rf out

.PHONY: default verilog all clean run
