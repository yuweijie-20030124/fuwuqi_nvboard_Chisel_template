# Chisel + NVBoard 示例工程

这个工程使用 Mill 编译 Chisel，并通过 Verilator 接入 NVBoard。顶层模块只有一个组合逻辑连接：

```text
SW0 (in) ───> out ───> LD0
```

在 `lesson1` 目录执行 `make run` 即可启动。`NVBOARD_HOME` 默认指向本仓库的上一级目录；如果工程被复制到其他位置，请先设置它：

```bash
export NVBOARD_HOME=/path/to/nvboard
make run
```

构建过程会依次执行：

1. Mill 调用 `top.runMain top.topMain -td vsrc --emit-modules verilog`，把 `scala/top.scala` 生成为 `vsrc/top.v`。
2. `auto_pin_bind.py` 根据 `constr/top.nxdc` 生成 NVBoard 引脚绑定代码。
3. Verilator 编译 Verilog、C++ 仿真入口和 NVBoard 库。

拨动 NVBoard 的 `SW0` 后，`LD0` 会显示同一个逻辑值。
