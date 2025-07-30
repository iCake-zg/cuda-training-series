
## 简单的过程流

- 1. 把输入的数据从CPU的内存中拷贝到GPU的内存
- 2. 为了功耗加载GPU程序并且在芯片上执行缓存数据
- 3. 把结果从GPU内存拷贝到CPU

## GPU kernels：DEVICE CODE

```cpp
// Function definition:
__golbal__ void mykernel(void){
}
```
cuda C++ keyword： **__global__   表示：

1. runs on device

2. is called from host code (can also called from other device code)

**NVCC** separates source code into host and device components

- Device function（**eg. mykernel()**） processed by **NVIDIA compiler**

- Host functions (**eg. main()**) processed bt standard **host compiler**

```cpp
// function call:
mukernel<<<1,1>>>()
```

## MEMORY MANAGEMENT

**Device** pointers point to GPU memory

	Typically passed to device code 

	Typically not dereferenced in host code

**Host** pointers point to CPU memory

	Typically not passed to device code

	Typically not dereferenced in device code

SImple CUDA API for handling device memoy

	cudaMalloc() , cudaFree() , cudaMemory()

## RUNNING CODE IN PARALLEL


GPU computing is about massive parallelism

	add<<<1,1>>>();

	add<<<n,1>>>(); //instead of execting add() once,excuate N times in parallel


## VECTOR ADDITION ON THE DEVICE

**add<<<n,1>>>();    ： cuda launch N block， each Block includs 1 thread**

**Grid contains Blocks and Thread**

each parallel invocation of **add()** is referred to as a ****Block****

	The set of all blocks is referred to as "Grid"

	 Each invocation can refer to its block index using "blockIdx.x"

	 blocdIdx.x is a build-in variable,

```cpp
__global__ void add(int *a, int *b, int *c){
	c[blockIdx.x] = a[blockIdx.x] + b[blockIdx.x];
}

```

```cpp
#define N 512
int main(void){

	int *a, *b, *c;
	int *d_a, *d_b, *d_c;
	int size = N * sizeof(int);


	// alloc space for device copies of a,b,c

	cudaMalloc((void**)&d_a, size);
	cudaMalloc((void**)&d_b, size);
	cudaMalloc((void**)&d_c, size);

	// alloc space for host copies of a,b,c and setup input values

	a = (int*)malloc(size) ; random_ints(a,N);
	b = (int*)malloc(size) ; randow_ints(b,N);
	c = (int*)malloc(size) ;

	// Copy inputs to device
	cudaMemcpy(d_a,a,size,cudaMemcpyHostToDevice);
	cudaMemcpy(d_b,b,size,cudaMemcpyHostToDevice);

	//launch add() kernel on GPU with N blocks

	add<<<N,1>>>(d_a,d_b,d_c);

	// Copy result back to host
	cudaMemcpy(c,d_c,dize,cudaMemcpyDeviceToHost);

	//claen up
	free(a);free(b);free(c);
	cudaFree(d_a);cudaFree(d_b);cudaFree(d_c);

	return 0;
}

```

## CUDA THREADS

	a block can be split into parallel threads

change **add()** to use parallel threads instead of parallel blocks

```cpp
__golbal__ void add(int *a, int *b, int *c){

	c[threadIdx.x] = a[threadIdx.x] + b[threadIdx.x];
}


// launch 1 block with N threads; 

add<<<1,N>>>(); 

```

## COMBINING BLOCKS -AND- THREADS

- Many blocks with one thread each.

- One block with many threads.

![[Pasted image 20250730142511.png]]


with M threads/block a unique index for each thread is given by

```cpp
int index = threadIdx.x + blockIdx.x
```
![[Pasted image 20250730142834.png]]


```cpp
__global__ void add(int *a, int *b,int *c){
	int index = threadIdx.x + blockIdx.x * blockDim.x;
	c[index] = a[index] + b[index];
}

```

```cpp

#define N 512
#define THREADS_PER_BLOCK 512
int main(void){

	int *a, *b, *c;
	int *d_a, *d_b, *d_c;
	int size = N * sizeof(int);


	// alloc space for device copies of a,b,c

	cudaMalloc((void**)&d_a, size);
	cudaMalloc((void**)&d_b, size);
	cudaMalloc((void**)&d_c, size);

	// alloc space for host copies of a,b,c and setup input values

	a = (int*)malloc(size) ; random_ints(a,N);
	b = (int*)malloc(size) ; randow_ints(b,N);
	c = (int*)malloc(size) ;

	// Copy inputs to device
	cudaMemcpy(d_a,a,size,cudaMemcpyHostToDevice);
	cudaMemcpy(d_b,b,size,cudaMemcpyHostToDevice);

	//launch add() kernel on GPU with N blocks

	add<<<N/THREADS_PER_BLOCK,THREADS_PER_BLOCK>>>(d_a,d_b,d_c);

	// Copy result back to host
	cudaMemcpy(c,d_c,dize,cudaMemcpyDeviceToHost);

	//claen up
	free(a);free(b);free(c);
	cudaFree(d_a);cudaFree(d_b);cudaFree(d_c);

	return 0;
}
```

## HANDLING ARBITRARY VECTOR SIZES

```cpp
__global__ void add(int *a, int *b,int *c,int n){
	int index = threadIdx.x + blockIdx.x * blockDim.x;
	if(index < n)
		c[index] = a[index] + b[index];
}

// launch function call

add<<<(N + M-1) / M,M>>>(d_a,d_b,d_c,N);

```

## WHY THREADS ?

Unlike parallel blocks, threads have mechanisms to :

	- Communicate
	- Synchronize


![[Pasted image 20250730152420.png]]
