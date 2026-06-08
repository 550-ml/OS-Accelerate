# OS-Accelerate

一个面向操作系统与高性能计算学习的矩阵乘法优化实验项目。项目以经典的
`C = A x B` 为核心，从朴素三重循环出发，逐步引入循环展开、寄存器缓存、
分块、矩阵打包、SIMD 和多线程等优化手段，用更直观的代码观察性能如何被
一点点“榨”出来。

## 项目亮点

- **可对照的基准实现**：`basic_matrix_multiplication` 保留朴素版本，方便做正确性校验。
- **多线程并行**：使用 `pthread` 将矩阵按行切分，让多个线程协同计算。
- **缓存友好的分块思路**：通过 block 和 pack 降低访存开销，提升数据局部性。
- **SIMD 实验代码**：`multiply.cpp` 中包含 SSE 向量化路径，适合在 x86/x64 环境继续调优。
- **内置性能统计**：程序会输出平均耗时与 GFLOPS，便于比较不同优化策略。

## 目录结构

```text
.
├── README.md
└── src
    ├── main.cpp        # 程序入口：初始化矩阵、正确性验证、性能计时
    ├── main.h          # 矩阵规模宏 N / M / P
    ├── matrix.cpp      # 随机初始化、朴素矩阵乘法、结果校验
    ├── matrix.h
    ├── multiply.cpp    # 分块、打包、SSE、多线程等优化实现
    ├── multiply12.cpp  # 基础 pthread 多线程实现
    └── multiply.h
```

## 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/550-ml/OS-Accelerate.git
cd OS-Accelerate
```

### 2. 编译运行

在 x86/x64 Linux 环境中，可以优先编译当前优化主实现：

```bash
g++ -std=c++11 -O2 src/main.cpp src/matrix.cpp src/multiply.cpp -lpthread -o os_accelerate
./os_accelerate
```

如果你在 Apple Silicon 或其他不支持 SSE 的平台上运行，可以先使用基础
pthread 版本：

```bash
g++ -std=c++11 -O2 -include cmath src/main.cpp src/matrix.cpp src/multiply12.cpp -lpthread -o os_accelerate
./os_accelerate
```

运行后会看到类似输出：

```text
[LOG] N=1024, M=1024, P=1024
[LOG] JUDGE_TIME
[LOG] loop=0, time=11390.000000 us
[LOG] average time=11048.800000 us
[LOG] GFlops = 194.363519
```

## 正确性验证

开启 `JUDGE_RIGHT` 宏后，程序会用朴素矩阵乘法生成标准答案，再与优化版本逐元素比较：

```bash
g++ -std=c++11 -O2 -DJUDGE_RIGHT src/main.cpp src/matrix.cpp src/multiply.cpp -lpthread -o os_accelerate_check
./os_accelerate_check
```

输出 `Accepted` 表示优化实现与基准实现结果一致。

在非 x86/x64 平台上，可以同样切换到 `multiply12.cpp`：

```bash
g++ -std=c++11 -O2 -include cmath -DJUDGE_RIGHT src/main.cpp src/matrix.cpp src/multiply12.cpp -lpthread -o os_accelerate_check
./os_accelerate_check
```

## 调整矩阵规模

默认矩阵规模定义在 `src/main.h`：

```cpp
#define N 1024
#define M 1024
#define P 1024
```

也可以在编译时直接覆盖宏：

```bash
g++ -std=c++11 -O2 -DN=512 -DM=512 -DP=512 src/main.cpp src/matrix.cpp src/multiply.cpp -lpthread -o os_accelerate
```

其中：

- `N`：矩阵 A 的行数，也是结果矩阵 C 的行数
- `M`：矩阵 A 的列数、矩阵 B 的行数
- `P`：矩阵 B 的列数，也是结果矩阵 C 的列数

## 优化路线

本项目的优化思路可以概括为：

1. **朴素矩阵乘法**：三重循环，作为正确性基准。
2. **循环顺序调整**：改善连续访存，减少 cache miss。
3. **循环展开**：减少循环控制开销，提高寄存器复用。
4. **寄存器缓存**：把小块结果暂存在寄存器中，减少重复写回。
5. **矩阵分块**：让计算块更贴近 CPU cache。
6. **Pack A / Pack B**：把访问不连续的数据整理成连续内存。
7. **SIMD 向量化**：使用 SSE 指令一次处理多个 double。
8. **多线程并行**：按行切分任务，利用多核 CPU。

## 已知说明

- `multiply.cpp` 使用 SSE 头文件，更适合 x86/x64 平台；Apple Silicon 上建议先使用 `multiply12.cpp`。
- `matrix.cpp` 中使用了 `std::abs`，部分编译器需要显式包含 `<cmath>`；如果遇到相关报错，可在编译命令中临时添加 `-include cmath`。
- `multiply.cpp` 和 `multiply12.cpp` 都实现了 `matrix_multiplication`，编译时请选择其中一个，不要同时加入编译命令。

## 后续可以继续探索

- 增加 Makefile 或 CMake，统一不同平台的编译入口。
- 针对 AVX/AVX2/AVX-512 扩展 SIMD 实现。
- 支持更多矩阵规模下的自动 block size 调参。
- 增加 benchmark 表格，记录不同优化版本的性能变化。

