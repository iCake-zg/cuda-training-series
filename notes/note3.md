
## OUTLINE

### Architecture

	Kepler / Maxwell / Pascal / Volta

### kernel optimization

	Launch configuration(use lots of threads)


| SM（Streaming Multiprocessor） | CUDA 中执行线程块的最小调度单位              |
| ---------------------------- | ------------------------------- |
| 一个 GPU                       | 包含多个 SM（如 20～100 个）             |
| 一个 SM                        | 可以调度多个 Thread Block，同时运行多个 Warp |
| 一个 Warp                      | 是 SM 调度的基本线程组，包含 32 个线程         |


## KEPLER- GK110

![[Pasted image 20250731115058.png]]

####  🧮 1. **192 SP units ("cores")**

- 每个 SMX 有 192 个 Scalar Processor（SP），也就是 CUDA Core，执行浮点/整数操作。
    
####  🔁 2. **64 DP units**

- 专门用于 double-precision（双精度）运算，浮点科学计算关键。
    
#### 💾 3. **LD/ST units + 64K registers**

- **LD/ST Units**：负责 Load/Store 内存读写。
    
- **Registers**：SM 内部的最快存储，每个线程会分配寄存器空间。
    
    - 图中 "Register File (65536 × 32-bit)" 表示一个 SMX 共有 **64K 个 32-bit 寄存器**

#### 🧵 4. **4 Warp Schedulers**

- 每个 SMX 有 4 个 warp scheduler，用于调度每个 warp 的指令。
    
- 每个 scheduler 每个周期可调度两个 warp 指令（**dual-issue capable**）
    
- 所以一个 SMX 每个 cycle 最多发出 **8 条指令**。


## EXCUTION MODEL

![[Pasted image 20250801113944.png]]

### 蓝色部分

- 每个 SM（Multiprocessor）中包含一个共享的 **Register File**
    
- 为 block 中的所有线程分配高速寄存器，用于局部变量、临时值等
    
- 非常快、并行读写、每个线程有独立的逻辑寄存器空间
    
- **受限资源**：寄存器数量是影响 block 数量/线程数调度的重要限制因素之一

### 绿色部分

- 每个 SM 内部的线程共享的低延迟缓存区域
    
- 可以被同一个 Thread Block 内的所有线程访问，用于线程间通信、数据重用
    
- 用户可以显式使用 `__shared__` 声明变量
    
- **也是资源瓶颈之一**，多个 block 要共享有限的 shared memory 空间

### 灰色部分

- 同时调度多个 thread blocks 的执行单元 SM


	Several concurrent thread blocks can reside on one multiprocessor - limited by shared memory and register file / 一个 SM 同时最多能运行多少个 block，是受其共享资源限制的


## WARPS

GPU的命令不是单线程的，而是一个WARP（32个线程）同时发布的

![[Pasted image 20250801141620.png]]

## LAUNCH CONFIGURATION

**Need enough threads to hide latency

![[Pasted image 20250801152632.png]]

	Need enough total threads to keep GPU busy

	Threads per block should be a multiple of warp size (32)

	SM can concurrently execute at least 16 thread blocks

想要高效使用 GPU，就必须启动足够多的线程，合理设计每个 block 的大小，让 GPU **既能调度得动，又能切换得开**。

| 项目                        | 建议值              |
| ------------------------- | ---------------- |
| 每个 block 的 thread 数       | 128 ~ 256        |
| 每个 SM 的总线程数               | 至少 512，建议接近 2048 |
| 是否对齐 warp 大小              | 是（32 的倍数）        |
| 是否 saturate global memory | 线程够多 + 访问模式对     |
